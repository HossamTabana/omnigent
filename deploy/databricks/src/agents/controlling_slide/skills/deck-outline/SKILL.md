---
name: deck-outline
description: Turn a faithful extraction into a clean, well-organized Markdown slide-deck outline with a storytelling arc, one idea per slide, every number tagged with its source, and a suggested chart/diagram per slide. Load this for phase 2. NEVER invent content to fill a slide — surface gaps as OPEN QUESTIONS.
---

# deck-outline — structure the facts into a story (Markdown deck outline)

Goal: a **clean, organized Markdown outline** of the deck that a human can read and
approve before any HTML is built. It carries the **story**, the **exact numbers**
(each with its source), and a **chart/diagram suggestion** per slide.

## Storytelling arc (one idea per slide)
Order the deck as: **cover → the question → the data → the mechanism → the
validation → the results → the next steps.** If content answers two questions,
split it into two slides. The **title of each slide is its conclusion**, in
sentence case, third person (not a category label).

## Per-slide block (use this exact shape)
```
### Slide N — <conclusion-style title>
- Eyebrow: <subject area>
- Key point: <the single idea>
- Content: <bullets / stat / table — ONLY facts from the extraction>
- Numbers: <each value> — source: <page/sheet/section from the extraction>
- Suggested visual: <chart or diagram type> — series/axes: <which exact numbers>
- Footer source: <data provenance line>
```
Cover slide carries: eyebrow (subject), title (the conclusion the deck proves),
one supporting sentence, and date · place · author · team (ask the user for any of
these you don't have).

## Suggesting charts/diagrams (for full storytelling)
Match the visual to the data, and name the EXACT numbers it plots:
- Trend over time → line/area; comparison across categories → bar; part-to-whole →
  donut/stacked bar; flow/process → funnel or flow diagram; before/after or
  driver breakdown → waterfall; relationships → simple node diagram.
- One highlight series only (the orange accent). Prefer one strong visual per
  slide over many weak ones. If the data doesn't support a chart, say "table" or
  "no chart" — don't manufacture one.

## NEVER GUESS
- Every number/claim in the outline MUST come from the extraction. If a slide needs
  a fact that isn't there (a label, a total, a unit, a date, the author/place for
  the cover), add it to `## OPEN QUESTIONS` — do not invent or estimate it.
- Do not silently compute new figures (totals, deltas, %s) unless the source
  supports them; if you derive one, show the arithmetic and its source inputs so
  Codex can check it.

## Output
The Markdown outline (cover + one block per slide) followed by `## OPEN QUESTIONS`.
The orchestrator shows this to the user, gets gaps answered, and only then proceeds
to BUILD.
