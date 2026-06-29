---
name: resilience
description: Handle large documents and model/provider limits (especially the Databricks model gateway) without failing. Chunk big content across multiple dispatches, checkpoint partial work so a retry resumes, and retry transient rate-limit / quota errors with backoff. Load this whenever the source is large or a worker hits a model limit.
---

# resilience — big content + Databricks gateway limits

Large uploads + the **Databricks model gateway's rate/throughput and per-request
token limits** are the main way a run fails. Plan around them; never let a big
document produce a truncated or failed deck.

## Why this matters here
- A Databricks-profile provider (vs a Claude/Codex subscription) routes through the
  workspace AI gateway, which enforces **request-rate** and **max-tokens-per-request**
  limits. A single giant extraction or build prompt can exceed them.
- Native workers (Claude Code / Codex) already retry transient errors a few times,
  but a too-large request or a sustained rate cap needs **chunking**, not just retry.

## Orchestrator strategy — CHUNK across dispatches (the real fix)
- **Size up first.** Before dispatching, gauge the source size (pages / sheets /
  rows / characters). If it's large, split the work:
  - EXTRACT: dispatch one `claude_code` task **per section / per sheet / per page
    range**, each returning a partial extraction; then assemble the parts. Never
    ask one worker to ingest a huge file in a single call.
  - OUTLINE / BUILD: build the deck **incrementally** — outline then build in
    batches of slides — rather than one massive prompt.
- **Checkpoint.** Tell each worker to write its partial result to a file in its
  worktree as it goes, so a re-dispatch resumes from the last good chunk instead of
  restarting. Collect partials via the inbox and stitch them.
- **Bounded fan-out.** Respect the per-turn dispatch cap (max 5); for many chunks,
  process in waves across turns rather than exceeding it.

## Worker behavior on a limit (put this in the task prompt)
Instruct `claude_code` / `codex`:
- On a transient **rate-limit / quota / "limit exceeded" / 429 / gateway timeout**:
  **retry with exponential backoff** (e.g. wait 5s, 10s, 20s, 40s; a few attempts).
  Let the native CLI's own retry run; only escalate if it still fails.
- On a **context / max-tokens** error: do NOT retry the same oversized prompt —
  split the input (fewer pages/rows per call) and process in parts, then report the
  parts. Reducing the request is the fix, not retrying it.
- **Never drop content to fit a limit.** If something can't be processed within the
  limits, report exactly what was left and let the orchestrator chunk it — don't
  silently omit it (that would violate NEVER GUESS / fidelity).

## When to ask the user
If the source is so large that even chunked processing will be slow/costly, tell
the user the size and options (process all of it in batches, or focus on specific
sections), and let them choose — don't silently truncate.
