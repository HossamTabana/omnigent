---
name: validate
description: The execute→validate loop. Claude Code (executor) performs a Lakebase Autoscaling change in its worktree; Codex (a different vendor) independently validates the result against an acceptance contract and the Lakebase rules. Use for EVERY change (create/update/scale/deploy/wire), never for read-only questions.
---

# validate — Claude Code executes, Codex validates

Every change to Lakebase (or its app wiring) goes through this loop. The point is
**independent cross-vendor verification**: the worker that makes the change is
never the worker that judges it.

## 0. Gate first (safety)
If the change CREATES or DELETES any Lakebase project/branch/endpoint or any UC
catalog/schema/table/volume, you must already have the user's explicit
confirmation (see the orchestrator's safety rules). If you don't, STOP and ask —
do not dispatch the executor.

## 1. Write the ACCEPTANCE CONTRACT
A short, checkable spec the validator can judge against:
- Exact resource names/paths (project, branch, endpoint, database, profile).
- Intended end-state (e.g. "endpoint `…/endpoints/primary` ACTIVE, autoscaling
  min 0.5 / max 4 CU, scale-to-zero 3600 s").
- Rules that must hold (CU range 0.5–32 & `max-min ≤ 16`; prod RW can't
  scale-to-zero; credential minted via `generate_database_credential`,
  `sslmode=require`; no unauthorized create/delete).

## 2. Execute (claude_code)
`sys_session_send` to `claude_code` with `args.purpose: "implement"`, a task
`title` like `create-dev-branch`, and a body containing the contract + the precise
steps. Require it to READ BACK the resulting state and report exact paths/values
and how it verified. End your turn; collect with `sys_read_inbox`.

## 3. Validate (codex) — different vendor
`sys_session_send` to `codex` with `args.purpose: "review"` and a body containing
ONLY: the user's request, the acceptance contract, and Claude Code's reported
plan/diff/result. Do NOT point Codex at the worktree. Ask for BLOCKING /
NON-BLOCKING / SUGGESTION buckets and a final `VALIDATION: PASS|FAIL` line.
End your turn; collect with `sys_read_inbox`.

## 4. Converge
- `VALIDATION: PASS` → report the executor's result AND the validator's verdict to
  the user; the change is done.
- `VALIDATION: FAIL` (blocking issues) → route a focused fix task back to
  `claude_code` (purpose `implement`) citing each blocking issue, then re-run step
  3. Repeat until PASS or until you need the user to decide. Never ship a change
  that failed validation; never let the validator edit the change itself.

## Notes
- Need both workers available (roster preflight). If `codex` isn't launchable you
  cannot run independent validation — tell the user rather than self-certifying.
- Keep each worker in its own worktree; record `conversation_id`s; inspect an
  empty/unclear result with `sys_session_get_history` before re-dispatching.
- Read-only questions do NOT use this skill — dispatch a single `explore`.
