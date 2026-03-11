# Git Pain Points & Solutions for Researchers

A quick-reference guide for common issues that come up when using Git in research workflows.

---

## 1. Authenticating with GitHub from the Command Line

GitHub no longer accepts passwords for command-line Git operations. You have three options:

### Option A: GitHub CLI (recommended for beginners)

```bash
# Install
brew install gh          # macOS
winget install GitHub.cli # Windows

# Authenticate (interactive — walks you through it)
gh auth login
```

This handles everything: HTTPS tokens, SSH keys, and even sets Git's credential helper for you. After running `gh auth login`, normal `git push` / `git pull` just work.

### Option B: Personal Access Token (PAT)

1. Go to **GitHub → Settings → Developer Settings → Personal Access Tokens → Tokens (classic)**
2. Generate a token with `repo` scope
3. When Git asks for your password, paste the token instead
4. To avoid re-entering it every time:

```bash
# Cache credentials for 8 hours (28800 seconds)
git config --global credential.helper 'cache --timeout=28800'

# Or store permanently (saved in plain text — less secure)
git config --global credential.helper store
```

**macOS** uses Keychain automatically. **Windows** can use:
```bash
git config --global credential.helper manager
```

### Option C: SSH Keys

```bash
# Generate a key pair
ssh-keygen -t ed25519 -C "you@example.com"

# Copy public key to clipboard
cat ~/.ssh/id_ed25519.pub   # then paste into GitHub → Settings → SSH Keys

# Test
ssh -T git@github.com
```

Then clone/add remotes using the SSH URL (`git@github.com:user/repo.git`) instead of HTTPS.

### Which should I recommend to participants?

| Audience | Recommendation |
|---|---|
| Brand new to Git | **GitHub Desktop** (handles auth automatically) |
| Comfortable with terminal | **GitHub CLI** (`gh auth login`) |
| Power users / HPC | **SSH keys** |

---

## 2. Managing Large / Data Files

Git stores every version of every file. A 500 MB CSV that changes 10 times = ~5 GB repo. This breaks things fast.

### Rule of thumb

| File size | Strategy |
|---|---|
| < 1 MB | Track normally with Git |
| 1–100 MB | Consider `.gitignore` + separate storage, or Git LFS |
| > 100 MB | Do **not** put in Git. Use Git LFS, git-annex, or external storage |

### Option A: Just don't track it

The simplest approach. Add data paths to `.gitignore` and document where to get the data:

```gitignore
data/raw/
data/external/
*.h5
*.nwb
*.tif
```

Include a `README.md` or `data/README.md` explaining:
- Where the data lives (e.g., server path, Dropbox, Zenodo DOI)
- How to download or regenerate it
- A small sample dataset *can* be committed for testing

### Option B: Git LFS (Large File Storage)

Replaces large files with lightweight pointers in Git, stores the actual content on a separate server.

```bash
# Install
brew install git-lfs      # macOS
git lfs install            # one-time setup

# Track file types
git lfs track "*.csv"
git lfs track "*.h5"

# This creates/updates .gitattributes — commit it!
git add .gitattributes
git commit -m "Track large files with LFS"

# Then use git normally
git add data/experiment.csv
git commit -m "Add raw experiment data"
git push origin main
```

**Limitations:**
- GitHub free: 1 GB storage, 1 GB/month bandwidth
- Every collaborator needs `git lfs install`
- Not ideal for thousands of large files

### Option C: git-annex / DataLad (for serious data management)

Best for neuroscience / large-scale datasets. DataLad wraps git-annex with a more user-friendly interface.

```bash
# Install
pip install datalad

# Create a dataset
datalad create my-project
cd my-project

# Add large files (annexed automatically)
datalad save -m "Add raw recordings" data/
```

**Hosting options for annexed data:**
- **GIN (G-Node Infrastructure)** — free, designed for scientific data, supports git-annex natively
- **DataLad Hub** — powered by Forgejo-aneksajo
- Any SSH server, S3 bucket, etc.

### Comparison

| | Git LFS | git-annex / DataLad |
|---|---|---|
| Setup complexity | Low | Medium–High |
| Best for | A few large files | Many large files, complex datasets |
| Hosting | GitHub, GitLab | GIN, any SSH server, S3 |
| Deduplication | No | Yes |
| Partial clone (get only some files) | Limited | Yes |
| Research community adoption | General | Neuroscience, psychology |

---

## 3. Notebooks (Jupyter, R Markdown)

Notebooks are JSON files (Jupyter) or mixed text (Rmd) that embed code, output, and sometimes base64-encoded images. This makes diffs noisy and merges painful.

### The core problem

```json
// A single-pixel change in a plot output looks like this in git diff:
- "image/png": "iVBORw0KGgoAAAANSUhEUgAAA...  (500 lines of base64)"
+ "image/png": "iVBORw0KGgoAAAANSUhEUgAAB...  (500 lines of base64)"
```

### Strategies

#### Strategy 1: Clear outputs before committing (simplest)

**Jupyter:**
```bash
# One-time: install nbstripout
pip install nbstripout

# Enable for this repo (auto-strips on commit)
nbstripout --install

# Or manually clear before committing
jupyter nbconvert --ClearOutputPreprocessor.enabled=True --inplace notebook.ipynb
```

**R Markdown:** Outputs are typically separate files (HTML, PDF), so just `.gitignore` them:
```gitignore
*.html
*.pdf
*_files/
```

#### Strategy 2: Pair notebooks with plain scripts

Keep a `.py` or `.R` script as the "source of truth" and treat the notebook as a rendered view:

```bash
# Using jupytext (auto-syncs .ipynb ↔ .py)
pip install jupytext
jupytext --set-formats ipynb,py:percent notebook.ipynb
```

Then commit the `.py` file and `.gitignore` the `.ipynb`, or commit both.

#### Strategy 3: Review tools for notebooks

- **nbdime** — diff and merge tool designed for Jupyter notebooks:
  ```bash
  pip install nbdime
  nbdime config-git --enable     # integrates with git diff/merge

  nbdiff notebook_v1.ipynb notebook_v2.ipynb        # terminal diff
  nbdiff-web notebook_v1.ipynb notebook_v2.ipynb     # visual diff in browser
  ```
- **GitHub** renders `.ipynb` files natively (but diffs are still noisy)
- **VS Code** has built-in notebook diffing

### Recommended workflow for the session

| Approach | When to use |
|---|---|
| `nbstripout` | Default — always install this in research repos |
| `jupytext` | When you want clean diffs *and* notebook convenience |
| `nbdime` | When collaborating on notebooks and need to review/merge |

---

## 4. Binary Files (.docx, .pdf, images)

Git can store binary files, but it can't show meaningful diffs — every change looks like "binary files differ."

### What to do

| File type | Recommendation |
|---|---|
| `.docx`, `.pptx` | Avoid in Git. Use Google Docs, Overleaf, or convert to Markdown/LaTeX |
| `.pdf` (generated) | `.gitignore` — regenerate from source |
| `.pdf` (archival, e.g., signed forms) | Track with Git or LFS, accept you won't get diffs |
| Images (`.png`, `.jpg`) | Track if small and infrequent. Use LFS if they change often |
| `.fig`, `.ai`, `.svg` | `.svg` is text-based (Git-friendly). Others are binary — use LFS |

### Enabling partial diffs for some binary formats

You can teach Git to extract text from binary files for diffing:

```bash
# .docx diffs (requires pandoc)
# Add to .gitattributes:
*.docx diff=word

# Add to .git/config or ~/.gitconfig:
[diff "word"]
    textconv = pandoc --to=plain
```

```bash
# PDF diffs (requires pdftotext from poppler)
*.pdf diff=pdf

[diff "pdf"]
    textconv = pdftotext -layout
```

These won't be perfect, but they let `git log -p` show *something* meaningful.

### Practical advice for the session

> **"If it's generated, don't track it. If it's source, track it. If it's binary source, consider whether a text-based alternative exists."**

```gitignore
# Generated outputs — don't track
*.pdf
*.docx
figures/*.png

# But DO track source
# manuscript.tex   ← yes
# analysis.R       ← yes
# figures/plot.R   ← yes (generates the .png)
```

---

## 5. Merge Conflicts

Merge conflicts happen when two branches (or two people) edit the same lines of the same file. Git doesn't know which version to keep, so it asks you.

### What a conflict looks like

```
The sample size was
<<<<<<< HEAD
n = 45 after filtering outliers (>3 SD).
=======
n = 50 including all participants.
>>>>>>> new-analysis
```

- **`<<<<<<< HEAD`** — what's in your current branch
- **`=======`** — separator
- **`>>>>>>> new-analysis`** — what's in the incoming branch

### How to resolve

#### Step 1: See which files are conflicted

```bash
git status
# Shows: "both modified: analysis.R"
```

#### Step 2: Open the file and pick what to keep

Edit the file to remove the conflict markers and keep the correct content:

```
The sample size was n = 45 after filtering outliers (>3 SD).
```

#### Step 3: Mark as resolved and commit

```bash
git add analysis.R
git commit -m "Resolve conflict: keep filtered sample size"
```

### Visual merge tools (easier than editing raw markers)

```bash
# Use VS Code as your merge tool
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait --merge $REMOTE $LOCAL $BASE $MERGED'

# When a conflict happens:
git mergetool
```

**GitHub Desktop** and **VS Code** both highlight conflicts visually and let you click to accept "mine," "theirs," or edit manually.

### How to minimize conflicts

| Practice | Why it helps |
|---|---|
| Pull before you start working | You're working on the latest version |
| Commit small and often | Smaller changes = smaller conflicts |
| Use branches for experiments | Keeps `main` stable |
| Don't edit the same file simultaneously | Communicate with collaborators |
| Avoid committing generated files | These change constantly and cause spurious conflicts |

### Live demo suggestion

Create a conflict on purpose to demystify it:

```bash
# Setup
git init conflict-demo && cd conflict-demo
echo "Result: 42" > results.txt
git add . && git commit -m "Initial result"

# Branch A
git checkout -b branch-a
echo "Result: 45 (filtered)" > results.txt
git commit -am "Update result with filtering"

# Branch B
git checkout main
git checkout -b branch-b
echo "Result: 50 (all subjects)" > results.txt
git commit -am "Update result with all subjects"

# Merge → conflict!
git checkout main
git merge branch-a        # succeeds
git merge branch-b        # CONFLICT — now resolve it together
```

---

## Quick Reference: What Goes Where

```
├── .git/                  ← Git internals (never touch)
├── .gitignore             ← Files Git should ignore
├── .gitattributes         ← LFS tracking, diff drivers
├── README.md              ← Always have one
├── code/
│   ├── analysis.R         ← ✅ Track
│   └── utils.py           ← ✅ Track
├── notebooks/
│   └── exploration.ipynb  ← ⚠️ Track with nbstripout
├── data/
│   ├── README.md          ← ✅ Track (describes data sources)
│   ├── raw/               ← 🚫 .gitignore (large, immutable)
│   └── processed/         ← 🚫 .gitignore (regenerable)
├── figures/
│   ├── plot_figure2.R     ← ✅ Track (source)
│   └── figure2.png        ← 🚫 .gitignore (generated)
└── manuscript/
    ├── manuscript.tex     ← ✅ Track
    └── manuscript.pdf     ← 🚫 .gitignore (generated)
```

---

*BOOST — Version Control for Reproducible Research — March 11, 2026*