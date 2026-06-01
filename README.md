# Beamer PDF Deduplicator

Remove redundant intermediate/transition pages from LaTeX Beamer PDFs.  
When you compile a Beamer presentation with animations (`\pause`, `\onslide`, `\item<n->`, overlay specifications), the resulting PDF contains many near-duplicate pages — one for each animation step. This tool strips those out, leaving only the **final complete state** of each slide.

## Quick Start

```bash
# Install dependency
pip install PyMuPDF

# Run on your beamer PDF
python3 dedup_beamer.py lecture.pdf

# Preview without writing (recommended first!)
python3 dedup_beamer.py lecture.pdf --dry-run
```

Output defaults to `<input>_clean.pdf`.

## How It Works

The script compares each adjacent pair of pages (page i vs page i+1) after stripping ephemeral footers like page numbers. A four-level detection cascade decides whether page i is a transitional half-state that should be removed:

| Level | Condition | Interpretation |
|-------|-----------|----------------|
| **1 — Exact Duplicate** | `text_i == text_{i+1}` | Same slide rendered on multiple pages (only page number differs) |
| **2 — Strict Subset** | `text_i ⊂ text_{i+1}` | Classic `\pause` — content accumulates across pages |
| **3 — Word-Set Subset** | `words_i ⊂ words_{i+1}` | Reordered headers, e.g. `"Outline"` → `"Overview Outline"` |
| **4 — Near-Duplicate** | Similarity ≥ 98% + safety checks pass | Text-extraction artifacts / formula rendering differences |

If none of the above holds, page i has unique content → **KEEP**.

### Safety Overrides

To prevent false positives, a removal is **vetoed** (page kept) if any of these fire:

- Page i has text blocks with <50% word overlap against page i+1 *(fuzzy matching — tolerates text inserted mid-block)*
- Page i has images not present in page i+1 *(compared by PDF xref)*
- Page i has ≤10 characters of text *(empty / image-only pages)*

The **last page is always kept**.

## Usage

```
usage: dedup_beamer.py [-h] [--dry-run] [--verbose] input [output]

positional arguments:
  input            Input PDF file path
  output           Output PDF file path (default: <input>_clean.pdf)

optional arguments:
  --dry-run        Analyze and report without writing output
  --verbose, -v    Show detailed per-page diagnostics
```

### Examples

```bash
# Basic dedup
python3 dedup_beamer.py lecture.pdf

# Custom output path
python3 dedup_beamer.py lecture.pdf clean_lecture.pdf

# Dry run — see what would be removed before committing
python3 dedup_beamer.py lecture.pdf --dry-run

# Verbose mode — inspect block counts, similarity scores per page
python3 dedup_beamer.py lecture.pdf --verbose
```

## Example Output

```
======================================================================
Beamer PDF Deduplicator
======================================================================
Input:  lecture.pdf
Output: lecture_clean.pdf
Pages:  376
======================================================================

Phase 1: Extracting text, blocks, and image metadata...
  Done. Extracted data from 376 pages.

Phase 2: Analyzing adjacent page pairs...

  ✓ Page   1  →  KEEP    [HIGH  ] has unique content not in next page
  ✗ Page   2  →  REMOVE  [HIGH  ] exact duplicate
  ✗ Page   3  →  REMOVE  [HIGH  ] strict text subset (pause transition)
  ✓ Page   4  →  KEEP    [HIGH  ] has unique content not in next page
  ...

======================================================================
Summary
======================================================================
  Pages kept:     189
  Pages removed:  187
  Reduction:      49.7%

  Removal reasons:
    Exact duplicates:         139
    Strict text subsets:       44
    Word-set subsets:           1
    Near-duplicates:            3

  ⚠  Non-HIGH-confidence removals (3):
    Page   3 [MEDIUM]: near-duplicate (similarity=1.0000)
    Page 132 [MEDIUM]: near-duplicate (similarity=0.9831)
    Page 345 [MEDIUM]: near-duplicate (similarity=1.0000)

  Output written to: lecture_clean.pdf
  Input size:  2.1 MB
  Output size: 1.8 MB
======================================================================
```

## Requirements

- Python 3.7+
- [PyMuPDF](https://pypi.org/project/PyMuPDF/) (`pip install PyMuPDF`)

## Limitations

- **Text-based comparison only.** Purely visual transitions (e.g., an arrow appearing on a diagram without text change) will not be detected as duplicates. In practice, LaTeX Beamer's `\pause` and `\item<n->` always affect text, so this covers the vast majority of cases.
- **Scanned/image PDFs** will not be processed (no extractable text to compare).
- **Formula-heavy slides** may produce MEDIUM-confidence results due to PDF rendering artifacts in math expressions. These are still removed but flagged for review.

## License

MIT
