# Conversion Notes: Bayley (1881) — The National Loans of the United States

## Project

Converting the 212-page 1881 US Treasury document from PDF to high-fidelity MyST Markdown,
targeting GitHub Pages publication. Adapted from the workflow in `book-sargent-1993-.../PROMPT-PDF-TO-MD.md`.

**Source**: `source/Bayley.pdf` (14 MB, 212 pages)
**Author**: Rafael A. Bayley, Treasury Department
**Publisher**: Government Printing Office, Washington, 1881
**Prepared for**: Tenth Census of the United States
**Digitized by**: Cornell University Library → Internet Archive

---

## Environment

- Conda env: `pdf_structuring` (Python 3.11)
- marker-pdf 1.10.2 (with torch 2.11.0, surya-ocr 0.17.1)
- mystmd: TBD (pending Node install)

---

## Step 1: marker-pdf Extraction

- **Started**: 2026-04-15
- **Command**: `marker_single source/Bayley.pdf --output_dir _archive/marker_output/`
- **Runtime**: _TBD_
- **Output**: _TBD_

### Extraction Quality Assessment
_To be filled after extraction completes._

---

## Step 2: Project Structure (Planned)

Split Part I (Historical, by loan) into 5 time-period files rather than 50+ per-loan sections.

```
paper/
  ch00_frontmatter.md          # Title page, letters of transmittal
  ch01_revolutionary.md         # 1776–1790 (Revolutionary War + early federal)
  ch02_early_republic.md        # 1790–1860 (19th c. early/mid)
  ch03_civil_war.md             # 1861–1865 (Civil War issues)
  ch04_postwar_resumption.md    # 1865–1880 (Reconstruction + specie resumption)
  ch05_recapitulation_tables.md # Summary tables (hardest section)
  ch06_index.md                 # Index (optional)
```

---

## Key Differences from Sargent (1993) Workflow

| Aspect | Sargent | Bayley | Impact |
|--------|---------|--------|--------|
| Equations | ~130 | 0 | Drop `$$...$$` labeling work |
| BibTeX refs | 198 | ~0 | `references.bib` minimal / skip |
| Tables | 0 | Many | **Main workload** — manual rebuild |
| Footnotes | 158 | Few | Easy |
| OCR source | Modern print | 1881 print, scanned | **Heavy OCR cleanup** |
| Chapters | 7 clear | 50+ loan sections | Grouped by decade |

---

## OCR Errors Noted
_To be tracked during conversion._

---

## Step 3: MyST Conversion Progress

| File | Status | Lines | Notes |
|------|--------|-------|-------|
| ch00_frontmatter.md | Pending | — | — |
| ch01_revolutionary.md | Pending | — | — |
| ch02_early_republic.md | Pending | — | — |
| ch03_civil_war.md | Pending | — | — |
| ch04_postwar_resumption.md | Pending | — | — |
| ch05_recapitulation_tables.md | Pending | — | Hardest — all tables |
| ch06_index.md | Pending | — | Optional |

---

## Step 4: Build & Deploy

- [ ] `myst build --html` passes
- [ ] All cross-references resolve
- [ ] GitHub repo created
- [ ] `.github/workflows/deploy.yml` configured
- [ ] GitHub Pages enabled (source: GitHub Actions)
- [ ] Public URL: _TBD_
