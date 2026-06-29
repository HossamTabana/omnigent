---
name: slide-build
description: Build the final single-file, on-brand HTML slide deck from the approved outline, using the embedded slide-deck design system (design-system.md, authoritative) plus the /frontend-design skill for polish. Load this for phase 3. Use the brand assets shipped in this agent's assets/ folder. Every number must match the approved outline exactly.
---

# slide-build — build the board-ready HTML deck

You build ONE self-contained `.html` deck from the **approved outline**. The look
and feel is **locked** by the design system shipped next to this skill.

## Authoritative spec — read it fully first
`design-system.md` (in THIS skill's folder) is the canonical Hapag-Lloyd slide-deck
design system — tokens, slide chrome, components, charts, the JS scaler, and the
build→validate checklist. **Read it end to end and follow it verbatim.** It is the
authority on every colour, size, font, and layout rule. (If the repo also has the
fuller `SlideDeck_DesignSystem_Guide.md` at its root, you may read it too — same spec.)

## Brand assets — use the ones shipped with this agent
This agent bundles the real brand assets so the deck is on-brand without external
files. Relative to this agent package (`<agent>/assets/`):
- **Cover hero photo** → `assets/hero-hamburg-port.datauri` — its content is a full
  `data:image/jpeg;base64,…` string. Inline it as the cover hero:
  `background-image: url('<contents of hero-hamburg-port.datauri>')`.
- **White HL logo** → `assets/logo-hl-white.datauri` — a full
  `data:image/svg+xml;base64,…` string. Use as the cover logo `src`.
Resolve the path from your worktree to the agent bundle; if you cannot locate the
assets, ASK the user for the hero image + logo — do NOT ship a deck with a missing
or invented logo.

## Integrate /frontend-design (polish within the brand)
Invoke your **`/frontend-design`** skill while building, to get clean alignment,
spacing rhythm, visual hierarchy, and a polished, non-generic result. BUT the
slide-deck **design system always wins on any conflict**: Arial only, square
corners (`border-radius:0`), flat (no shadows/gradients except the cover hero
tint), the locked palette only, 1920×1080 canvas with the JS scaler, 4px orange
top bar + 8px title marker. frontend-design improves the execution *within* these
constraints — it never overrides the brand tokens.

## Build procedure (from design-system.md)
1. Start from the §1 skeleton; paste the §2 (`:root` + base), §3 (chrome +
   components), §4 (cover), §6 (charts), §7 (JS scaler) CSS/JS **verbatim**.
2. Build the cover (embed the hero + logo data-URIs from `assets/`), then ONE
   `<section class="slide">` per outline slide.
3. Hand-build every chart as inline SVG in the locked palette (no chart library);
   one orange highlight series; value labels carry context; map data→pixels and
   read the rendered output back against the numbers.
4. **Accuracy gate:** every number on a slide must equal the approved outline /
   source exactly (money ≥ 2 decimals; TEU integer + thousands separators; % ≥ 2
   decimals; `&minus;` for negatives). Each slide has a footer source line.
5. **Self-review with the design-system §10 checklist BEFORE reporting:**
   slide-wrap count == slide count; all `data:image` embedded; zero external paths
   (`../`, `./assets/`, http) in the final HTML; the JS scaler present; banned
   visible terms absent (SQL/Python/Databricks/table/job/tool/AI names); open it
   mentally at 1920 / 1280 / 768px — nothing overlaps, nothing clipped, no
   empty-card syndrome, charts legible, palette correct.

## NEVER GUESS
Build only from the approved outline. If a slide still needs a value the outline
didn't supply, STOP and report it as an OPEN QUESTION — do not fabricate it. Output
the single `.html` file path and a short note of how you verified it.
