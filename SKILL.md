# Tax Organizer for Personal & Consultant

Organizes and processes personal tax documents to generate accurate, tax reports and drafts. Use when
user uploads or describes tax forms (W-2, 1099s, K-1s, 1098), mentions rental properties, royalties,
partnerships/S-corps, business expenses, donations, or needs Schedule E drafts. Handles complex income sources, expense
matching, per diem optimization, and charitable receipt identification.

## When to Use

- Organizing personal and consultant tax documents
- Processing rental property expenses, income, and depreciation
- Matching business trip expenses with per diem optimization
- Generating tax-ready XLS reports for accountant review
- Classifying W-2s, 1099s, and donation receipts
- Tracking incremental document additions across multiple sessions

## Usage

```
Navigate to your tax year folder and run this skill.
Drop documents anytime — the skill detects new files and only processes what's changed.
```

## Folder Structure

The skill expects and creates this layout in the working directory:

```
Tax-2025/
├── README.md                     # Global config + instructions
├── Rental-123-Main-St/
│   ├── README.md                 # Business config (see template below)
│   └── (drop receipts/invoices here)
├── Freelance-Consulting/
│   ├── README.md
│   └── ...
├── W2s/                          # W-2 forms
├── 1099s/                        # All 1099 variants
├── Donations/                    # Charitable contribution receipts
├── Unclassified/                 # Files the skill couldn't classify
├── Reports/                      # Generated XLS outputs
└── .processed-files.log          # Internal: tracks processed files
```

### Special folders

- **`W2s/`** — W-2 forms. Organized for reference (Form 1040, not Schedule E).
- **`1099s/`** — All 1099 variants (NEC, MISC, INT, DIV, K-1, etc.).
- **`Donations/`** — Charitable contribution receipts (Form 8283 / Schedule A).
- **`Unclassified/`** — Documents that couldn't be auto-classified. Flagged for manual review.
- **`Reports/`** — Generated XLS report files. Regenerated on every run.

## Root README.md Template

When no root `README.md` exists, generate one from this template (fill in from user responses):

```markdown
# Tax Documents — {{TAX_YEAR}}

## Instructions

This folder organizes your tax documents for the {{TAX_YEAR}} tax year.

### How to use

1. **Drop receipts and documents** into the tax year folder
   — or into any existing business subfolder
2. **Run the skill** — it scans your documents, identifies business
   lines, and guides you through setup
3. **Add more documents anytime** — the skill detects new files and
   only processes what's changed

### Special folders

- `W2s/` — Drop W-2 forms here (organized for reference)
- `1099s/` — Drop all 1099 forms here
- `Donations/` — Drop charitable contribution receipts here

## Filing Information

- **Tax Year**: {{TAX_YEAR}}
- **Filing Status**: {{FILING_STATUS}}
- **Preparer**: {{PREPARER_NAME}}
```

## Business README.md Template

Each business folder gets a `README.md` that serves as both config and documentation. Generate from this template:

```markdown
# {{BUSINESS_NAME}}

## Business Information

- **Type**: {{BUSINESS_TYPE}}
- **EIN**: {{EIN}}
- **Address**: {{ADDRESS}}
- **Ownership**: {{OWNERSHIP_PERCENTAGE}}%

## Property Details

<!-- Only for rental properties -->

- **In-Service Date**: {{IN_SERVICE_DATE}}
- **Cost Basis**: ${{COST_BASIS}}
- **Land Value**: ${{LAND_VALUE}}
- **Depreciation Method**: {{DEPRECIATION_METHOD}}

## Business Trips

<!-- List trips related to this business. The skill will auto-match
     expenses (flights, hotels, meals, taxi, conference fees) that
     fall within these date ranges. -->

### {{TRIP_NAME}} — {{TRIP_DESTINATION}}

- **Dates**: {{TRIP_START}} – {{TRIP_END}}
- **Purpose**: {{TRIP_PURPOSE}}

## Important Events

<!-- Note major events that affect tax treatment -->

- **{{EVENT_DATE}}**: {{EVENT_DESCRIPTION}}

## Notes

<!-- Any additional notes for your records or your accountant -->
```

**Business types**: Residential Rental, Commercial Rental, Royalties, Partnership (K-1), S-Corp (K-1),
Freelance/Self-Employment, Farm Rental

**Depreciation methods**:

- Residential rental: Straight-line over 27.5 years
- Commercial rental: Straight-line over 39 years
- Include mid-month convention for the placed-in-service year

## Incremental Processing

The skill maintains a `.processed-files.log` in the root folder to track what has been processed:

```
# Tax Organizer — Processed Files Log
# Format: SHA256_HASH | ORIGINAL_PATH | ORGANIZED_PATH | TIMESTAMP
a1b2c3d4... | ./invoice_march.pdf | ./Rental-123-Main-St/2024-03-15 Plumber Co - Repairs - Pipe fix.pdf | 2024-12-01T10:30:00
e5f6g7h8... | ./receipt.jpg | ./Rental-123-Main-St/2024-04-20 Home Depot - Supplies - Door hardware.jpg | 2024-12-01T10:30:00
```

### Processing rules

On each run, hash all files in the directory tree (excluding `Reports/` and the log itself), then compare
against `.processed-files.log`:

| Condition                                      | Action                               |
|------------------------------------------------|--------------------------------------|
| **New file** (hash not in log)                 | Process normally                     |
| **Duplicate file** (same hash, different path) | Flag as duplicate, skip, notify user |
| **Already processed** (hash + path match)      | Skip silently                        |
| **Moved/renamed** (hash matches, path changed) | Update log entry, skip reprocessing  |

Report to user: "Found X new files, Y duplicates, Z already processed"

## Workflow

Execute these 9 steps in order:

---

### Step 1: Scan & Discover

1. Scan the working directory recursively for all documents: PDF, JPG, PNG, JPEG, TIFF, XLS, XLSX, CSV, EML, MSG. For
   XLS/XLSX files, use the `/xlsx` skill to read their contents.
2. Exclude `Reports/` folder and `.processed-files.log`
3. Compute SHA-256 hash for each file found
4. Load `.processed-files.log` if it exists
5. Categorize every file as: new, already-processed, duplicate, or moved/renamed (see Processing Rules above, use **mv** instead of cp)
6. Detect existing `README.md` configs in business folders
7. Report to user:
    - Total files found
    - New files to process
    - Duplicates found
    - Already processed (skipped)
    - File types breakdown

If there are zero new files to process, skip to Step 8 (regenerate reports) or inform the user and stop.

---

### Step 2: Interactive Business Setup

**First run** (no root `README.md` found):

1. Ask user for:
    - Tax year
    - Filing status (Single, Married Filing Jointly, Married Filing Separately, Head of Household, Qualifying Surviving
      Spouse)
    - Preparer name (their name or accountant's name)
2. Generate the root `README.md` using the template above
3. Create empty special folders: `W2s/`, `1099s/`, `Donations/`, `Unclassified/`, `Reports/`
4. Analyze the new documents found in Step 1 and identify likely business lines:
    - Property addresses on invoices/receipts → suggest a rental or commercial property folder
    - Payer names on 1099-NEC/MISC → suggest a freelance or royalty folder
    - K-1 forms → suggest a Partnership or S-Corp folder
    - Group documents that share an address or entity name into a single proposed business
5. Present the suggested business lines to the user, for example:
   ```
   I found documents referencing these potential business lines:

   1. "456 Oak Ave" — appears on 3 receipts and a 1098. Looks like a rental property.
   2. "Acme Consulting LLC" — appears on a 1099-NEC. Looks like freelance income.

   For each one I'll ask a few questions to set it up.
   ```
6. For each confirmed business line, ask:
    - Business name (suggest one based on address or entity name)
    - Business type (Residential Rental, Commercial Rental, Royalties, Partnership, S-Corp, Freelance/Self-Employment,
      Farm Rental)
    - EIN (if applicable)
    - Address (pre-fill from document if found)
    - Ownership percentage
    - For rentals: in-service date, cost basis, land value, depreciation method
7. For each business, ask if there were any related business trips:
    - Trip name, destination, date range, purpose
8. Generate a `README.md` in each business folder using the template above
9. If any documents don't suggest a clear business line, ask the user whether to create a new business folder or
   place them in `Unclassified/`

**Subsequent runs** (root `README.md` exists):

1. Parse existing `README.md` configs from root and all business folders
2. If new documents suggest a business line not covered by any existing folder, prompt the user to set it up
   (same flow as steps 4–9 above, for the new business only)
3. Do NOT re-prompt for existing businesses unless user explicitly requests changes
4. Proceed to Step 3

---

### Step 3: Document Classification

For each **new/unprocessed** document, read and classify it:

| Document Type        | Destination              | Identification Clues                                                                    |
|----------------------|--------------------------|-----------------------------------------------------------------------------------------|
| **W-2**              | `W2s/`                   | "Wage and Tax Statement", Form W-2, boxes for wages/withholding                         |
| **1099-NEC**         | `1099s/`                 | "Nonemployee Compensation", Box 1                                                       |
| **1099-MISC**        | `1099s/`                 | "Miscellaneous Income", rents/royalties boxes                                           |
| **1099-INT**         | `1099s/`                 | "Interest Income"                                                                       |
| **1099-DIV**         | `1099s/`                 | "Dividends and Distributions"                                                           |
| **1099-K**           | `1099s/`                 | "Payment Card and Third Party Network"                                                  |
| **1099-B**           | `1099s/`                 | "Proceeds From Broker", stock/investment sales                                          |
| **1099-R**           | `1099s/`                 | "Distributions From Pensions"                                                           |
| **1099-S**           | `1099s/`                 | "Proceeds From Real Estate Transactions"                                                |
| **1099-G**           | `1099s/`                 | "Government Payments"                                                                   |
| **Schedule K-1**     | `1099s/`                 | Partner's/Shareholder's share of income                                                 |
| **Donation receipt** | `Donations/`             | 501(c)(3) reference, "tax-deductible", "charitable contribution", church/nonprofit name |
| **Business expense** | Matching business folder | Vendor name, property address, expense context matches a business                       |
| **Unclassifiable**   | `Unclassified/`          | Cannot determine type or business; flag for manual review                               |

**Classification logic for business expenses:**

1. Check if the document mentions a property address matching a business folder
2. Check if the vendor name appears in previous documents for a business
3. Check if the expense date falls within a business trip date range
4. If multiple businesses could match, ask the user
5. If no match found, place in `Unclassified/`

---

### Step 4: Extract Information

For each document, extract all relevant data fields:

**All documents:**

- Date (transaction date, statement date, or tax year)
- Vendor / payer name
- Amount(s)
- Description / memo

**W-2 fields:**

- Employer name and EIN
- Employee name and SSN (last 4 only — do not store full SSN)
- Box 1: Wages, tips, other compensation
- Box 2: Federal income tax withheld
- Box 3: Social security wages
- Box 4: Social security tax withheld
- Box 5: Medicare wages
- Box 6: Medicare tax withheld
- Box 12 codes (DD, W, etc.)
- Box 13: Statutory employee, retirement plan, third-party sick pay
- State wages and withholding

**1099 fields (varies by type):**

- Payer name, TIN
- Recipient name
- All box amounts relevant to the 1099 type
- State tax withheld if applicable

**Expense fields:**

- Vendor name
- Transaction date
- Amount (pre-tax, tax, total)
- Expense category (map to Schedule E line — see line reference table)
- Payment method if visible
- Property/business reference if visible

**Donation fields:**

- Organization name
- Date of contribution
- Amount (or fair market value for non-cash)
- Cash vs. non-cash
- 501(c)(3) status / tax-exempt confirmation
- Description of donated items (non-cash)
- Whether goods/services were received in return

**Spreadsheet files (XLS/XLSX):**

- Invoke the `/xlsx` skill to read and extract data from Excel files
- These may contain multiple transactions (bank statements, property management reports, accounting exports)
- Extract each row as a separate transaction with the fields above
- CSV files can be read directly without the `/xlsx` skill

**When extraction is incomplete:**

- Use filename clues (dates in filename, vendor name patterns)
- Use file metadata (creation date, modification date)
- Flag incomplete extractions — never silently guess missing values
- Mark confidence level: HIGH (clear document), MEDIUM (partial info), LOW (mostly inferred)

---

### Step 5: Business Trip Expense Matching

1. Parse trip date ranges from each business folder's `README.md`:
    - Extract trip name, destination city + state, start date, end date, purpose
2. For each expense document dated within a trip's date range, check if it matches trip-related categories:
    - Airfare / flights
    - Hotels / lodging
    - Meals (50% deductible per IRS rules)
    - Taxi / rideshare / car rental
    - Conference / registration fees
    - Parking and tolls
    - Baggage fees
3. Also check expenses ±1 day outside trip dates — flag as "potentially trip-related" for user review
4. Look up GSA per diem rates for the trip destination:
    - Use the GSA per diem API or published rates for the tax year
    - Lodging rate (per night)
    - M&IE rate (meals and incidental expenses per day)
    - First and last day of travel: 75% of M&IE rate
5. Calculate per diem totals:
    - Lodging: nightly rate × number of nights
    - M&IE: full-day rate × (trip days − 2) + 75% rate × 2 (first/last day)
    - If trip is 1 day: 75% of M&IE, no lodging
6. Compare actual tracked expenses vs. per diem allowance
7. Recommend whichever method produces the **larger deduction**, noting:
    - Per diem: simpler, no receipt tracking needed for meals
    - Actual: may be higher for expensive destinations
    - IRS requires consistency within a trip (can't mix methods)
8. Tag matched expenses with trip name for reporting

---

### Step 6: Rename & Organize Files

1. Generate a rename/organize plan for all new files. Naming convention:
   ```
   YYYY-MM-DD Vendor - Category - Description.ext
   ```
   Examples:
    - `2024-03-15 Plumber Co - Repairs - Kitchen pipe fix.pdf`
    - `2024-04-20 Home Depot - Supplies - Door hardware.jpg`
    - `2024-01-31 Acme Corp - W2 - Wages.pdf`
    - `2024-12-15 Red Cross - Donation - Annual gift.pdf`
    - `2024-09-06 Marriott - Travel - BiggerPockets Conference hotel.pdf`

2. **Show the complete before/after plan to the user:**
   ```
   File Organization Plan:

   NEW FILES (5):
   ├── invoice_march.pdf
   │   → Rental-123-Main-St/2024-03-15 Plumber Co - Repairs - Pipe fix.pdf
   ├── receipt.jpg
   │   → Rental-123-Main-St/2024-04-20 Home Depot - Supplies - Door hardware.jpg
   ├── W2_acme.pdf
   │   → W2s/2024-01-31 Acme Corp - W2 - Wages.pdf
   ...

   DUPLICATES (1):
   ├── invoice_march_copy.pdf (same as invoice_march.pdf) — SKIPPED

   UNCLASSIFIED (1):
   ├── mystery_doc.pdf → Unclassified/mystery_doc.pdf — NEEDS REVIEW
   ```

3. **Wait for user approval before executing any file operations**
4. **Copy** files to their organized locations (preserve originals)
5. Update `.processed-files.log` with entries for each processed file:
   ```
   SHA256_HASH | ORIGINAL_PATH | ORGANIZED_PATH | ISO_TIMESTAMP
   ```

---

### Step 7: Calculate Depreciation

For each rental property with property details in its `README.md`:

1. **Depreciable basis** = Cost Basis − Land Value
2. **Annual depreciation**:
    - Residential (27.5 years): Depreciable Basis ÷ 27.5
    - Commercial (39 years): Depreciable Basis ÷ 39
3. **Mid-month convention** (placed-in-service year):
    - Month placed in service: count as half-month
    - Formula: (Annual Depreciation ÷ 12) × (12 − month placed in service + 0.5)
    - Example: Placed in service June 15 → (Annual ÷ 12) × (12 − 6 + 0.5) = 6.5 months
4. **Current year depreciation**:
    - If placed in service in the current tax year → use mid-month convention
    - If placed in service in a prior year → full annual depreciation
    - If disposed of during the year → mid-month convention for disposal month
5. Output the depreciation amount as Schedule E Line 18 for each property
6. Show calculation details to user:
   ```
   Depreciation — 123 Main St:
     Cost Basis:        $285,000
     Land Value:       −$ 60,000
     Depreciable Basis: $225,000
     Method:            27.5-year straight-line (residential)
     In-Service:        2019-06-15
     Years in Service:  5.5 (full year)
     Annual Depr:       $  8,181.82
     2024 Deduction:    $  8,181.82 (Line 18)
   ```

---

### Step 8: Generate Reports

Generate XLS files in `Reports/`. **Always regenerate from ALL processed files** (both new and previously processed),
using data from `.processed-files.log` and extracted information.

#### `Reports/schedule-e-draft.csv`

One section per property. Columns: `Property`, `Line`, `Category`, `Amount`, `Details`

Schedule E line reference:

| Line | Category                    | Common Expenses                            |
|------|-----------------------------|--------------------------------------------|
| 3    | Rents received              | Tenant payments, Airbnb income             |
| 4    | Royalties received          | Royalty payments                           |
| 5    | Advertising                 | Listing fees, rental ads                   |
| 6    | Auto and travel             | Business trips, mileage                    |
| 7    | Cleaning and maintenance    | Cleaning service, lawn care, pest control  |
| 8    | Commissions                 | Property manager commissions, leasing fees |
| 9    | Insurance                   | Property insurance, liability, umbrella    |
| 10   | Legal and professional fees | Attorney, accountant, tax prep             |
| 11   | Management fees             | Property management company fees           |
| 12   | Mortgage interest           | Mortgage interest (Form 1098)              |
| 13   | Other interest              | HELOC interest, business loan interest     |
| 14   | Repairs                     | Plumbing, electrical, appliance repair     |
| 15   | Supplies                    | Hardware, paint, cleaning supplies         |
| 16   | Taxes                       | Property tax, special assessments          |
| 17   | Utilities                   | Water, electric, gas, trash, internet      |
| 18   | Depreciation                | Calculated depreciation (see Step 7)       |
| 19   | Other                       | Expenses not fitting lines 5–18            |

#### `Reports/business-trips.csv`

Columns: `Business`, `Trip Name`, `Destination`, `Dates`, `Category`, `Vendor`, `Amount`, `Per Diem Equivalent`,
`Recommended Method`, `Notes`

Include a summary row per trip with:

- Total actual expenses
- Total per diem allowance
- Recommended method (Actual or Per Diem)
- Savings from recommended method

#### `Reports/donations-summary.csv`

Columns: `Date`, `Organization`, `Amount`, `Cash/Non-Cash`, `501c3 Confirmed`, `Description`, `Source File`

#### `Reports/w2-summary.csv`

Columns: `Employer`, `EIN`, `Wages (Box 1)`, `Federal Withheld (Box 2)`, `SS Wages (Box 3)`, `SS Withheld (Box 4)`,
`Medicare Wages (Box 5)`, `Medicare Withheld (Box 6)`, `State`, `State Wages`, `State Withheld`, `Source File`

#### `Reports/1099-summary.csv`

Columns: `Type`, `Payer`, `TIN (last 4)`, `Amount`, `Box`, `Description`, `Source File`

---

### Step 9: Completion Summary

Present a clear summary to the user:

```
=== Tax Organizer — Complete ===

FILES
  Processed (new):      12
  Skipped (existing):    5
  Duplicates found:      2
  Needs manual review:   1 (see Unclassified/)

INCOME & EXPENSES BY PROPERTY
  123 Main St (Residential Rental)
    Rents received:     $ 24,000.00
    Total expenses:     $ 15,432.18
    Depreciation:       $  8,181.82
    Net income (loss):  $    386.00

  456 Oak Ave (Commercial Rental)
    Rents received:     $ 36,000.00
    Total expenses:     $ 12,100.00
    Depreciation:       $  5,128.21
    Net income (loss):  $ 18,771.79

BUSINESS TRIPS
  Property Inspection — Austin, TX (Mar 10–12)
    Actual expenses:    $    842.00
    Per diem allowance: $    716.00
    Recommendation:     Use ACTUAL method (saves $126.00)

  BiggerPockets Conf — Las Vegas, NV (Sep 5–8)
    Actual expenses:    $  1,450.00
    Per diem allowance: $  1,680.00
    Recommendation:     Use PER DIEM method (saves $230.00)

DONATIONS
  Total charitable contributions: $2,500.00
  Cash: $2,000.00 | Non-cash: $500.00

W-2 SUMMARY
  Total wages:          $ 85,000.00
  Total federal withheld: $ 15,300.00

1099 SUMMARY
  1099-NEC:  $12,000.00 (2 forms)
  1099-INT:  $   450.00 (1 form)

REPORTS GENERATED
  Reports/schedule-e-draft.csv
  Reports/business-trips.csv
  Reports/donations-summary.csv
  Reports/w2-summary.csv
  Reports/1099-summary.csv

NEXT STEPS
  1. Review Unclassified/ for 1 file needing manual classification
  2. Verify Schedule E draft against actual IRS Form 1040 Schedule E
  3. Provide reports to your tax preparer
  4. Drop additional documents anytime and re-run to update
```

## Sample Content Reference

These are the annotated example templates the skill uses to explain folder structure and README format to the user
(e.g., when the user asks "what should my business README look like?"). **Do not create a `_sample/` folder or any
sample files on disk** — these are reference templates only.

### Root folder README example

```markdown
# Tax Documents — 2025

## Instructions

This folder organizes your tax documents for the 2025 tax year.

### How to use

1. **Drop receipts and documents** into the tax year folder
   — or into any existing business subfolder
2. **Run the skill** — it scans your documents, identifies business
   lines, and guides you through setup
3. **Add more documents anytime** — the skill detects new files and
   only processes what's changed

### Special folders

- `W2s/` — Drop W-2 forms here (organized for reference)
- `1099s/` — Drop all 1099 forms here
- `Donations/` — Drop charitable contribution receipts here

## Filing Information

- **Tax Year**: 2025
- **Filing Status**: Married Filing Jointly
- **Preparer**: Jane Smith
```

### Business folder README example

```markdown
# 789 Example Boulevard Rental

## Business Information

- **Type**: Residential Rental
  <!-- Options: Residential Rental, Commercial Rental, Royalties,
       Partnership (K-1), S-Corp (K-1), Freelance/Self-Employment,
       Farm Rental -->
- **EIN**: 98-7654321
  <!-- Your Employer Identification Number, if you have one -->
- **Address**: 789 Example Blvd, Denver, CO 80202
- **Ownership**: 100%
  <!-- Your ownership percentage. Use 50% for 50/50 partnerships, etc. -->

## Property Details

<!-- Only include this section for rental properties -->

- **In-Service Date**: 2020-03-01
  <!-- The date you started renting the property -->
- **Cost Basis**: $350,000
  <!-- Total purchase price including closing costs -->
- **Land Value**: $75,000
  <!-- Assessed land value (check your property tax statement) -->
- **Depreciation Method**: Straight-line (27.5 years residential)
  <!-- Residential: 27.5 years | Commercial: 39 years -->

## Business Trips

<!-- List any trips related to this property.
     The skill will automatically match receipts that fall within
     these date ranges to this trip. -->

### Property Visit — Denver, CO

- **Dates**: April 5–7, 2024
- **Purpose**: Tenant move-out inspection and unit turnover

## Important Events

<!-- Note anything that affects how expenses are categorized -->

- **2024-04-10**: Full interior repaint — maintenance ($3,200)
- **2024-08-15**: New roof — capital improvement ($12,000)
  <!-- Capital improvements are depreciated separately, not expensed -->

## Notes

Property managed by Denver Property Mgmt Co.
Monthly management fee: $175 (2% of rent).
```

## Edge Cases

- **Multi-page PDF**: Treat as a single document. Extract data from all pages.
- **Image quality**: If an image/scan is too low quality to read, flag it for manual review rather than guessing.
- **Multiple amounts**: Some invoices have multiple line items. Extract the total and note individual items in the
  description.
- **Foreign currency**: Flag for manual review. Note the currency and suggest the user provide the USD equivalent.
- **Personal vs. business**: If a receipt could be personal or business (e.g., Home Depot), classify based on context (
  property address, trip dates) or ask the user.
- **Partial year ownership**: Use mid-month convention for depreciation. Prorate rental income if property was not
  rented the full year.
- **Multiple properties on one receipt**: If a vendor receipt covers multiple properties, ask the user how to split the
  expense.
- **Duplicate file names**: The hash-based system handles this — same content with different names is flagged as a
  duplicate.
- **Empty or corrupted files**: Flag for manual review. Do not process.
- **Spreadsheets (XLS/XLSX)**: Use the `/xlsx` skill to read and process Excel files. These might be bank statements,
  accounting exports, or property management reports with multiple transactions. CSV files can be read directly.

## Security Notes

- **Never store full SSNs or TINs** — only reference last 4 digits in reports
- **Do not transmit** tax documents outside the local filesystem
- **Preserve originals** — always copy, never move or delete source files
- All processing is local — no external API calls except GSA per diem rate lookup

---

## Cryptocurrency & Mining Income (Schedule C)

### Spreadsheet Pool Exports
Mining pools typically export daily payout data as `.xlsx` files. Each row is one settlement.
Use the `/xlsx` skill to read these. Key columns to extract:

- `Time of Settlement` — income date (use date portion only; UTC offset doesn't change the tax date)
- `Amount (Total)` — BTC received that day (this is the income, at FMV on that date)
- `Avg. Hashrate Per Day` — informational; document for activity records
- `Description` — payment scheme (FPPS, PPLNS, etc.)

### USD FMV Conversion
Each day's BTC payout is **ordinary income at the fair market value on the date received** (IRS Rev. Rul. 2023-14).
To calculate USD income:

1. Fetch daily BTC closing prices from Yahoo Finance:
   ```
   https://query1.finance.yahoo.com/v8/finance/chart/BTC-USD?interval=1d&period1=UNIX_TS&period2=UNIX_TS
   ```
   - Replace `UNIX_TS` with Unix timestamps for your date range
   - CoinGecko free API returns 401 — prefer Yahoo Finance
2. Match each settlement date to the closing price for that date
3. `USD income = BTC amount × closing price`
4. Sum for total Schedule C mining income

Generate a `Reports/btc-mining-income.csv` with one row per settlement date showing: date, BTC amount, USD price, USD income.

### Cost Basis of Unsold BTC
The FMV at time of receipt becomes the cost basis for any BTC held and later sold (reported on Schedule D / Form 8949 in the year of sale). The per-day breakdown in `btc-mining-income.csv` is the source for this basis tracking.

### Business Closure Mid-Year
If the business closed partway through the year, confirm the export covers the full active period. A partial-year export can be complete if the business stopped operating before year-end — **do not flag it as missing data** once the user confirms the closure date.

### Additional Reports for Crypto Businesses
Generate these extra reports alongside the standard set:

- **`Reports/schedule-c-draft.csv`** — Income, expenses, net profit/loss for each Schedule C business
- **`Reports/btc-mining-income.csv`** — Daily breakdown: date, BTC, USD price, USD income
- **`Reports/bitcoin-mining-detail.csv`** — Summary metadata (total BTC, total USD, price source, action items)

---

## International Business Travel

### Per Diem Rate Source
For **domestic US travel**, use GSA per diem rates (gsa.gov).
For **international travel**, use **US State Department / DoD per diem rates** — not GSA.

- Lookup URL: `https://www.perdiem101.com/oconus/{YEAR}/{CITY-COUNTRY-rate}` (reliable mirror)
- Official source: `allowances.state.gov` (may be slow or return errors for some country codes)
- Rates include: lodging (per night) and M&IE (meals and incidental expenses, per day)
- Rates are typically published per city and updated periodically (check effective date)

### First/Last Day Rule (International)
Same as domestic: 75% of M&IE on the first and last calendar day in the foreign country.

```
First day in foreign country:   M&IE × 75%
Full days in foreign country:   M&IE × 100% each
Last day in foreign country:    M&IE × 75%
```

Do **not** count the US departure or return days as foreign-country days unless the taxpayer
overnights abroad on that day.

### Stayed with Family / Friends — $0 Lodging
When the taxpayer stayed with family or friends instead of a hotel:
- Lodging deduction = **$0** (no actual expense)
- M&IE per diem still applies for meals — use per diem method when no meal receipts exist
- The per diem lodging rate is irrelevant (not deductible without actual cost)
- Note this clearly in `business-trips.csv` so the accountant is not confused by the $0

### Missing Receipts — Per Diem vs. Actual
If meal receipts are unavailable, recommend the **per diem method** for M&IE.
Transportation and airfare always use actual cost (per diem does not cover these).
Document the choice in the report — IRS requires consistency within a single trip.

---

## Combined Property Tax Payments

### Multiple Parcels on One Payment
County tax portals often allow paying multiple parcels in a single transaction.
When the same payment confirmation covers more than one property (e.g., a primary residence
and a rental), the document shows parcel numbers and amounts **but not property addresses**.

**Do not guess which parcel belongs to which property from the payment PDF alone.**

Steps to resolve:
1. Ask the user to upload the official county tax **bills** (not the payment confirmation) —
   these always show the situs (property) address alongside the parcel number.
2. Alternatively, look up each parcel on the county assessor's website.
3. Once confirmed, document the mapping in each property's `README.md`.

### Near-Duplicate Payment Confirmations
The same payment event can produce **two different PDF files** with **different SHA-256 hashes**:
- One saved from the payment portal immediately after submitting
- One printed later from the payment history page

These are not true duplicates (different hashes) but represent the same deduction.
Flag as **near-duplicate**, keep the cleaner copy in the property folder, and move the other to `Unclassified/`
with a note.

### Parcel → Property Assignment Tip
When unsure, compare the **assessed value / tax amount** to the **property values**:
- Higher-value property → typically higher annual tax bill
- Lower-value property → typically lower annual tax bill

Use this as a sanity check, but **always confirm with actual tax bills** before finalizing reports.

---

## Additional Edge Cases (Discovered in Practice)

### Property Tax: Installment Timing
Many counties collect property tax in two installments that may **cross tax years**:
- Installment 1 (due ~November): often paid in September of the same tax year
- Installment 2 (due ~April): paid in the following calendar year

Both installments paid in the **same calendar year** are deductible in that year for
cash-basis taxpayers. Always check payment dates, not due dates.

### Personal vs. Business Software Licenses
Some software vendors sell "Personal" and "Business" tiers. A "Personal" license used
exclusively for business is **still deductible** — the license type doesn't determine
deductibility; actual use does. Ask the user to confirm usage percentage and note it in the report.

### Mortgage Interest Deduction Limit (Primary Residence)
For MFJ filers, mortgage interest is deductible only on the first $750,000 of principal.
If the outstanding balance exceeds $750,000, prorate the deductible interest:

```
Deductible Interest = Total Interest × ($750,000 / Average Outstanding Balance)
```

Flag this in the Schedule A notes and provide the calculation to the preparer.

### Form 1095-A (Health Insurance Marketplace)
The 1095-A is not a 1099 but should be classified alongside income forms in `1099s/`.
Key fields to extract:
- Monthly enrollment premium (Box A)
- Monthly SLCSP premium (Box B) — used to calculate allowed credit on Form 8962
- Advance Premium Tax Credit received (Box C) — must be reconciled on **Form 8962**

Always flag that **Form 8962 is required** when a 1095-A is present.
If APTC was received for only part of the year (e.g., Jan–Oct), note the reason and confirm with the user.

### Investment Interest Expense (Margin Interest)
Margin interest paid to a broker is reported on the year-end statement (not a 1099).
It is **potentially deductible** on Schedule A, Form 4952, up to net investment income.
When present, include it in the 1099 summary with a note about deductibility and Form 4952.

### Corrected 1099s
A corrected 1099 (marked "CORRECTED" on the form) **supersedes** the original.
When both exist:
- Use the corrected version for all report amounts
- Keep both files organized (rename with "CORRECTED" in the filename)
- Note the correction in the 1099 summary

### Taiwan / Foreign E-Invoices
Taiwan uses a government e-invoice system (電子發票 / GUI). These PDFs look like
foreign-language receipts with a QR code and a random 4-digit verification code.
Extract: vendor name, date (in ROC calendar format — add 1911 to convert to AD year),
amount in New Taiwan Dollars (NT$), and convert to USD using the exchange rate on that date.

---

## Technical Notes

### File Hashing with Special Characters
Use Python `hashlib` (not shell `sha256sum` with `xargs`) when file paths contain:
- Parentheses, ampersands, accented characters, CJK characters, or long spaces
- Shell `xargs sha256sum` silently produces the empty-file hash for unescaped names

```python
import hashlib, os
h = hashlib.sha256()
with open(path, 'rb') as f:
    h.update(f.read())
digest = h.hexdigest()
```

### Python Dependencies
`pandas` and `openpyxl` may not be pre-installed. Install before reading `.xlsx` files:
```bash
pip install pandas openpyxl --quiet
```

### Working Directory
Always confirm the working directory before running file scans. The `.processed-files.log`
and all relative paths in the log must match the directory from which the skill runs.
Use `os.walk('.')` from the correct root, not an absolute path, to keep log entries consistent.

### BTC Price API — Yahoo Finance
```
GET https://query1.finance.yahoo.com/v8/finance/chart/BTC-USD
    ?interval=1d
    &period1={unix_timestamp_start}
    &period2={unix_timestamp_end}
```
Returns OHLCV data. Use the `close` price array. No API key required.
CoinGecko free tier returns HTTP 401 — avoid for automated use.
