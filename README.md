# Beamer PDF Deduplicator

Remove redundant intermediate/transition pages from LaTeX Beamer PDFs.

When you compile a Beamer presentation with animations (`\pause`, `\onslide`, `\item<n->`, overlay specifications), the resulting PDF contains many near-duplicate pages — one for each animation step. This tool strips those out, leaving only the **final complete state** of each slide.

## Table of Contents

- [Setup](#setup)
  - [Prerequisites](#prerequisites)
  - [Clone the repository](#clone-the-repository)
  - [Create a virtual environment (recommended)](#create-a-virtual-environment-recommended)
  - [Install dependencies](#install-dependencies)
  - [Verify installation](#verify-installation)
- [Quick Start](#quick-start)
- [How It Works](#how-it-works)
  - [Detection cascade](#detection-cascade)
  - [Safety overrides](#safety-overrides)
- [Usage](#usage)
  - [Command-line arguments](#command-line-arguments)
  - [Examples](#examples)
- [Example Output](#example-output)
- [Troubleshooting](#troubleshooting)
- [Limitations](#limitations)
- [License](#license)

## Setup

### Prerequisites

You need the following installed on your system before you can use this tool:

| Requirement | Minimum Version | How to Check |
|-------------|-----------------|--------------|
| **Python**  | 3.7+            | `python3 --version` |
| **pip**     | (bundled with Python 3.4+) | `pip3 --version` |
| **Git**     | any recent version | `git --version` |

If you don't have Python 3.7+, install it via one of the following:

- **macOS (Homebrew):** `brew install python3`
- **Ubuntu/Debian:** `sudo apt install python3 python3-pip python3-venv`
- **Windows:** Download the installer from [python.org](https://www.python.org/downloads/) (check "Add Python to PATH" during installation)
- **Other Linux:** Use your distribution's package manager (e.g., `dnf`, `pacman`, `zypper`)

### Clone the repository

```bash
git clone https://github.com/reeseliu/beamer-pdf-dedup.git
cd beamer-pdf-dedup
```

> If you downloaded the ZIP from GitHub instead, extract it and open a terminal inside the extracted folder.

### Create a virtual environment (recommended)

Using a virtual environment keeps the project's dependencies isolated from your system Python and other projects. This step is technically optional but **strongly recommended**.

#### macOS / Linux

```bash
python3 -m venv venv
source venv/bin/activate
```

You should see `(venv)` appear at the beginning of your terminal prompt — this means the virtual environment is active.

#### Windows (Command Prompt)

```cmd
python -m venv venv
venv\Scripts\activate
```

#### Windows (PowerShell)

```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
```

> **Note:** If PowerShell blocks the activation script, run the following first, then retry:
> ```powershell
> Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
> ```

#### Using Conda (alternative)

If you prefer Conda over venv:

```bash
conda create -n beamer-dedup python=3.11
conda activate beamer-dedup
```

### Install dependencies

Once your virtual environment is active, install the only required dependency:

```bash
pip install PyMuPDF
```

That's it — `PyMuPDF` (also known as `fitz`) handles all PDF text extraction, image comparison, and PDF rewriting.

If you want to freeze the exact dependency versions for reproducibility:

```bash
pip freeze > requirements.txt   # optional — not required for usage
```

### Verify installation

Run the script with the `--help` flag to confirm everything is wired up correctly:

```bash
python3 dedup_beamer.py --help
```

You should see the usage message listing `input`, `output`, `--dry-run`, and `--verbose`. If you get a `ModuleNotFoundError: No module named 'fitz'`, double-check that you activated your virtual environment and ran `pip install PyMuPDF`.

---

## Quick Start

Once setup is complete, you can deduplicate a Beamer PDF in one line:

```bash
# Preview what would be removed (recommended first!)
python3 dedup_beamer.py lecture.pdf --dry-run

# Actually write the deduplicated PDF
python3 dedup_beamer.py lecture.pdf
```

Output defaults to `<input>_clean.pdf` (e.g., `lecture_clean.pdf`).

---

## How It Works

The script compares each adjacent pair of pages (page *i* vs page *i*+1) after stripping ephemeral footers like page numbers. A four-level detection cascade decides whether page *i* is a transitional half-state that should be removed:

### Detection cascade

| Level | Condition | Interpretation |
|-------|-----------|----------------|
| **1 — Exact Duplicate** | `text_i == text_{i+1}` | Same slide rendered on multiple pages (only page number differs) |
| **2 — Strict Subset** | `text_i ⊂ text_{i+1}` | Classic `\pause` — content accumulates across pages |
| **3 — Word-Set Subset** | `words_i ⊂ words_{i+1}` | Reordered headers, e.g. `"Outline"` → `"Overview Outline"` |
| **4 — Near-Duplicate** | Similarity ≥ 98% + safety checks pass | Text-extraction artifacts / formula rendering differences |

If none of the above holds, page *i* has unique content → **KEEP**.

### Safety overrides

To prevent false positives, a removal is **vetoed** (page kept) if any of these fire:

- Page *i* has text blocks with <50% word overlap against page *i*+1 *(fuzzy matching — tolerates text inserted mid-block)*
- Page *i* has images not present in page *i*+1 *(compared by PDF xref)*
- Page *i* has ≤10 characters of text *(empty / image-only pages)*

The **last page is always kept**.

---

## Usage

### Command-line arguments

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
# Basic dedup — output written to lecture_clean.pdf
python3 dedup_beamer.py lecture.pdf

# Custom output path
python3 dedup_beamer.py lecture.pdf clean_lecture.pdf

# Dry run — see what would be removed before committing
python3 dedup_beamer.py lecture.pdf --dry-run

# Verbose mode — inspect block counts and similarity scores per page
python3 dedup_beamer.py lecture.pdf --verbose

# Combine flags — verbose dry run for a full audit
python3 dedup_beamer.py lecture.pdf --dry-run --verbose

# Process a file in a different directory
python3 dedup_beamer.py ~/Documents/slides.pdf ~/Desktop/slides_clean.pdf
```

### Recommended workflow

1. **Always run `--dry-run` first** to preview what will be removed.
2. Check the summary — especially the "non-HIGH-confidence removals" section at the end.
3. If everything looks correct, run without `--dry-run` to produce the output file.
4. Open the output PDF and spot-check a few slides to confirm nothing important was lost.

---

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

---

## Troubleshooting

### `ModuleNotFoundError: No module named 'fitz'`

You forgot to install PyMuPDF, or your virtual environment is not active.

```bash
# Check if venv is active — you should see (venv) in your prompt
# If not, activate it:
source venv/bin/activate       # macOS / Linux
venv\Scripts\activate           # Windows

# Then install:
pip install PyMuPDF
```

### `python3: command not found` (macOS / Linux)

Your system may use `python` instead of `python3`. Try:

```bash
python --version
```

If that shows 3.7+, use `python` in place of `python3` throughout these instructions.

### `pip: command not found` (Linux)

Some Linux distributions don't include pip by default:

```bash
# Ubuntu / Debian
sudo apt install python3-pip

# Fedora
sudo dnf install python3-pip

# Arch
sudo pacman -S python-pip
```

### PDF has 0 pages processed / "Extracted data from 0 pages"

Your PDF likely contains only scanned images with no extractable text layer. This tool works by comparing text, so image-only PDFs cannot be processed. See [Limitations](#limitations).

### Too many pages were removed / content is missing

Re-run with `--verbose` to see per-page decisions, and with `--dry-run` to avoid writing. Look for `MEDIUM` or `LOW` confidence removals in the summary — these are more likely to be false positives. If you consistently see incorrect removals, please file a GitHub issue with a description of the problematic slide structure.

---

## Limitations

- **Text-based comparison only.** Purely visual transitions (e.g., an arrow appearing on a diagram without text change) will not be detected as duplicates. In practice, LaTeX Beamer's `\pause` and `\item<n->` always affect text, so this covers the vast majority of cases.
- **Scanned/image PDFs** will not be processed (no extractable text to compare).
- **Formula-heavy slides** may produce MEDIUM-confidence results due to PDF rendering artifacts in math expressions. These are still removed but flagged for review.

---

## License

MIT
