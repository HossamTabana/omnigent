---
name: lakebase-autoscaling
description: Core Databricks Lakebase Autoscaling knowledge — projects, branches, endpoints/computes, scale-to-zero, autoscaling CU limits, credentials, reverse-ETL synced tables, app connection — plus the hard-won gotchas. Load this for ANY Lakebase Autoscaling reasoning or before dispatching an executor task.
---

# Lakebase Autoscaling — operating knowledge

Lakebase Autoscaling is Databricks' next-gen managed PostgreSQL for OLTP:
autoscaling compute, database branching, scale-to-zero, instant restore, and
Delta→Postgres synced tables (reverse ETL).

> **There is no separate "Lakebase SDK".** Use `databricks-sdk`
> (`WorkspaceClient().postgres`) for management + minting DB credentials; use a
> standard Postgres driver (`psycopg`, SQLAlchemy `postgresql+psycopg://`) for SQL.
> CLI mirrors the SDK under `databricks postgres …`.

## Resource model
```
Project                      projects/{project_id}
  └── Branch                 projects/{project_id}/branches/{branch_id}
        ├── Endpoint/compute  …/endpoints/{endpoint_id}   (one primary read-write; optional read-only replicas)
        ├── Roles
        └── Databases         …/databases/{database_id}
```
Defaults on project creation: branch `production`, database `databricks_postgres`,
a primary read-write endpoint, a Postgres role for the creator's identity,
Postgres 17. Most create/update/delete calls return long-running operations —
call `.wait()`. GET returns effective props under `.status`; create/update take `.spec`.

## Core operations (SDK)
```python
from databricks.sdk import WorkspaceClient
from databricks.sdk.service import postgres as pg
w = WorkspaceClient(profile="da_token")   # default profile unless told otherwise

# Create project (ASK the user before creating anything)
w.postgres.create_project(
    project=pg.Project(spec=pg.ProjectSpec(
        display_name="…", pg_version="17",
        default_endpoint_settings=pg.ProjectDefaultEndpointSettings(
            autoscaling_limit_min_cu=0.5, autoscaling_limit_max_cu=4.0,
            suspend_timeout_duration=pg.Duration(seconds=3600),  # enable scale-to-zero (non-prod)
        ),
    )),
    project_id="my-app",
).wait()

# Branches (dev/test/staging, CI, point-in-time)
w.postgres.create_branch(parent="projects/my-app",
    branch=pg.Branch(spec=pg.BranchSpec(source_branch="projects/my-app/branches/production",
        ttl=pg.Duration(seconds=604800))),  # or no_expiry=True
    branch_id="development").wait()
w.postgres.reset_branch(name=".../branches/development").wait()
w.postgres.delete_branch(name=".../branches/development").wait()   # ASK first

# Endpoints / computes
w.postgres.create_endpoint(parent=".../branches/development",
    endpoint=pg.Endpoint(spec=pg.EndpointSpec(
        endpoint_type=pg.EndpointType.ENDPOINT_TYPE_READ_WRITE,
        autoscaling_limit_min_cu=0.5, autoscaling_limit_max_cu=4.0,
        suspend_timeout_duration=pg.Duration(seconds=3600))),
    endpoint_id="primary").wait()
# Resize an EXISTING endpoint (FieldMask required); autoscaling CU only:
w.postgres.update_endpoint(name="…/endpoints/primary",
    endpoint=pg.Endpoint(name="…/endpoints/primary", spec=pg.EndpointSpec(
        endpoint_type=pg.EndpointType.ENDPOINT_TYPE_READ_WRITE,
        autoscaling_limit_min_cu=2.0, autoscaling_limit_max_cu=8.0)),
    update_mask=pg.FieldMask(field_mask=[
        "spec.autoscaling_limit_min_cu","spec.autoscaling_limit_max_cu"])).wait()
host = w.postgres.get_endpoint(name="…/endpoints/primary").status.hosts.host
```

## Compute sizing rules
- Autoscaling range **0.5–32 CU**, and **`max - min <= 16`** (e.g. 4–20, 8–16 ok; 0.5–32 invalid).
- ~2 GB RAM per CU. Connection limit scales with **max** CU.
- Fixed always-on computes 40–112 CU (no autoscaling).
- Set **min** high enough for working-set cache / latency.

## Credentials / connection (CRITICAL)
- Mint a Lakebase-scoped token: `w.postgres.generate_database_credential(endpoint=<endpoint_path>)`; use `.token` as the Postgres password with `sslmode=require`.
- Do NOT use `w.config.token` / a workspace OAuth token as the Postgres password — it fails at login.
- Production app pattern: `psycopg_pool.ConnectionPool` with an `OAuthConnection` that mints on connect, `max_lifetime=2700` (45-min recycle before the ~60-min token expiry).

## Reverse ETL (synced tables)
- Delta→Postgres only (NOT Postgres→Delta). Triggered/Continuous modes require Delta Change Data Feed. Use for serving features/reference data into Lakebase.

## GOTCHAS (verified — do not relearn the hard way)
- **A PRODUCTION read-write endpoint CANNOT scale to zero.** RW endpoints cannot
  be deleted, and `suspend_timeout_duration` is rejected by `update_endpoint`'s
  field mask and is not applied to the auto-created prod primary. It runs at min
  CU 24/7. True scale-to-zero needs a non-production branch.
- **`no_suspension` may only be set to `true`** (to DISABLE suspend). To ENABLE
  suspend, set `suspend_timeout_duration` and OMIT `no_suspension`.
- `EndpointSpec` requires `endpoint_type` even on update.
- Scale-to-zero: default inactivity 5 min, minimum 60 s; first connection wakes
  the compute (apps should retry); reactivated compute starts at MIN size and
  resets session state (temp tables, prepared stmts, caches).
- macOS DNS can fail on long Lakebase hostnames; if so resolve to IP and pass
  both `host` and `hostaddr` to psycopg.
- Lakebase DB resource path uses a hyphen (`…/databases/databricks-postgres`)
  while the PostgreSQL database NAME uses an underscore (`databricks_postgres`).

## Safety (this agent's law)
- NEVER GUESS names. **ASK the user before creating or deleting** any project,
  branch, endpoint, or any Unity Catalog catalog/schema/table/volume. Destructive
  ops (delete/reset) need explicit confirmation every time. Default profile
  `da_token`; the user owns object placement.
