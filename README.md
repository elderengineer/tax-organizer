# Tax Organizer — Setup Guide

Organizes your personal and consultant tax documents (W-2s, 1099s, rental expenses, donations, business trips) and generates ready-to-review CSV reports for your accountant. Powered by Claude Code.

---

## What This Skill Does

- Reads PDF receipts, invoices, W-2s, 1099s, and spreadsheet exports
- Scans your documents and guides you through business line setup — no manual folder creation needed
- Renames and sorts files into the right folders automatically
- Calculates rental property depreciation
- Compares business trip actual expenses vs. GSA per diem rates
- Generates Schedule E, W-2, 1099, donations, and business trip CSV reports

---

## Step 1 — Install Claude Code

Claude Code is a command-line tool from Anthropic. You need Node.js 18+ installed first.

**Install Node.js** (if you don't have it): https://nodejs.org — download the LTS version.

**Install Claude Code:**

```bash
npm install -g @anthropic/claude-code
```

**Log in:**

```bash
claude
```

Follow the prompts to sign in with your Anthropic account. If you don't have one, create a free account at https://claude.ai.

---

## Step 2 — Install the Required Skills

Skills are Claude Code commands stored as Markdown files. This project needs three skills: `pdf`, `xlsx`, and the tax organizer itself.

Skills live in `~/.claude/commands/` (your home directory, works across all projects).

**Create the commands folder:**

```bash
mkdir -p ~/.claude/commands
```

### Install the PDF skill

The PDF skill is included in this repository under `skills/pdf/`.

```bash
cp skills/pdf/SKILL.md ~/.claude/commands/pdf.md
```

### Install the XLSX skill

The XLSX skill is included in this repository under `skills/xlsx/`.

```bash
cp skills/xlsx/SKILL.md ~/.claude/commands/xlsx.md
```

### Install the Tax Organizer skill

```bash
cp SKILL.md ~/.claude/commands/tax-organizer.md
```

**Verify all three skills are installed:**

```bash
ls ~/.claude/commands/
# Should show: pdf.md  xlsx.md  tax-organizer.md
```

---

## Step 3 — Create Your Tax Year Folder

Create a dedicated folder for your tax documents. One folder per tax year.

```bash
mkdir ~/Documents/Tax-2025
cd ~/Documents/Tax-2025
```

Drop all your documents into this folder — receipts, invoices, W-2s, 1099s, spreadsheet exports, whatever you have. You can also add more documents later; the skill only processes new files on each run.

---

## Step 4 — Run the Tax Organizer

Open Claude Code in your tax folder:

```bash
cd ~/Documents/Tax-2025
claude
```

Then type the slash command:

```
/tax-organizer
```

**First run:** The skill will:
1. Ask for your tax year, filing status, and preparer name
2. Scan all the documents you dropped in
3. Identify likely business lines from your documents (rental properties, freelance income, partnerships, etc.)
4. Walk you through confirming each one and filling in any missing details (cost basis, in-service date, etc.)
5. Create the folder structure and generate all business `README.md` config files for you
6. Organize, rename, and generate reports

**Subsequent runs:** Drop new documents into the folder and run `/tax-organizer` again. It detects only new files and skips everything already processed. If new documents suggest a new business line, it will prompt you to set it up.

---

## Folder Structure (Created Automatically)

```
Tax-2025/
├── README.md                  # Your tax year config
├── Rental-123-Main-St/        # One folder per business/rental (created by the skill)
│   ├── README.md              # Business config (address, cost basis, trips)
│   └── 2024-03-15 Plumber Co - Repairs - Pipe fix.pdf
├── W2s/
├── 1099s/
├── Donations/
├── Unclassified/              # Files that need manual review
├── Reports/                   # Generated CSVs — ready for your accountant
│   ├── schedule-e-draft.csv
│   ├── business-trips.csv
│   ├── donations-summary.csv
│   ├── w2-summary.csv
│   └── 1099-summary.csv
└── .processed-files.log       # Internal tracking (don't edit)
```

---

## Dropping Documents

You can drop documents anywhere in the tax folder — the skill classifies them automatically:

| Document | Where to drop |
|---|---|
| W-2 forms | Anywhere — moved to `W2s/` |
| 1099s (NEC, MISC, INT, DIV, K, B, R, S) | Anywhere — moved to `1099s/` |
| Rental receipts / invoices | Anywhere — matched to the right property |
| Donation receipts | Anywhere — moved to `Donations/` |
| Business trip receipts | Anywhere — matched by date to trips in business README |
| Excel / CSV bank exports | Anywhere — extracted via `/xlsx` |
| Anything else | Moved to `Unclassified/` if unclear |

---

## Tips

- **Just drop and run** — you don't need to pre-sort or rename anything. The skill figures it out.
- **Never delete `.processed-files.log`** — it's how the skill tracks what's been done.
- **Originals are preserved** — the skill copies files to organized locations; your originals are untouched.
- **Add documents anytime** — run `/tax-organizer` after each batch; it skips already-processed files.
- **Check `Unclassified/`** after each run for anything that needs manual assignment.
- **Reports are always regenerated** from the full history, so they're always up to date.
