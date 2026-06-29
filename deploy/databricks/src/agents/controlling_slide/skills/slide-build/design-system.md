# Slide-Deck Design System — Build Guide for Claude Code

> **Purpose.** This is the single, self-contained reference for building a Hapag-Lloyd–branded HTML
> slide deck **identical in look and feel** to `docs/NetFreightRevenue_SlideDeck_charge_code.html`.
> Every token, size, colour, font, layout rule, chart pattern, and the scaler below is **extracted
> verbatim from that file** — not invented. If a new session is given a topic + the text/numbers and
> asked to "make a slide deck like the charge-code one", follow THIS file end to end and the result
> will match.
>
> **Companion rules in CLAUDE.md** (still apply, this file is the concrete implementation of them):
> "Mandatory Rules for Building HTML Slide Decks" (brand discipline, JS scaler, validation) and
> Rules 48–58 (per-deck adjustment protocol). When in doubt, this file is the canonical spec.

---

## 0. The 10 non-negotiables (the "feel")

1. **Arial only.** `Arial, "Helvetica Neue", Helvetica, "Liberation Sans", sans-serif` — set on `html, body, .slide, button, input, select, textarea` AND on `svg, svg text` (SVG does not inherit it).
2. **One fixed canvas: 1920 × 1080** per slide, scaled to the viewport by a tiny JS scaler (§7). Never design at any other size.
3. **Square corners everywhere.** `border-radius: 0`. No rounded cards/buttons/charts.
4. **Flat — no shadows, no gradients.** The ONLY gradient is the cover hero tint. Elevation = a `1px var(--hl-slate-200)` border, never blur.
5. **Orange `#FF6600` is an accent, never a background fill.** Allowed orange surfaces: the 4px top bar, the 8px title marker, icons, ONE highlight data series, accent text (eyebrows/key numbers), the tint callout/card.
6. **4px orange top bar on every content slide; 8px orange title marker** left of every title.
7. **Sentence case. Third person. No emoji. No exclamation marks.** Bold is the only emphasis; italics only for quoted definitions.
8. **Body 13–14px, titles 25px, big stats 28–56px** (exact scale in §3). Tight, dense, controlling-deck style.
9. **Colours come ONLY from the locked palette** (§2). No purple, no teal, no extra blue.
10. **Nothing overlaps, nothing is clipped, nothing escapes the slide** (§6 + §8). Every container has `overflow: hidden; min-height: 0`.

---

## 1. File skeleton (single self-contained .html)

One file. Everything embedded (CSS inline in `<style>`, images as base64 data URLs, SVG icons as `<symbol>` defs). No external links, no asset folder — it must open offline and email cleanly.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>… deck title …</title>
<style> /* §2 + §3 — the entire design system, pasted verbatim */ </style>
</head>
<body>
  <!-- hidden SVG <defs> of reusable <symbol> icons (§5) -->
  <svg width="0" height="0" style="position:absolute"><defs> … </defs></svg>

  <div class="deck">
    <div class="slide-wrap"><section class="slide cover"> … §4 … </section></div>
    <div class="slide-wrap"><section class="slide"> … §3 content slide … </section></div>
    <!-- one slide-wrap + slide per slide -->
  </div>

  <script> /* §7 — the JS scaler, pasted verbatim */ </script>
</body>
</html>
```

**Structural invariant:** the number of `<div class="slide-wrap">` must equal the number of
`<section class="slide …">`. Count them before shipping.

---

## 2. Design tokens — paste this `:root` and base verbatim

```css
:root {
  --hl-orange:       #FF6600;   /* accent only */
  --hl-orange-hover: #E65C00;
  --hl-orange-tint:  #FFE0CC;   /* emphasis card/callout/total-row bg */
  --hl-ink:          #414048;   /* primary text (graphite, NOT pure black) */
  --hl-white:        #FFFFFF;
  --hl-slate-900:    #425464;
  --hl-slate-700:    #556A7A;   /* secondary text, axis labels */
  --hl-slate-500:    #768A9D;   /* tertiary text, ticks, meta */
  --hl-slate-300:    #ADBAC7;   /* axis lines, light bar series */
  --hl-slate-200:    #CCD6DF;   /* borders, grid lines */
  --hl-slate-100:    #F0F2F5;   /* card / surface background */
  --hl-positive:     #2E7D32;   /* green — only for genuinely positive deltas */
  --hl-negative:     #C62828;   /* red — only for genuinely negative deltas */
}
*, *::before, *::after { box-sizing: border-box; }
html, body { margin: 0; padding: 0; background: #1a1a1a; }   /* dark gutter behind the deck */
html, body, .slide, button, input, select, textarea {
  font-family: Arial, "Helvetica Neue", Helvetica, "Liberation Sans", sans-serif;
}
body { color: var(--hl-ink); font-size: 14px; line-height: 1.4; }
svg, svg text { font-family: Arial, "Helvetica Neue", Helvetica, "Liberation Sans", sans-serif; }
```

**Colour usage map (memorise):**

| Element | Colour |
|---|---|
| Primary text / titles / big numbers (ink) | `--hl-ink` `#414048` |
| Secondary text, axis labels, eyebrow-slate | `--hl-slate-700` |
| Ticks, meta line, stat labels (uppercase) | `--hl-slate-500` |
| Axis lines, the "light" bar series | `--hl-slate-300` |
| Borders, grid lines, table rules | `--hl-slate-200` |
| Card / surface background | `--hl-slate-100` |
| Accent: top bar, title marker, icons, eyebrow, key number, 1 highlight series | `--hl-orange` |
| Emphasis card / callout / table total row | `--hl-orange-tint` |
| Positive / negative delta text ONLY | `--hl-positive` / `--hl-negative` |

---

## 3. Slide chrome + components — paste verbatim, then fill content

### 3a. The slide grid (every content slide)

```css
.deck { display: flex; flex-direction: column; align-items: center; gap: 24px; padding: 24px 0 48px; }
.slide-wrap {
  width: min(100vw, 1920px); aspect-ratio: 16 / 9;
  position: relative; overflow: hidden; background: #fff;
}
.slide {
  width: 1920px; height: 1080px;
  position: absolute; top: 0; left: 0;
  transform-origin: top left;
  transform: scale(var(--scale, 1));          /* set by the JS scaler */
  background: #fff; color: var(--hl-ink);
  display: grid;
  grid-template-rows: 4px auto 1fr auto;       /* topbar / header / body / footer */
  overflow: hidden;
}
.slide--surface { background: var(--hl-slate-100); }   /* optional: tinted slide bg */
.topbar { background: var(--hl-orange); }              /* the 4px row */
```

Every content slide is exactly:
```html
<div class="slide-wrap"><section class="slide">
  <div class="topbar"></div>
  <div class="header"> … title + meta … </div>
  <div class="body"> … the one idea of this slide … </div>
  <div class="footer"> … source line + page number … </div>
</section></div>
```

### 3b. Header (title + meta), footer (source + page)

```css
.header {
  padding: 32px 64px 18px;
  display: grid; grid-template-columns: minmax(0, 1fr) auto; gap: 32px; align-items: end;
  border-bottom: 1px solid var(--hl-slate-200);
}
.title-block { display: grid; grid-template-columns: 8px minmax(0, 1fr); gap: 18px; align-items: start; }
.title-marker { background: var(--hl-orange); width: 8px; min-height: 36px; align-self: stretch; }
.title-text {
  font-size: 25px; font-weight: 700; line-height: 1.22; color: var(--hl-ink);
  letter-spacing: -0.002em; overflow-wrap: break-word; max-width: 1500px;
}
.meta { font-size: 13px; color: var(--hl-slate-500); white-space: nowrap; letter-spacing: 0.04em; padding-bottom: 4px; }

.body { padding: 24px 64px; overflow: hidden; min-height: 0; display: grid; grid-template-rows: 1fr; }

.footer {
  padding: 12px 64px 18px;
  display: grid; grid-template-columns: minmax(0, 1fr) auto; gap: 24px; align-items: center;
  font-size: 11px; color: var(--hl-slate-500); border-top: 1px solid var(--hl-slate-200);
}
.footer__src { letter-spacing: 0.02em; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
.footer__pg  { letter-spacing: 0.12em; text-transform: uppercase; color: var(--hl-slate-500); font-weight: 700; }
```

Markup:
```html
<div class="header">
  <div class="title-block">
    <div class="title-marker"></div>
    <h2 class="title-text">Sentence-case title that reads like the slide's conclusion</h2>
  </div>
  <div class="meta">04/06/26 &nbsp;|&nbsp; Deck subject line</div>
</div>
...
<div class="footer">
  <div class="footer__src">Source line — the data provenance (table/run), NOT jargon for the audience</div>
  <div class="footer__pg">04 &nbsp;/&nbsp; 28</div>
</div>
```
- The **title is the conclusion** of the slide, not a category label. Wraps inside `max-width:1500px`.
- The **footer source line** is the ONLY place engineering names (table names, run dates) may appear — small grey, ellipsised. Page number right-aligned, `NN / TOTAL`.

### 3c. Eyebrows, cards, stats, callouts, tables, splits — the component CSS (paste verbatim)

```css
.eyebrow { font-size: 11px; font-weight: 700; color: var(--hl-orange); letter-spacing: 0.10em; text-transform: uppercase; }
.eyebrow--slate { color: var(--hl-slate-700); }
.vlabel { font-size: 11px; font-weight: 700; color: var(--hl-slate-700); letter-spacing: 0.08em; text-transform: uppercase; }

/* Card grids — N columns, all minmax(0,1fr) so they shrink, never overflow */
.cards { display: grid; gap: 18px; min-height: 0; }
.cards--stretch { align-items: stretch; }   /* use when content fills the card */
.cards--start { align-items: start; }
.cards--2 { grid-template-columns: repeat(2, minmax(0, 1fr)); }
.cards--3 { grid-template-columns: repeat(3, minmax(0, 1fr)); }
.cards--4 { grid-template-columns: repeat(4, minmax(0, 1fr)); }
.cards--6 { grid-template-columns: repeat(3, minmax(0, 1fr)); grid-template-rows: repeat(2, minmax(0, 1fr)); }

.card {
  background: var(--hl-slate-100); border: 1px solid var(--hl-slate-200);
  padding: 22px 26px; display: flex; flex-direction: column; gap: 10px;
  overflow: hidden; min-height: 0;
}
.card--white { background: #fff; }
.card--tint  { background: var(--hl-orange-tint); border-color: var(--hl-orange); }   /* emphasis card */
.card__title { font-size: 17px; font-weight: 700; line-height: 1.28; margin: 4px 0; color: var(--hl-ink); overflow-wrap: break-word; }
.card__body  { font-size: 13px; line-height: 1.55; color: var(--hl-ink); overflow-wrap: break-word; }
.card__body + .card__body { margin-top: 8px; }
.card__icon  { color: var(--hl-orange); display: block; }
.card__icon svg { width: 44px; height: 44px; }
.card__stat  { font-size: 28px; font-weight: 700; line-height: 1; color: var(--hl-orange); letter-spacing: -0.012em; margin-top: auto; }
.card__stat-label { font-size: 11px; font-weight: 700; color: var(--hl-slate-500); letter-spacing: 0.06em; text-transform: uppercase; }
.card__bullets { list-style: none; margin: 0; padding: 0; }
.card__bullets li { font-size: 13px; line-height: 1.5; margin-bottom: 7px; display: grid; grid-template-columns: 6px 1fr; gap: 10px; align-items: start; }
.card__bullets li::before { content: ""; display: block; width: 6px; height: 6px; background: var(--hl-orange); margin-top: 8px; }

/* Big standalone numbers */
.stat-num { font-size: 56px; font-weight: 700; line-height: 1; color: var(--hl-orange); letter-spacing: -0.012em; }
.stat-num--ink { color: var(--hl-ink); }
.stat-num--md { font-size: 42px; }
.stat-num--sm { font-size: 32px; }
.stat-label { font-size: 11px; font-weight: 700; color: var(--hl-slate-500); letter-spacing: 0.06em; text-transform: uppercase; }

/* Callout band (the orange-bar takeaway) */
.callout { display: grid; grid-template-columns: 8px 1fr; background: #fff; border: 1px solid var(--hl-slate-200); }
.callout__bar { background: var(--hl-orange); }
.callout__body { padding: 16px 22px; }
.callout__label { font-size: 11px; font-weight: 700; color: var(--hl-orange); letter-spacing: 0.10em; text-transform: uppercase; }
.callout__text { font-size: 16px; font-weight: 700; line-height: 1.4; margin-top: 6px; color: var(--hl-ink); }

/* Two-column splits (chart + cards, etc.) */
.split { display: grid; gap: 28px; min-height: 0; height: 100%; }
.split--12 { grid-template-columns: minmax(0, 1.2fr) minmax(0, 1fr); }
.split--11 { grid-template-columns: minmax(0, 1fr) minmax(0, 1fr); }
.split--21 { grid-template-columns: minmax(0, 2fr) minmax(0, 1fr); }
.stack { display: flex; flex-direction: column; gap: 16px; min-height: 0; }

/* Tables */
table.hl { width: 100%; border-collapse: collapse; font-size: 12px; }
table.hl th, table.hl td { padding: 7px 12px; text-align: right; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
table.hl th { font-weight: 700; color: var(--hl-slate-700); border-bottom: 1px solid var(--hl-slate-200); font-size: 11px; letter-spacing: 0.04em; }
table.hl th:first-child, table.hl td:first-child { text-align: left; }   /* first column left, numbers right */
table.hl tbody tr { border-bottom: 1px solid var(--hl-slate-200); }
table.hl tbody tr.tot { background: var(--hl-orange-tint); font-weight: 700; }   /* total row */
table.hl tbody tr.tot td { border-bottom: 0; color: var(--hl-ink); }
table.hl td.hi { color: var(--hl-orange); font-weight: 700; }   /* highlight a cell */
```

**Type scale (the only sizes you use):**

| Token | Size / weight | Use |
|---|---|---|
| Cover title | 56px / 700 | cover only |
| `.stat-num` | 56px / 700 | hero number (orange or `--ink`) |
| `.stat-num--md` / `--sm` | 42 / 32px | secondary hero numbers |
| `.card__stat` | 28px / 700 | in-card stat |
| Slide `.title-text` | 25px / 700 | every content-slide title |
| Cover subtitle | 18px | cover only |
| `.card__title` | 17px / 700 | card heading |
| `.callout__text` | 16px / 700 | takeaway line |
| `.meta` | 13–14px | header meta / body copy / cards |
| `.card__body` | 13px / 1.55 | card paragraph |
| `table.hl td` | 12px | table cells |
| `.eyebrow` / `.stat-label` / `.tick` / `.footer` | 11px / 700 uppercase, 0.06–0.10em tracking | labels, ticks, footer |

**Spacing constants (don't drift):** slide H-padding `64px`; header `32px 64px 18px`; body `24px 64px`;
footer `12px 64px 18px`; card padding `22px 26px`; card grid gap `18px`; split gap `28px`; stack gap `16px`.

---

## 4. Cover slide — image, subject, template

```css
.slide.cover { grid-template-rows: 1fr; background: #0e1a26; position: relative; overflow: hidden; }
.cover__hero {
  position: absolute; inset: 0;
  background-image: url('data:image/jpeg;base64,<HERO_IMAGE_BASE64>');   /* embed the photo */
  background-size: cover; background-position: center 35%;
}
.cover__tint {                                   /* the ONE sanctioned gradient — legibility tint */
  position: absolute; inset: 0;
  background: linear-gradient(94deg,
    rgba(14,26,38,0.86) 0%, rgba(14,26,38,0.60) 38%,
    rgba(14,26,38,0.22) 72%, rgba(14,26,38,0.05) 100%);
}
.cover__topbar { position: absolute; top: 0; left: 0; right: 0; height: 4px; background: var(--hl-orange); z-index: 3; }
.cover__logo   { position: absolute; top: 72px; left: 96px; height: 44px; z-index: 2; }   /* white HL logo, base64 */
.cover__content{ position: absolute; left: 96px; right: 96px; bottom: 130px; z-index: 2; max-width: 1300px; color: #fff; display: grid; gap: 18px; }
.cover__eyebrow{ font-size: 14px; letter-spacing: 0.18em; text-transform: uppercase; color: var(--hl-orange); font-weight: 700; }
.cover__title  { font-size: 56px; font-weight: 700; line-height: 1.10; letter-spacing: -0.005em; color: #fff; max-width: 1180px; }
.cover__sub    { font-size: 18px; color: rgba(255,255,255,0.85); max-width: 1100px; line-height: 1.45; }
.cover__meta   { font-size: 14px; letter-spacing: 0.06em; color: rgba(255,255,255,0.80); margin-top: 16px; }
.cover__sep    { color: var(--hl-orange); padding: 0 12px; font-weight: 700; }
```

```html
<div class="slide-wrap"><section class="slide cover">
  <div class="cover__hero" aria-hidden="true"></div>
  <div class="cover__tint" aria-hidden="true"></div>
  <div class="cover__topbar"></div>
  <img class="cover__logo" src="data:image/svg+xml;base64,<LOGO_WHITE_BASE64>" alt="Hapag-Lloyd">
  <div class="cover__content">
    <div class="cover__eyebrow">Eyebrow — the deck subject area</div>
    <h1 class="cover__title">The headline — one sentence stating what the deck proves</h1>
    <p class="cover__sub">One supporting sentence, plain language.</p>
    <div class="cover__meta">DD/MM/YY <span class="cover__sep">|</span> Hamburg <span class="cover__sep">|</span> Author <span class="cover__sep">|</span> Financial Analytics</div>
  </div>
</section></div>
```

**Cover rules:**
- **One photograph per deck, cover only.** Marine / industrial — port at dawn, warm-orange sky over steely water. Embed as base64 (`background-image`). The brand photo lives at `DesignSystem/hapag-lloyd-design-system/project/assets/hero-hamburg-port.jpeg`; the white logo at `assets/logo-hl-white.svg` — base64-encode both into the file.
- The cover **omits the chrome grid** (`grid-template-rows: 1fr`) but keeps a 4px orange top bar for brand continuity.
- The hero tint is the **only** gradient allowed in the entire deck.
- "**Subject**" usage: the eyebrow = the subject area (e.g. "Net Freight Revenue Forecast — Charge-code level"); the title = the conclusion; the meta line = date · place · author · team, separated by orange `|`.
- Use `&#8209;` (non-breaking hyphen) in compound terms you don't want to wrap (e.g. "Charge&#8209;code", "Day&#8209;1").

---

## 5. Icons & illustrations (reusable SVG symbols)

Define icons once as `<symbol viewBox="0 0 48 48">` in a hidden `<defs>` at the top of `<body>`, then
`<use>` them. Stroke/fill inherits `currentColor` so `.card__icon { color: var(--hl-orange) }` makes
them orange.

```html
<svg width="0" height="0" style="position:absolute"><defs>
  <symbol id="icn-doc" viewBox="0 0 48 48"> … paths … </symbol>
  <symbol id="icn-container" viewBox="0 0 48 48"> … </symbol>
  <symbol id="icn-route" viewBox="0 0 48 48"> … </symbol>
  <symbol id="icn-dollar" viewBox="0 0 48 48"> … </symbol>
  <symbol id="icn-tag" viewBox="0 0 48 48"> … </symbol>
  <symbol id="icn-globe" viewBox="0 0 48 48"> … </symbol>
  <!-- larger industry illustrations, used inline on story slides -->
  <symbol id="vessel" viewBox="0 0 240 120"> … </symbol>
  <symbol id="wave"   viewBox="0 0 240 40"> … </symbol>
  <symbol id="port"   viewBox="0 0 200 200"> … </symbol>
</defs></svg>

<span class="card__icon"><svg><use href="#icn-doc"/></svg></span>   <!-- 44×44, orange -->
```
For revenue/forecast decks, prefer **industry-native** illustrations built inline as SVG (vessels,
cranes, containers, waves, currency tokens) in the orange-stroke / slate-fill convention — never stock
photos (other than the cover).

---

## 6. Charts — hand-built SVG, locked palette, no overlap

Charts are **inline SVG inside a `.viz` frame**, drawn with **manually computed coordinates** (no chart
library). The frame:

```css
.viz { background: #fff; border: 1px solid var(--hl-slate-200); padding: 18px 22px; display: flex; flex-direction: column; gap: 10px; overflow: hidden; min-height: 0; }
.viz--surface { background: var(--hl-slate-100); }
.viz__svg { flex: 1 1 auto; min-height: 0; }
.viz__svg svg { width: 100%; height: 100%; display: block; }
.viz__legend { display: flex; gap: 22px; flex-wrap: wrap; font-size: 11px; color: var(--hl-slate-700); align-items: center; }
.viz__legend span { display: inline-flex; align-items: center; gap: 8px; }
.viz__sw { width: 14px; height: 12px; display: inline-block; }

/* SVG primitive classes — use these, never inline ad-hoc colours */
svg .axis-line { stroke: var(--hl-slate-300); stroke-width: 1; }
svg .grid-line { stroke: var(--hl-slate-200); stroke-width: 1; }
svg .tick   { font-size: 11px; fill: var(--hl-slate-500); }
svg .lbl    { font-size: 12px; fill: var(--hl-slate-700); }
svg .lbl-b  { font-size: 12px; font-weight: 700; fill: var(--hl-ink); }
svg .num    { font-size: 11px; font-weight: 700; fill: var(--hl-ink); }
svg .num-o  { fill: var(--hl-orange); }            /* highlighted value label */
svg .bar-slate  { fill: var(--hl-slate-700); }     /* default series */
svg .bar-light  { fill: var(--hl-slate-300); }     /* secondary series */
svg .bar-orange { fill: var(--hl-orange); }        /* THE highlight series (use sparingly) */
svg .bar-tint   { fill: var(--hl-orange-tint); }
```

**Frame + SVG markup:**
```html
<div class="viz">
  <div class="viz__svg">
    <svg viewBox="0 0 1720 320" preserveAspectRatio="xMidYMid meet"> … </svg>
  </div>
  <div class="viz__legend">
    <span><i class="viz__sw" style="background:var(--hl-orange)"></i>Forecast</span>
    <span><i class="viz__sw" style="background:var(--hl-slate-700)"></i>Actual</span>
  </div>
</div>
```

**Chart rules (all from the deck):**
1. **Always `viewBox="0 0 W H"` + `preserveAspectRatio="xMidYMid meet"`**, and CSS `width:100%; height:100%`. The SVG scales to its container and never overflows. Pick a viewBox aspect close to the container (e.g. wide bars `1720×320`, square-ish `880×540`).
2. **Manually map data → pixels.** Bars: `y = chartH − (val − min)/(max − min) × usableH`; bar `height = chartH − y − originOffset`. Donut: precompute each slice path with sin/cos — `M (startOuter) A R R 0 large-arc 1 (endOuter) L (startInner) A r r 0 large-arc 0 (endInner) Z`, `large-arc-flag = 1` if slice > 180° else 0. Then **read the rendered output back against the data** to verify.
3. **Palette only:** highlight series `.bar-orange` (one), others `.bar-slate` / `.bar-light`; axes `.axis-line`; grid `.grid-line`; ticks `.tick` 11px slate-500; value labels `.num` 11px bold ink, highlighted `.num-o` orange.
4. **Value labels carry context** — `$1,377.57` over `Day 0 · 2025-06-30`, not just `$1,378`. Money ≥ 2 decimals; TEU integer with separators; % ≥ 2 decimals; use `&minus;` for negatives.
5. **No overlap / no clipping:** leave margin inside the viewBox for axis labels and end annotations; don't let text run under bars or off the edge. Eyeball the rendered SVG.
6. Diagrams (process flow, funnel, timeline) follow the same convention: orange dashed `stroke-dasharray="4 4"` separators, orange `FILTER N` eyebrow text (11px, 0.10em, `--hl-orange`), ink titles (13px bold), slate captions (11px), orange arrowheads. (See the data-funnel slide as the template.)

---

## 7. The JS scaler — paste verbatim (critical, and the #1 bug to avoid)

`transform: scale(calc(...))` is **invalid** (scale needs a unitless number; a `calc()` returning a
length fails silently → the slide renders at native 1920px and overflows every viewport). Use this JS
scaler that sets a CSS variable per slide:

```html
<script>
  // Each .slide is a fixed 1920×1080 canvas; scale it to its wrapper's actual width
  // and apply via the CSS variable so transform: scale(var(--scale)) is always a number.
  function rescale() {
    var wraps = document.querySelectorAll('.slide-wrap');
    for (var i = 0; i < wraps.length; i++) {
      var w = wraps[i].clientWidth;
      var slide = wraps[i].querySelector('.slide');
      if (slide) slide.style.setProperty('--scale', String(w / 1920));
    }
  }
  document.addEventListener('DOMContentLoaded', rescale);
  window.addEventListener('load', rescale);
  window.addEventListener('resize', rescale);
</script>
```

Print support (1920×1080 pages, no scaling):
```css
@media print {
  @page { size: 1920px 1080px; margin: 0; }
  body { background: #fff; }
  .deck { gap: 0; padding: 0; }
  .slide-wrap { width: 1920px; aspect-ratio: auto; height: 1080px; page-break-after: always; }
  .slide { transform: none; }
}
```

---

## 8. Layout / "beautiful, aligned, no overlap" rules

1. **The 4-row grid absorbs free space** (`grid-template-rows: 4px auto 1fr auto`) — the `1fr` body row, never the chrome.
2. **Every container clips:** `.body`, `.card`, `.viz`, `.title-text` all have `overflow: hidden; min-height: 0`. `min-height: 0` is what lets grid/flex children actually shrink.
3. **Grid columns use `minmax(0, 1fr)`** (never plain `1fr`) so columns can shrink below content width → no horizontal overflow.
4. **Cards: `--stretch`** when content fills (push the stat to the bottom with `margin-top: auto`); **`--start`** when sparse. If a card is >~60% empty, add substance (bullets, a stat, a mini-viz) or restructure the body into a top viz row + bottom card row (`grid-template-rows: minmax(0,1.1fr) minmax(0,1fr)`).
5. **One idea per slide.** If it answers two questions, split it.
6. **Titles** have `max-width` + `overflow-wrap: break-word` → they wrap, never escape.
7. **Numbers:** money 2 decimals; TEU integer + thousands separators; % 2 decimals; `&minus;` for minus signs; every time-point label carries its date.
8. **Verify in a real browser at 1920 / 1280 / 768px** before shipping — the CSS fails silently in subtle ways if you only check one width. Confirm: nothing overlaps, nothing clipped, every delta box inside bounds, Y-axis zoom makes small variation visible, every series visually distinct.

---

## 9. Content & language discipline (what the audience reads)

- **Every number verified** against the live source; a **footer source line** on every slide names the provenance (table/run) — the ONLY place engineering names are allowed.
- **Banned in visible text:** SQL, Python, Databricks, table/job/script names, "Rule N", CLAUDE.md, AI/Claude/tool names, internal jargon. Business language only. (Same as CLAUDE.md Rule J / Rule 53a — pre-commit grep the rendered text.)
- **Titles read like a conclusion**, sentence case, third person. No "for management", no emoji, no "!".
- **Story arc:** open with the question (slide 2), then data → mechanism → validation → results → next steps; one question per slide; explicitly cover any Q&A the audience raised.

---

## 10. Build → validate → ship checklist

1. Start from the §1 skeleton; paste §2 + §3 + §4 + §6 + §7 CSS/JS **verbatim**.
2. Build the cover (§4), then one `<section class="slide">` per idea (§3 chrome + body).
3. Charts hand-built per §6; icons/illustrations per §5.
4. **Structural check:** `slide-wrap` count == `slide` count; count `data:image` (== embedded assets); zero unresolved placeholders; zero external paths (`../`, `./assets/`); the JS scaler present.
5. **Every number** greps back to a verified source; **banned-term grep** on the rendered text passes (§9).
6. **Open in a real browser at 1920 / 1280 / 768px** (§8.8): no overlap, no clipping, no empty-card syndrome, charts legible, palette correct.
7. Only then commit. A slide deck is a Board-facing artefact — same gate as the HTML-report rules.

---

## 11. Source-of-truth files

| File | Role |
|---|---|
| `docs/NetFreightRevenue_SlideDeck_charge_code.html` | **The canonical deck.** This guide is extracted from it; match it exactly. |
| `docs/NetFreightRevenue_SlideDeck.html` | Sibling deck (cohort/category level), same design system. |
| `DesignSystem/hapag-lloyd-design-system/project/colors_and_type.css` | Brand tokens (already inlined in §2). Copy values; never `<link>` to it. |
| `DesignSystem/hapag-lloyd-design-system/project/assets/hero-hamburg-port.jpeg` | Cover hero — base64-embed it. |
| `DesignSystem/hapag-lloyd-design-system/project/assets/logo-hl-white.svg` | White cover logo — base64-embed it. |
| `CLAUDE.md` → "Mandatory Rules for Building HTML Slide Decks" + Rules 48–58 | The discipline rules this guide implements. |

> **One-line mandate:** given a topic + the text/numbers, paste the §2/§3/§4/§6/§7 blocks verbatim,
> fill the cover + one slide per idea, hand-build charts in the locked palette, run the §10 checklist —
> and the deck will look exactly like `NetFreightRevenue_SlideDeck_charge_code.html`.
