# PDF-to-MyST Automated Conversion Pipeline

When the user asks you to convert a PDF in this folder (or says "process the PDF", "convert the book", etc.), execute the following pipeline end-to-end. Do NOT ask for confirmation between steps — just run everything and report the final result.

## Step 0: Detect the PDF

- Scan the current working directory for any `.pdf` file.
- If multiple PDFs exist, ask the user which one to convert.
- If no PDF is found, tell the user to place a PDF in this folder.
- Read the first 10 pages of the PDF using `pypdfium2` to extract: title, author, page count, and a rough table of contents.

## Step 1: Environment Setup

Check and install dependencies as needed. Do NOT ask — just do it.

```
# Check for conda env "pdf_structuring"
conda activate pdf_structuring 2>/dev/null
# If it doesn't exist, create it:
conda create -n pdf_structuring python=3.11 -y
conda activate pdf_structuring
pip install marker-pdf

# Check for mystmd
myst --version
# If not found:
npm install -g mystmd

# Check for gh CLI
gh --version
# If not found:
brew install gh
```

## Step 2: Project Structure Setup

Create the following directory structure in the current working directory:

```
paper/
  figures/
source/
_archive/
  marker_output/
```

- Copy the PDF into `source/`.
- Create `.gitignore` with: `_build/`, `.venv/`, `__pycache__/`, `*.pyc`, `.DS_Store`
- Create `.github/workflows/deploy.yml` using this template:

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

## Step 3: Run marker-pdf Extraction

```bash
conda run -n pdf_structuring marker_single source/<filename>.pdf --output_dir _archive/marker_output/
```

- Run this in the background. It takes 15–90 minutes depending on page count.
- While waiting, use `caffeinate -i -s &` to prevent sleep on macOS.
- After completion, copy extracted images: `cp _archive/marker_output/<name>/*.{jpeg,png} paper/figures/ 2>/dev/null`

## Step 4: Assess Extraction Quality and Plan Chapters

After marker completes:

1. Read the raw `.md` output in `_archive/marker_output/`.
2. Run `grep -n "^#" _archive/marker_output/<name>/<name>.md` to find all headings.
3. Determine the book's structure and decide how to split into chapter files.
4. Classify the book type to calibrate OCR cleanup expectations:
   - **Modern academic** (post-1970): equations + citations, moderate OCR effort
   - **Historical** (pre-1950): heavy OCR errors, many garbled names/numbers
   - **Technical/financial**: heavy tables (marker often fails on these)
   - **Literary/narrative**: mostly prose, lightest effort

Rules for splitting:
- Books ≤ 30 pages: single `paper/paper.md`
- Books > 30 pages: split into `paper/ch00_frontmatter.md`, `paper/ch01_<slug>.md`, etc.
- Aim for 5–10 files total. Group by chapters, parts, or logical periods.
- Note the line ranges in the raw markdown for each planned chapter.

## Step 5: Create myst.yml, index.md, and Chapter Files

Create `myst.yml` with the detected book metadata and planned TOC:

```yaml
version: 1
project:
  title: "<detected title>"
  authors:
    - name: <detected author>
  date: <publication date>
  toc:
    - file: index.md
    - title: <Book Title>
      children:
        - file: paper/ch00_frontmatter.md
          title: "Front Matter"
        - file: paper/ch01_<slug>.md
          title: "<Chapter 1 Title>"
        # ... one entry per chapter
site:
  template: book-theme
  options:
    base_url: /<repo-name>
```

Create `index.md` with a landing page describing the book.

## Step 6: Convert Each Chapter to MyST Markdown

For each chapter, read the corresponding line range from the raw marker output and write a clean MyST `.md` file. Apply these rules:

### OCR Cleanup Rules
- Fix ALL OCR errors. Common patterns:
  - `<sup>I</sup>` → `I` (OCR artifact from old "I" letterforms)
  - `tlie` → `the`, `witli` → `with`, `bo` → `be`, `aud` → `and`
  - Fix garbled proper names, place names, institutions
  - For pre-1900 texts: expect `E`↔`F`, `I`↔`1`↔`l`, `O`↔`0`, `rn`↔`m` confusion
- **NEVER rewrite or paraphrase**. Fix only OCR errors.

### MyST Formatting Rules
- Add YAML frontmatter with `title` to each chapter file
- Section labels: `(sec-descriptive-slug)=` before each `##` heading
- Footnotes: convert `<sup>*</sup>`, `<sup>†</sup>`, `<sup>‡</sup>` etc. to sequential `[^fnN]`
  - Number continuously across chapters (ch01 ends fn35 → ch02 starts fn36)
- Tables:
  - Clean small tables: fix `<br>` artifacts into proper markdown tables
  - Empty/garbled large tables: use placeholder:
    ```
    :::{admonition} Table: [description]
    *[To be transcribed from the original PDF]*
    :::
    ```
- Equations (if applicable): `$$ ... $$ (eq-label)`
- Block quotes (`>`) for letters and quoted passages
- Figures: `:::{figure} figures/<filename>` directive
- Citations (if applicable): `{cite}`key`` + create `paper/references.bib`

### Processing Strategy
- Process one chapter at a time.
- For very large chapters (>500 lines of raw text), split into two parts, convert separately, then merge.
- After writing each chapter, immediately run `myst build --html` to catch errors.
- Fix any build errors before moving to the next chapter.

## Step 7: Build and Verify

```bash
myst build --html
```

- All pages should build with zero `⛔️` errors.
- Warnings (`⚠️`) about Unicode characters are acceptable.
- Fix any duplicate labels, missing files, or broken references.

## Step 8: Create README.md

Create a `README.md` describing:
- Book title, author, year
- Source of the PDF
- Project structure
- How to build locally (`myst build && myst start`)

## Step 9: Git Commit, Create Repo, Deploy

```bash
git init && git branch -m main
git add -A
git commit -m "Initial MyST conversion of <Book Title>"
```

Then ask the user: **"Ready to publish to GitHub Pages? I'll create a public repo and deploy. OK?"**

If yes:
```bash
gh repo create <repo-slug> --public --source=. --push \
  --description "MyST Markdown edition of <Book Title>"
gh api repos/<username>/<repo-slug>/pages -X POST -f "build_type=workflow"
```

Wait for the GitHub Actions workflow to complete, then report the live URL:
```
https://<username>.github.io/<repo-slug>/
```

## Step 10: Report Summary

After everything is done, report to the user:

1. Total pages processed
2. Number of chapters created
3. Number of OCR corrections made (rough estimate)
4. Number of table placeholders (tables that need manual transcription)
5. The GitHub repo URL
6. The live site URL
7. Any known issues or items needing manual attention

---

## Reference: MyST Syntax

```markdown
# Section label
(sec-slug)=
## Heading

# Equation
$$
x = y + z
$$ (eq-label)

# Footnote
Text.[^fn1]
[^fn1]: Content.

# Figure
:::{figure} figures/img.jpeg
:name: fig-name
:width: 80%
Caption text
:::

# Table
| Col A | Col B |
|-------|-------|
| val1  | val2  |

# Table placeholder
:::{admonition} Table: Description
*[To be transcribed from the original PDF]*
:::

# Block quote
> Quoted text.

# Citation
{cite}`author1990`
```
