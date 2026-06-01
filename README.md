# Beamer PDF Deduplicator

Remove redundant intermediate/transition pages from LaTeX Beamer PDFs.

When you compile a Beamer presentation with animations (`\pause`, `\onslide`, `\item<n->`, overlay specifications), the resulting PDF contains many near-duplicate pages — one for each animation step. This tool strips those out, leaving only the **final complete state** of each slide.

## Table of Contents

- [A note before you start — please read this first](#a-note-before-you-start--please-read-this-first)
- [One-shot setup (do this once)](#one-shot-setup-do-this-once)
- [How to run (do this each time you have a PDF to deduplicate)](#how-to-run-do-this-each-time-you-have-a-pdf-to-deduplicate)
- [How It Works](#how-it-works)
- [Example Output](#example-output)
- [Troubleshooting](#troubleshooting)
- [Limitations](#limitations)
- [License](#license)

---

## A note before you start — please read this first

This tool is a **command-line script**. You don't open a graphical app — you type commands into a terminal (Terminal.app on macOS, Command Prompt or PowerShell on Windows, or your favourite terminal emulator on Linux).

The most important thing to understand is **file paths**. When you run:

```bash
python3 dedup_beamer.py lecture.pdf
```

…the computer needs to know where `dedup_beamer.py` and `lecture.pdf` actually live on your disk. It won't magically find them — **you** must tell it.

Here is a typical scenario after you download this project:

```
your computer
├── ~/Downloads/
│   └── my-presentation.pdf          ← your Beamer PDF is here
│
└── ~/Desktop/
    └── beamer-pdf-dedup/            ← this project is here
        ├── dedup_beamer.py
        └── README.md
```

In this case, the command you'd actually run is:

```bash
cd ~/Desktop/beamer-pdf-dedup
python3 dedup_beamer.py ~/Downloads/my-presentation.pdf
```

- `cd ~/Desktop/beamer-pdf-dedup` — first navigate into the project folder
- `~/Downloads/my-presentation.pdf` — then tell the script exactly where your PDF is

**Every `python3` command below assumes you have already `cd`'d into the project folder.** If you see a `can't open file 'dedup_beamer.py'` error, it means you're in the wrong directory (see [Troubleshooting](#troubleshooting)).

> **Windows users:** replace `~/Desktop` with `C:\Users\YourName\Desktop` and use backslashes `\` instead of forward slashes `/`. Or, even simpler: drag the PDF file from File Explorer into the terminal window — it'll paste the full path for you!

---

## One-shot setup (do this once)

Run these commands **in order**. Copy and paste them one block at a time into your terminal.

### Step 1: Check your Python version

```bash
python3 --version
```

You need **Python 3.7 or higher**. If the output shows something like `Python 3.12.3`, you're good. If it says `command not found`, try `python --version` instead — and if that works, use `python` wherever the rest of this guide says `python3`.

<details>
<summary><b>Don't have Python 3.7+? Click here for install instructions.</b></summary>

- **macOS (Homebrew):** `brew install python3`
- **Ubuntu/Debian Linux:** `sudo apt install python3 python3-pip python3-venv`
- **Fedora Linux:** `sudo dnf install python3 python3-pip`
- **Arch Linux:** `sudo pacman -S python python-pip`
- **Windows:** Download the installer from [python.org](https://www.python.org/downloads/). **Important:** check the box that says "Add Python to PATH" during installation, then restart your terminal.

</details>

### Step 2: Get the code

#### Option A: Using Git (recommended)

Pick a folder where you want to keep the project (e.g., your Desktop), then:

```bash
cd ~/Desktop
git clone https://github.com/reeseliu/beamer-pdf-dedup.git
cd beamer-pdf-dedup
```

#### Option B: Download ZIP

Go to the [GitHub repository](https://github.com/reeseliu/beamer-pdf-dedup), click the green **Code** button → **Download ZIP**. Extract the ZIP file, then open a terminal inside the extracted folder.

On macOS, the easiest way to open a terminal in a folder is: right-click the folder in Finder → **Services** → **New Terminal at Folder**.

### Step 3: Create a virtual environment

This creates an isolated Python environment inside the project folder so dependencies don't clash with your other projects. Run this inside the `beamer-pdf-dedup` folder:

**macOS / Linux:**

```bash
python3 -m venv venv
source venv/bin/activate
```

**Windows (Command Prompt):**

```cmd
python -m venv venv
venv\Scripts\activate
```

**Windows (PowerShell):**

```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
```

After activation, your terminal prompt should now start with `(venv)` — that's how you know it worked.

<details>
<summary><b>PowerShell says "running scripts is disabled"?</b></summary>

Run this first, then retry the activation command above:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

</details>

<details>
<summary><b>Prefer Conda?</b></summary>

```bash
conda create -n beamer-dedup python=3.11
conda activate beamer-dedup
```

</details>

### Step 4: Install PyMuPDF

```bash
pip install PyMuPDF
```

This is the only dependency. It handles all PDF reading, text extraction, and PDF writing.

### Step 5: Verify everything works

```bash
python3 dedup_beamer.py --help
```

You should see a usage message listing `input`, `output`, `--dry-run`, and `--verbose`. If you instead see `ModuleNotFoundError: No module named 'fitz'`, go back to Step 4 and make sure `(venv)` is visible in your prompt before running `pip install PyMuPDF`.

**Setup is complete!** You only need to do these steps once. Next time you want to use the tool, skip straight to the section below.

---

## How to run (do this each time you have a PDF to deduplicate)

### Step 1: Open a terminal and navigate to the project

```bash
cd ~/Desktop/beamer-pdf-dedup
```

> Change `~/Desktop/beamer-pdf-dedup` to wherever you actually put the project. If you cloned it to `~/Documents/beamer-pdf-dedup`, use that path instead.

### Step 2: Activate the virtual environment

```bash
source venv/bin/activate        # macOS / Linux
venv\Scripts\activate            # Windows
```

You should see `(venv)` appear in your prompt.

### Step 3: Run the tool on your PDF

**CRITICAL — read this before copy-pasting:** The word `lecture.pdf` in the examples below is a **placeholder**. You must replace it with the actual path to your Beamer PDF file. For example:

- If your PDF is on your Desktop: `~/Desktop/my-slides.pdf`
- If your PDF is in Downloads: `~/Downloads/presentation.pdf`
- If you're not sure of the path, **drag the PDF file directly into the terminal** — your OS will paste the full path for you.

Now choose one of the following:

```bash
# === SAFEST: Preview first (no file is written) ===
python3 dedup_beamer.py ~/path/to/your-file.pdf --dry-run

# === Once you're happy with the preview, produce the output ===
python3 dedup_beamer.py ~/path/to/your-file.pdf
```

The output file will be saved next to your original PDF, with `_clean` added to the name. For example, `slides.pdf` → `slides_clean.pdf`.

### Concrete example

Let's say your Beamer PDF is at `~/Documents/talks/my-lecture.pdf`. Here's exactly what you'd type:

```bash
cd ~/Desktop/beamer-pdf-dedup
source venv/bin/activate
python3 dedup_beamer.py ~/Documents/talks/my-lecture.pdf --dry-run
```

If the preview looks good, re-run without `--dry-run`:

```bash
python3 dedup_beamer.py ~/Documents/talks/my-lecture.pdf
```

The cleaned file will be at `~/Documents/talks/my-lecture_clean.pdf`.

### Other useful commands

```bash
# Save the output to a specific location
python3 dedup_beamer.py ~/Documents/slides.pdf ~/Desktop/slides_clean.pdf

# See detailed per-page decisions (useful if something looks wrong)
python3 dedup_beamer.py ~/Documents/slides.pdf --verbose

# Combine flags — verbose + dry-run for a full audit
python3 dedup_beamer.py ~/Documents/slides.pdf --dry-run --verbose
```

### Command reference

```
usage: dedup_beamer.py [-h] [--dry-run] [--verbose] input [output]

positional arguments:
  input            Path to your Beamer PDF file (required)
  output           Where to save the cleaned PDF (default: <input>_clean.pdf)

optional arguments:
  --dry-run        Show what would be removed, but don't write a file
  --verbose, -v    Show per-page details: why each page was kept or removed
```

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

### `can't open file 'dedup_beamer.py': [Errno 2] No such file or directory`

Your terminal is in the wrong folder. You need to `cd` into the project directory first:

```bash
cd ~/Desktop/beamer-pdf-dedup     # or wherever you put the project
```

To check where you currently are, type `pwd` (macOS/Linux) or `cd` (Windows). To see what files are in your current directory, type `ls` (macOS/Linux) or `dir` (Windows) — you should see `dedup_beamer.py` listed.

### `No such file or directory: 'lecture.pdf'`

You used `lecture.pdf` as the input, but there's no file called `lecture.pdf` in your current directory. Remember: **`lecture.pdf` is a placeholder** — replace it with the actual path to your PDF:

```bash
# ❌ Wrong — lecture.pdf doesn't exist
python3 dedup_beamer.py lecture.pdf

# ✅ Correct — use the real path to your PDF
python3 dedup_beamer.py ~/Downloads/my-actual-slides.pdf
```

Not sure of the exact path? **Drag the PDF file from your file manager into the terminal** — the full path will appear as if by magic.

### `ModuleNotFoundError: No module named 'fitz'`

You're either missing PyMuPDF or your virtual environment isn't active. Fix it by running:

```bash
# First, make sure (venv) appears in your terminal prompt
source venv/bin/activate       # macOS / Linux
venv\Scripts\activate           # Windows

# Then install PyMuPDF
pip install PyMuPDF
```

### `python3: command not found`

Your system might use `python` (without the `3`) instead. Check with:

```bash
python --version
```

If that shows Python 3.7+, use `python` instead of `python3` in all the commands above.

### `pip: command not found`

Install pip via your system's package manager:

- **Ubuntu/Debian:** `sudo apt install python3-pip`
- **Fedora:** `sudo dnf install python3-pip`
- **Arch:** `sudo pacman -S python-pip`
- **macOS (Homebrew):** pip comes with `brew install python3`
- **Windows:** pip is included with the Python installer from python.org

### PDF has 0 pages processed / "Extracted data from 0 pages"

Your PDF likely contains only scanned images with no extractable text layer. This tool compares text, so image-only PDFs cannot be processed. See [Limitations](#limitations).

### Too many pages were removed / content is missing

Re-run with `--verbose` and `--dry-run` to see per-page decisions:

```bash
python3 dedup_beamer.py ~/path/to/your-file.pdf --dry-run --verbose
```

Look for `MEDIUM` or `LOW` confidence removals — these are the most likely false positives. If you consistently see incorrect removals, please file a GitHub issue describing the problematic slide structure.

---

## Limitations

- **Text-based comparison only.** Purely visual transitions (e.g., an arrow appearing on a diagram without text change) will not be detected as duplicates. In practice, LaTeX Beamer's `\pause` and `\item<n->` always affect text, so this covers the vast majority of cases.
- **Scanned/image PDFs** will not be processed (no extractable text to compare).
- **Formula-heavy slides** may produce MEDIUM-confidence results due to PDF rendering artifacts in math expressions. These are still removed but flagged for review.

---

## License

MIT
