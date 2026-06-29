---
name: validate
description: The execute→review loop for each phase of the deck pipeline. Claude Code produces (extract / outline / build); Codex (a different vendor) independently reviews against the source and the rules; blocking issues go back to Claude Code. Load this to run any phase's review. Use for EVERY phase before moving on.
---

# validate — Claude Code produces, Codex reviews, loop until PASS

Independent cross-vendor verification at every phase. The worker that produces an
artifact is never the worker that judges it.

## The loop (per phase)
1. **Produce (claude_code).** `sys_session_send` to `claude_code` (`args.purpose:
   "implement"`) with a clear task + the inputs (the source, or the approved prior
   artifact) and an ACCEPTANCE CONTRACT (what "correct" means for this phase).
   End your turn; collect with `sys_read_inbox`.
2. **Review (codex).** `sys_session_send` to `codex` (`args.purpose: "review"`)
   with ONLY: the phase, the SOURCE / prior approved artifact, the acceptance
   contract, and claude_code's output. Do NOT point it at the worktree. Ask for
   BLOCKING / NON-BLOCKING / SUGGESTION buckets and a final `VALIDATION: PASS|FAIL`.
   End your turn; collect with `sys_read_inbox`.
3. **Converge.**
   - `VALIDATION: PASS` → record it and move to the next phase (or present to the user).
   - `VALIDATION: FAIL` → route a FIX task back to `claude_code` citing each BLOCKING
     issue verbatim, then re-review (step 2). Repeat until PASS or until an OPEN
     QUESTION needs the user. Never advance a phase that failed review.

## Acceptance contracts per phase
- EXTRACT: nothing dropped/altered vs source; numbers/units/dates exact; ambiguities
  flagged as OPEN QUESTIONS, not guessed.
- OUTLINE: every number traces to the extraction; nothing invented; one idea/slide;
  story arc holds; chart suggestions fit the data.
- BUILD: design-system compliant (brand/palette/Arial/square/1920×1080/scaler),
  numbers match the outline exactly, footer source lines present, banned terms
  absent, nothing overlaps/clips.

## Notes
- Need BOTH workers available (roster preflight). If `codex` can't boot you cannot
  run independent review — tell the user; never self-certify.
- Keep each worker in its own worktree; record `conversation_id`s; inspect an
  empty/unclear result with `sys_session_get_history`.
- The NEVER-GUESS rule is the top review priority: any invented or unverifiable
  number is always a BLOCKING issue, at every phase.
