---
name: ingest-extract
description: Parse a user-uploaded document of ANY type (Markdown, PDF, text, Excel, Word, PowerPoint, CSV, images, etc.) and extract ALL of its content faithfully into Markdown. Load this for phase 1. The rule is fidelity — capture everything exactly, invent nothing, flag anything unclear as an OPEN QUESTION.
---

# ingest-extract — faithful extraction from any document

Goal: turn the uploaded file(s) into a **complete, exact Markdown extraction** that
later phases build on. **Fidelity over polish.** Do not summarize, round, reorder,
or interpret yet — just capture.

## Pick the right tool per format, then VERIFY the output
| Format | Preferred → fallback |
|---|---|
| Markdown / `.txt` / `.csv` | read directly; for CSV, render as a Markdown table preserving every cell |
| PDF | your **`pdf`** skill → `pdftotext -layout` → `markitdown` (for scanned PDFs, OCR via the pdf skill; if still unreadable, OPEN QUESTION) |
| Word `.docx`/`.doc` | your **`docx`** skill → `markitdown` → `soffice --headless --convert-to markdown` |
| Excel `.xlsx`/`.xls` | your **`xlsx`** skill → `pandas.read_excel(sheet_name=None)` / `openpyxl` — capture EVERY sheet, header, and used cell, including formulas' computed values |
| PowerPoint `.pptx` | your **`pptx`** skill → `markitdown` (capture every slide's text + table + notes) |
| Images | the multimodal read / OCR; transcribe text and describe any chart only as far as the data is legible |
| Anything else | `markitdown` (`uv tool install markitdown` or `pip install markitdown` if absent); if it can't, say so and ask the user |

Always **read the extraction back against the source** to confirm nothing was
dropped or altered. If a tool is unavailable and can't be installed, report it —
don't substitute a guess.

## What to capture (all of it)
- Every **number** with its exact value, **unit**, and sign — never round or
  reformat at this stage.
- Every **table** in full (all rows/columns), as a Markdown table.
- Headings, labels, dates, footnotes, definitions, and any stated assumptions.
- **Provenance** for each fact: page / sheet / slide / section, so later phases
  can cite a source line.

## OPEN QUESTIONS (the NEVER-GUESS valve)
Maintain an explicit `## OPEN QUESTIONS` list for anything that is:
illegible, truncated/cut-off, ambiguous (which unit? which period? which entity?),
internally inconsistent (two numbers disagree), or missing but needed for a deck.
Do **not** resolve these yourself — the orchestrator will ask the user.

## Large documents (avoid model/gateway limits)
If the file is large (many pages/sheets/rows), do NOT extract it in one giant pass —
that hits the Databricks gateway's per-request token limit. Extract **per section /
per sheet / per page range**, writing each partial extraction to a file as you go
(checkpoint), then assemble. On a rate-limit/quota error, back off and retry; on a
context/max-tokens error, split into smaller parts. **Never drop content to fit a
limit** — report what remains so the orchestrator can chunk it. (See the
`resilience` skill.)

## Output shape
```
# Extraction — <source filename(s)>
## Source map
- <file> — <pages/sheets/slides>, <doc type>, <tool used>
## Content
<faithful Markdown: sections, full tables, exact numbers, with provenance notes>
## OPEN QUESTIONS
- <each gap/ambiguity, with where it occurs>
```
Return this extraction and the OPEN QUESTIONS. Build nothing else this phase.
