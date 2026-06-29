---
name: slide-build
description: Build the final single-file, on-brand Hapag-Lloyd HTML slide deck from the approved outline, using the embedded design system (design-system.md, authoritative) + the bundled HL brand assets (assets/) + the /frontend-design skill for polish. Load this for phase 3. The HL brand (cover photo, logo, fonts, palette) is MANDATORY for every deck regardless of subject. Never substitute a neutral cover. Every number must match the approved outline exactly.
---

# slide-build — build the board-ready, Hapag-Lloyd-branded HTML deck

You build ONE self-contained `.html` deck from the **approved outline**. The look,
feel, AND brand are **locked** — this is the company house style.

## RULE 0 — the Hapag-Lloyd brand is MANDATORY (do not "adapt" it away)
Every deck this agent produces is a **Hapag-Lloyd-branded** deck, no matter the
subject (finance, a product, Databricks, Omnigent, anything). You must NOT decide
that "this topic isn't about Hapag-Lloyd, so I'll use a neutral cover" — that is
wrong. ALWAYS:
- Use the **HL cover template** from `design-system.md` §4 with the **real HL hero
  photo** and the **white HL logo** shipped in this skill's `assets/` folder.
- Use **Arial**, the **locked palette** (§2), **square corners**, the **4px orange
  top bar + 8px title marker**, the **1920×1080 canvas + JS scaler**.
The deck's TOPIC is just the content that fills this brand; it never changes the brand.

## Brand assets — shipped with this skill (use them; never invent a logo)
Co-located in THIS skill's folder:
- `assets/hero-hamburg-port.datauri` — the cover hero. Its content is a full
  `data:image/jpeg;base64,…` string → cover `background-image: url('<that string>')`.
- `assets/logo-hl-white.datauri` — the white HL logo, a full
  `data:image/svg+xml;base64,…` string → the cover logo `src`.
**Delivery (orchestrator does this):** because the executor works in a separate
worktree, the orchestrator that loaded this skill MUST copy `design-system.md`,
`assets/hero-hamburg-port.datauri`, and `assets/logo-hl-white.datauri` from this
skill's folder INTO the executor's build worktree (e.g. a `./_deck_assets/` dir)
and tell the executor their paths, so the executor reads them locally. If the
assets genuinely cannot be located/read, **STOP and ask the user to provide the HL
hero image + logo** — do NOT build a deck without the real HL cover.

## Authoritative spec — read it fully first
`design-system.md` (in THIS skill's folder) is the canonical HL slide-deck design
system — tokens, chrome, components, charts, the JS scaler, the build checklist.
**Read it end to end and follow it verbatim.** It is the authority on every colour,
size, font, and layout rule.

## Banned terms — applied CORRECTLY (don't drop real subject matter)
`design-system.md` §9 bans certain visible terms. That ban targets **internal /
backstage engineering jargon** — SQL, Python, table/job/script names, run ids,
pipeline/tool plumbing, "Rule N", CLAUDE.md, and the names of the *authoring* tools
(Claude Code, Codex, Cursor, Pi) when they are not the topic. It does **NOT** ban
the deck's actual **subject**: if the deck is about a product or platform
(e.g. Databricks, Omnigent), naming that product is REQUIRED — it's the content.
So: keep subject/product names; strip only the backstage how-it-was-made jargon.

## QUALITY — make it smart, visual, and a real story (not sparse boxes)
The deck must look like a board-ready controlling deck, densely and intelligently
designed — NOT empty cards with placeholder rectangles.
- **Every content slide carries a real visual or substantive content.** Hand-build
  the chart/diagram the outline suggested as inline SVG per `design-system.md` §6
  (one orange highlight series; value labels with context; map data→pixels; read it
  back against the numbers). NEVER leave an empty box / placeholder rectangle / a
  card that's >60% empty — that's the "empty-card syndrome" the design system
  forbids (§8.4). If a slide is thin, add a chart, a stat, bullets, or merge it.
- **Tell the story.** Title = the slide's conclusion. Follow the arc (question →
  data → mechanism → validation → results → next steps). One idea per slide.
- **Use the components** (cards, callouts, stats, tables, splits, viz frames) to
  give each slide structure and visual hierarchy — vary the layout slide to slide.
- **Accuracy:** every number equals the approved outline/source exactly (money ≥ 2
  decimals; TEU integer + separators; % ≥ 2 decimals; `&minus;` for negatives).
  Each slide has a footer source line.

## Integrate /frontend-design (polish within the brand)
Invoke your **`/frontend-design`** skill for alignment, spacing rhythm, visual
hierarchy, and a polished non-generic result — BUT the HL design system **wins on
every conflict** (Rule 0). frontend-design improves execution *within* the brand;
it never overrides Arial/square/flat/palette/the HL cover.

## Build procedure
1. Paste the §2/§3/§4/§6/§7 CSS/JS from `design-system.md` **verbatim**.
2. Build the **HL cover** (embed the hero + logo data-URIs from `assets/`), then ONE
   `<section class="slide">` per outline slide, each with a real chart/visual.
3. Hand-build every chart (no library) in the locked palette.
4. **Self-review with the §10 checklist BEFORE reporting:** slide-wrap count ==
   slide count; HL hero + logo present on the cover; all `data:image` embedded; zero
   external paths; JS scaler present; **no empty/placeholder boxes**; numbers match
   the outline; banned backstage jargon absent (subject names kept); open it at
   1920/1280/768px — nothing overlaps, nothing clipped, charts legible, palette correct.

## NEVER GUESS
Build only from the approved outline; if a slide needs a value the outline didn't
supply, STOP and report it as an OPEN QUESTION. Output the single `.html` path and a
short note of how you verified it (incl. that the HL cover + assets are in place).
