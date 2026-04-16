# Guide: Converting a PDF Book to a MyST Markdown Website

A step-by-step guide for converting any PDF book into a high-fidelity MyST Markdown site, deployable to GitHub Pages. This workflow combines automated PDF extraction (`marker-pdf`) with LLM-assisted cleanup and formatting.

**Estimated time**: 2–6 hours depending on book length and complexity.

---

## Table of Contents

1. [Prerequisites](#1-prerequisites)
2. [Environment Setup](#2-environment-setup)
3. [Project Initialization](#3-project-initialization)
4. [PDF Extraction with marker-pdf](#4-pdf-extraction-with-marker-pdf)
5. [Extraction Quality Assessment](#5-extraction-quality-assessment)
6. [Structure Planning](#6-structure-planning)
7. [LLM-Assisted Conversion to MyST](#7-llm-assisted-conversion-to-myst)
8. [Build and Local Preview](#8-build-and-local-preview)
9. [GitHub Pages Deployment](#9-github-pages-deployment)
10. [Quality Assurance Checklist](#10-quality-assurance-checklist)
11. [Common Pitfalls and Solutions](#11-common-pitfalls-and-solutions)
12. [Appendix: MyST Syntax Quick Reference](#appendix-myst-syntax-quick-reference)

---

## 1. Prerequisites

| Tool | Purpose | Install |
|------|---------|---------|
| **Conda** (or `venv`) | Python environment management | [miniconda](https://docs.conda.io/en/latest/miniconda.html) |
| **Python 3.11** | Required by marker-pdf (3.13 may have compatibility issues) | via conda |
| **Node.js** | Required by MyST | `brew install node` (macOS) or [nodejs.org](https://nodejs.org/) |
| **Git** | Version control | Pre-installed on most systems |
| **GitHub CLI (`gh`)** | Repository creation and deployment | `brew install gh` (macOS) or [cli.github.com](https://cli.github.com/) |
| **An LLM** | OCR cleanup and MyST formatting (Claude Code, Copilot, ChatGPT, etc.) | IDE integration or API |

---

## 2. Environment Setup

**One-time setup** — reuse this environment for all future PDF conversions.

```bash
# Create a dedicated conda environment
conda create -n pdf_structuring python=3.11 -y
conda activate pdf_structuring

# Install marker-pdf (includes PyTorch, surya-ocr — ~3-5 GB download)
pip install marker-pdf

# Install MyST globally (Node.js package, independent of Python)
npm install -g mystmd

# Verify installations
marker_single --help
myst --version

# Authenticate GitHub CLI (one-time)
gh auth login
```

> **Note**: `marker-pdf` downloads large ML models on first run. Ensure ~5 GB of free disk space. If using Apple Silicon (M1/M2/M3), marker will run on CPU — MPS is not supported for the table detection model.

---

## 3. Project Initialization

For each new book, create a fresh project directory:

```bash
# Replace <book-slug> with a short identifier, e.g., "smith-1920-trade-policy"
PROJECT=<book-slug>
mkdir -p $PROJECT/{paper/figures,source,_archive/marker_output}
cd $PROJECT

# Copy your PDF into the source directory
cp /path/to/your/book.pdf source/

# Initialize git
git init && git branch -m main
```

Create the following files:

### `.gitignore`

```
_build/
.venv/
__pycache__/
*.pyc
.DS_Store
.ipynb_checkpoints/
```

### `myst.yml` (starter template — update after structure planning)

```yaml
version: 1
project:
  title: "<Book Title>"
  subtitle: "<Subtitle or edition info>"
  authors:
    - name: <Author Name>
      affiliations:
        - <Affiliation>
  date: <YYYY-MM-DD>
  keywords:
    - <keyword1>
    - <keyword2>
  # bibliography:
  #   - paper/references.bib    # Uncomment if the book has citations
  toc:
    - file: index.md
    - title: Book
      children:
        - file: paper/ch00_frontmatter.md
          title: "Front Matter"
        # Add more chapters after structure planning (Step 6)

site:
  template: book-theme
  options:
    base_url: /<repo-name>
```

### `index.md`

```markdown
---
title: "<Book Title>"
---

# <Book Title>

**<Author>** · <Publisher> · <Year>

<Brief description of what this book is and why it was digitized.>

## About this edition

- **Source**: <Link to original PDF or archive>
- **Conversion**: PDF → MyST Markdown via `marker-pdf` + LLM-assisted cleanup
- **License**: <License information>
```

### `NOTES.md` (conversion log — fill in as you go)

```markdown
# Conversion Notes: <Book Title>

## Project
Converting <page count>-page <description> from PDF to MyST Markdown.

## Step 1: marker-pdf Extraction
- **Date**: <date>
- **Runtime**: _TBD_
- **Output**: _TBD_

### Extraction Quality Assessment
_To be filled after extraction completes._

## Step 2: Structure
_To be filled after reviewing extraction output._

## Step 3: Conversion Progress
| Chapter | File | Status | Lines | Notes |
|---------|------|--------|-------|-------|
| | | | | |

## OCR Errors Noted
_Track corrections made during conversion._
```

---

## 4. PDF Extraction with marker-pdf

```bash
conda activate pdf_structuring

# Run marker extraction (this is the slow step)
marker_single source/<book>.pdf --output_dir _archive/marker_output/
```

### What to expect

| Book size | Approximate runtime (CPU) |
|-----------|---------------------------|
| 50 pages | 15–20 minutes |
| 100 pages | 30–40 minutes |
| 200 pages | 60–90 minutes |
| 400+ pages | 2–3 hours |

> **Tip**: If running on a laptop, prevent sleep during extraction:
> ```bash
> # macOS — keep awake while plugged in
> caffeinate -i -s &
> ```

### Output structure

After extraction, you'll find in `_archive/marker_output/<BookName>/`:

| File | Description |
|------|-------------|
| `<BookName>.md` | Raw extracted Markdown (text + equations + image refs) |
| `<BookName>_meta.json` | Structural metadata |
| `*.jpeg / *.png` | Extracted figures and images |

Copy extracted images to your paper directory:

```bash
cp _archive/marker_output/<BookName>/*.{jpeg,png} paper/figures/ 2>/dev/null
```

---

## 5. Extraction Quality Assessment

Open `_archive/marker_output/<BookName>/<BookName>.md` and assess quality across these dimensions:

| Dimension | What to check | Typical quality |
|-----------|--------------|-----------------|
| **Prose** | Read 3–5 paragraphs from different sections | Modern print: Good. Pre-1950: Fair. Pre-1900: Poor (heavy OCR errors) |
| **Headings** | Are chapter/section titles captured? | Usually good, but OCR may garble them |
| **Equations** | Are `$...$` and `$$...$$` blocks intact? | Modern LaTeX-typeset: Good. Older typesetting: Fair to Poor |
| **Tables** | Are table structures and data present? | Small tables: Partial. Large financial/data tables: Often empty |
| **Figures** | Count extracted images vs. original | Usually good; may miss 2nd figure on same page |
| **Footnotes** | Are they captured? (look for `<sup>` tags) | Early sections: Good. Later sections: Often dropped |
| **References/Bibliography** | Is the reference list extracted? | Usually good (plain text) |

Record your findings in `NOTES.md`. This assessment determines how much manual work is needed.

### Classifying your book type

| Book type | Key characteristics | Expected effort |
|-----------|-------------------|-----------------|
| **Modern academic** (post-1970) | Equations, citations, clean print | Medium — equations and refs need formatting |
| **Historical document** (pre-1950) | OCR errors, old typography, tables | High — heavy OCR cleanup, table reconstruction |
| **Technical manual** | Many tables, diagrams, numbered lists | High — tables are the bottleneck |
| **Literary/narrative** | Mostly prose, few tables/equations | Low — mostly OCR cleanup |

---

## 6. Structure Planning

### Deciding file structure

**Single file** — for documents ≤ ~30 pages:

```
paper/
  paper.md
```

**Multi-chapter** — for documents > ~30 pages (strongly recommended for books):

```
paper/
  ch00_frontmatter.md
  ch01_<slug>.md
  ch02_<slug>.md
  ...
```

### How to split

1. **Scan headings** in the raw markdown:
   ```bash
   grep -n "^#" _archive/marker_output/<BookName>/<BookName>.md | head -60
   ```

2. **Identify natural divisions**: chapters, parts, major sections. Aim for 5–10 files for a typical book.

3. **Note the line numbers** where each chapter begins and ends — you'll use these ranges when feeding content to the LLM.

4. **Update `myst.yml`** with the planned TOC structure:
   ```yaml
   toc:
     - file: index.md
     - title: Book
       children:
         - file: paper/ch00_frontmatter.md
           title: "Front Matter"
         - file: paper/ch01_introduction.md
           title: "1 Introduction"
         - file: paper/ch02_methodology.md
           title: "2 Methodology"
         # ... etc.
   ```

5. **Create placeholder files** for each planned chapter:
   ```bash
   for f in ch00_frontmatter ch01_introduction ch02_methodology; do
     echo -e "---\ntitle: \"Placeholder\"\n---\n\n*To be converted.*" > paper/${f}.md
   done
   ```

---

## 7. LLM-Assisted Conversion to MyST

This is the core work. Feed the raw marker output to an LLM (Claude Code, Copilot, ChatGPT, etc.) **one chapter at a time**, with the following instructions:

### Prompt template

Adapt this prompt for your specific book:

> You are converting raw marker-pdf output into high-fidelity MyST Markdown.
>
> **Source**: `_archive/marker_output/<BookName>/<BookName>.md`, lines [START]–[END].
>
> **Write to**: `paper/chXX_<slug>.md`
>
> **Rules**:
> 1. **Fix all OCR errors**. Common patterns:
>    - `<sup>I</sup>` → `I` (OCR artifact from old letterforms)
>    - `tlie` → `the`, `witli` → `with`, `bo` → `be`
>    - Fix garbled proper names, place names, institutions
>    - `$` before numbers = dollar signs (keep them)
> 2. **Preserve original prose exactly** — only fix OCR, never rewrite or paraphrase
> 3. **Section labels**: `(sec-descriptive-slug)=` before each heading
> 4. **Footnotes**: Convert `<sup>*</sup>`, `<sup>†</sup>` etc. to sequential `[^fnN]` with definitions
> 5. **Tables**: Clean up `<br>` artifacts into proper markdown tables. For empty/garbled tables, use:
>    ```
>    :::{admonition} Table: [description]
>    *[To be transcribed from the original PDF]*
>    :::
>    ```
> 6. **Equations** (if applicable): Use `$$ ... $$ (eq-label)` for numbered equations
> 7. **Block quotes** (`>`) for letters and quoted passages
> 8. **Citations** (if applicable): Use `{cite}`key`` and create `references.bib`
> 9. **Figures**: Use `:::{figure}` directives pointing to `figures/<filename>`
>
> **Frontmatter** (first chapter file only):
> ```yaml
> ---
> title: "<Chapter Title>"
> ---
> ```

### Tips for efficient conversion

- **Work one chapter at a time** — keeps LLM context manageable and errors isolated.
- **Parallel processing**: If using Claude Code or similar, you can run multiple chapters simultaneously via subagents.
- **Footnote numbering**: Keep a running count across chapters (ch01 ends at fn35 → ch02 starts at fn36).
- **Verify OCR corrections**: When unsure about a name or number, cross-check against the original PDF.
- **Build frequently**: Run `myst build` after each chapter to catch formatting errors early.

### Content-type-specific guidance

**For books with equations (academic/scientific)**:
- Use `$$` blocks with labels: `$$ E = mc^2 $$ (eq-einstein)`
- Reference with `{eq}`eq-einstein``
- Check every equation against the PDF — OCR frequently garbles math symbols

**For books with citations**:
- Create `paper/references.bib` with BibTeX entries
- Use `{cite}`authorYear`` in text
- Add `bibliography: paper/references.bib` to `myst.yml`

**For books with heavy tables (financial, statistical)**:
- Small tables: Convert to markdown tables or MyST `list-table`
- Large tables with data: Use `list-table` directive
- Tables where OCR failed: Use admonition placeholder and plan to transcribe later

**For historical documents (pre-1900)**:
- Expect 2–5x more OCR errors than modern print
- Common OCR confusions: `E`↔`F`, `I`↔`1`↔`l`, `O`↔`0`, `rn`↔`m`
- Headings are frequently garbled — verify every heading against the PDF
- Dollar amounts and dates are error-prone — spot-check numerical values

---

## 8. Build and Local Preview

After converting each chapter (or all chapters):

```bash
# Build the HTML site
myst build --html

# Start local preview server
myst start
```

Open `http://localhost:3000` in your browser. Check:

- [ ] All chapters appear in the sidebar navigation
- [ ] Prose renders correctly (no raw HTML artifacts)
- [ ] Equations render (no broken LaTeX)
- [ ] Tables display properly
- [ ] Figures appear at appropriate locations
- [ ] Footnotes are linked and numbered
- [ ] Cross-references resolve (no broken links)
- [ ] No build errors in the terminal output

### Common build issues

| Issue | Solution |
|-------|----------|
| `Table of contents entry does not exist` | File name in `myst.yml` doesn't match actual file name |
| `Duplicate identifier` | Two files use the same `(sec-label)=` — rename one |
| Missing image | Check file exists in `paper/figures/` and path is correct |
| Unicode warnings (em dashes, fractions) | Generally safe to ignore; use ASCII alternatives if needed |

---

## 9. GitHub Pages Deployment

### Create the deployment workflow

Create `.github/workflows/deploy.yml`:

```yaml
name: MyST GitHub Pages Deploy
on:
  push:
    branches:
      - main
env:
  BASE_URL: /${{ github.event.repository.name }}

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Pages
        uses: actions/configure-pages@v3
      - uses: actions/setup-node@v4
        with:
          node-version: 20.x
      - name: Install MyST Markdown
        run: npm install -g mystmd
      - name: Build HTML Assets
        run: myst build --html
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: "./_build/html"
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### Create repo and deploy

```bash
# Stage and commit all files
git add -A
git commit -m "Initial MyST conversion of <Book Title>"

# Create GitHub repository and push
gh repo create <repo-name> --public --source=. --push \
  --description "MyST Markdown edition of <Book Title>"

# Enable GitHub Pages (source: GitHub Actions)
gh api repos/<username>/<repo-name>/pages -X POST -f "build_type=workflow"

# Monitor deployment
gh run list --repo <username>/<repo-name> --limit 1
```

After the workflow completes (~1–2 minutes), your site will be live at:

```
https://<username>.github.io/<repo-name>/
```

### Subsequent updates

After making changes, simply commit and push — the workflow redeploys automatically:

```bash
git add -A
git commit -m "Improve chapter X conversion"
git push
```

---

## 10. Quality Assurance Checklist

After conversion, produce a fidelity report in `NOTES.md` covering:

### Structure
- [ ] All chapters/sections present with correct headings
- [ ] Section labels defined for cross-referencing
- [ ] Table of contents in `myst.yml` matches actual files

### Prose
- [ ] Sample 3–5 paragraphs from different chapters — compare word-for-word with PDF
- [ ] OCR errors corrected (document corrections in NOTES.md)
- [ ] Italics, bold, and emphasis match original

### Equations (if applicable)
- [ ] Count matches original
- [ ] LaTeX renders correctly in `myst build`
- [ ] Cross-references resolve

### Tables
- [ ] Count matches original (including sub-tables)
- [ ] Numerical values spot-checked (3–5 values per table)
- [ ] Tables with placeholder admonitions listed for future transcription

### Figures
- [ ] Count matches original
- [ ] Images display correctly
- [ ] Captions match original

### Footnotes
- [ ] Count matches original
- [ ] Content matches original
- [ ] Numbered sequentially

### Citations & References (if applicable)
- [ ] All in-text citations use `{cite}` syntax
- [ ] All entries present in `references.bib`

### Build
- [ ] `myst build` completes without errors
- [ ] HTML output renders correctly
- [ ] GitHub Pages deployment successful

---

## 11. Common Pitfalls and Solutions

| Pitfall | Cause | Solution |
|---------|-------|----------|
| marker-pdf takes hours | Large PDF, CPU-only processing | Use `caffeinate` to prevent sleep; plan for the wait |
| Empty tables in extraction | marker-pdf loses complex/large tables | Transcribe manually from PDF; use admonition placeholders |
| Missing footnotes in later chapters | marker-pdf drops footnotes deep in documents | Scan PDF page-by-page for footnote markers |
| Garbled equations | OCR misreads math symbols | Compare each equation against PDF; rewrite LaTeX by hand |
| Duplicate section labels | Copy-paste across chapters | Use chapter-prefixed labels: `sec-ch02-introduction` |
| `myst build` errors on tables | `list-table` is whitespace-sensitive | Test each table individually |
| GitHub Pages 404 | Missing `base_url` configuration | Set `site.options.base_url: /<repo-name>` in `myst.yml` |
| LLM rewrites prose | LLM "improves" original text | Explicitly instruct: "fix only OCR errors, never rewrite" |
| Python 3.13 incompatibility | marker-pdf/PyTorch may not support latest Python | Use Python 3.11 via conda |
| Second figure on page missing | marker-pdf extracts only one per page sometimes | Manually extract from source PDF |
| Pre-1900 OCR extremely poor | Old typography, scanned copies | Budget 2–5x more time for OCR cleanup |

---

## Appendix: MyST Syntax Quick Reference

```markdown
# YAML Frontmatter
---
title: "Chapter Title"
authors:
  - name: Author Name
bibliography: references.bib
---

# Abstract (for papers)
+++ {"part": "abstract"}
Abstract text here.
+++

# Section with label (for cross-referencing)
(sec-my-section)=
## 1. My Section Title

# Equation with label
$$
x = y + z
$$ (eq-my-equation)

# Inline math
The variable $x$ represents...

# Citation
{cite}`author1990`

# Cross-references
{ref}`sec-my-section`
{eq}`eq-my-equation`
{numref}`tbl-my-table`
{numref}`fig-my-figure`

# Footnote
Some text.[^fn1]
[^fn1]: Footnote content here.

# Figure
:::{figure} figures/my_figure.png
:name: fig-my-figure
:width: 80%
Figure 1: Caption text
:::

# Table (list-table)
:::{list-table} Table 1: Caption
:header-rows: 1
:name: tbl-my-table

* - Col A
  - Col B
* - val1
  - val2
:::

# Admonition (for placeholders, notes, warnings)
:::{admonition} Note
Content here.
:::

# Block quote
> Quoted text here.
>
> — Attribution

# Definition / Theorem
:::{admonition} Definition: Term
Formal definition text.
:::
```

---

## Lessons Learned

### From Sargent (1993) — 204-page academic book
- Split into per-chapter files — essential for LLM context management
- Chapters can be converted in parallel via subagents
- marker-pdf occasionally misses the second figure from a page
- `BASE_URL` in `myst.yml` is needed for GitHub Pages

### From Bayley (1881) — 212-page historical government document
- Pre-1900 print produces extremely dense OCR errors
- Large financial tables are almost never extracted successfully by marker-pdf
- The book structure may not follow modern chapter conventions — group by logical periods
- No equations or citations needed — adapt the workflow to the actual content
- Prose conversion is still valuable even with table placeholders

### General principles
1. **marker-pdf gets you 60–70%** of the way. Plan for significant manual cleanup.
2. **Tables are always the bottleneck.** Budget extra time for books with many tables.
3. **Build early, build often.** Run `myst build` after each chapter.
4. **Section-by-section QA is essential.** A full-pass conversion produces a good draft, but comparison against the original PDF catches missing content.
5. **The conversion log (NOTES.md) is a deliverable.** It creates an auditable record.
6. **One environment, many books.** The conda environment and tools are reusable.

---

*This guide was developed through the conversion of Sargent (1993) "Bounded Rationality in Macroeconomics" and Bayley (1881) "The National Loans of the United States". For questions or improvements, open an issue on the repository.*
