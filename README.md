# The National Loans of the United States, 1776–1880

**Rafael A. Bayley (1881)** — MyST Markdown edition of the U.S. Treasury report prepared for the Tenth Census.

## About

- Original: Bayley, R. A. (1881). *The National Loans of the United States, from July 4, 1776, to June 30, 1880*. Washington: Government Printing Office.
- Source PDF: [Internet Archive](http://www.archive.org/details/cu31924030228245) via Cornell University Library.
- This repository contains a MyST Markdown conversion with companion tooling.

## Project Structure

```
├── paper/                 # MyST Markdown source, one file per historical period
│   ├── ch00_frontmatter.md
│   ├── ch01_revolutionary.md
│   ├── ch02_early_republic.md
│   ├── ch03_civil_war.md
│   ├── ch04_postwar_resumption.md
│   ├── ch05_recapitulation_tables.md
│   └── figures/
├── source/                # Original PDF
├── _archive/              # Raw marker-pdf extraction output
├── myst.yml               # MyST site configuration
├── index.md               # Landing page
├── NOTES.md               # Conversion log
└── .github/workflows/     # GitHub Pages deploy
```

## Build locally

```bash
conda activate pdf_structuring
myst build           # build site
myst start           # local preview at http://localhost:3000
```

## Conversion workflow

See `NOTES.md` for the step-by-step log and the referenced `PROMPT-PDF-TO-MD.md`
(in the sibling `book-sargent-1993-.../` repository) for the general PDF → MyST
prompt template.
