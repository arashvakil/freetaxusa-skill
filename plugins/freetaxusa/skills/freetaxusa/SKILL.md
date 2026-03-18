---
name: freetaxusa
description: Navigate FreeTaxUSA to verify entered tax data against source documents (W-2s, 1099s, spreadsheets), resolve Tax Return Alerts, enter or correct values in the browser, and run planning tools like estimated tax and extension calculations. Use when the user wants to check, fix, complete, or plan around their FreeTaxUSA return.
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, Bash, Write, mcp__claude-in-chrome__tabs_context_mcp, mcp__claude-in-chrome__find, mcp__claude-in-chrome__computer, mcp__claude-in-chrome__javascript_tool, mcp__claude-in-chrome__read_page, mcp__claude-in-chrome__navigate
---

# FreeTaxUSA Skill

---

## Commands

The skill supports the following explicit commands. Detect the command from the user's message:

| Command | What it does |
|---------|-------------|
| `/freetaxusa` | Interactive — ask check vs file if ambiguous |
| `/freetaxusa check` | Read-only audit. Compare every document to FreeTaxUSA. No changes made. Output audit report. |
| `/freetaxusa file` | Full run — verify, correct mismatches, resolve alerts, proceed toward filing |
| `/freetaxusa alerts` | Jump directly to Tax Return Alerts and resolve them |
| `/freetaxusa status` | Show what's been verified, what's pending, current balance due or refund |
| `/freetaxusa w2` | Verify W-2 entries only |
| `/freetaxusa schedule-e` | Verify Schedule E rental properties only |
| `/freetaxusa schedule-c` | Verify Schedule C business income only |
| `/freetaxusa investments` | Verify 1099-DIV, 1099-B, capital gains, and carryovers |
| `/freetaxusa k1` | Walk through K-1 partnership entries one by one |
| `/freetaxusa compare` | Generate year-over-year comparison from CLAUDE.md data |
| `/freetaxusa estimate` | Calculate Q1–Q4 estimated tax payments for next year |
| `/freetaxusa extension` | File Form 4868 extension and calculate how much to pay with it |
| `/freetaxusa whatif` | Run what-if scenarios (401k contribution, property sale, income change) |

---

## Step 0: Startup Sequence

Perform these steps in order before doing anything else.

### 0a. Detect command and mode

Parse the user's invocation to identify the command (see table above).

For `/freetaxusa` with no argument, detect mode:
- **`check` mode** — Read-only verification. Compare source documents to FreeTaxUSA. Report mismatches. Make **NO changes** to FreeTaxUSA. Output an audit report.
- **`file` mode** — Full correction + submit. Verify, correct mismatches, resolve alerts, and proceed toward filing.

If ambiguous, ask: *"Should I check and report only, or also make corrections and proceed toward filing?"*

Section-specific commands (`w2`, `schedule-e`, etc.) skip to that section directly — do not run the full workflow.

Planning commands (`estimate`, `extension`, `whatif`, `compare`) do not open FreeTaxUSA — they work from documents and CLAUDE.md data only.

### 0b. Check for checkpoint

Look for a checkpoint file in the tax documents folder:

```bash
ls "$(pwd)/.freetaxusa-checkpoint.json" 2>/dev/null
```

If found, read it and ask: *"A previous session was in progress (last run: DATE, mode: MODE, completed: SECTIONS). Resume from where it left off, or restart from the beginning?"*

Checkpoint file structure:
```json
{
  "completed": ["w2", "schedule_e", "investments"],
  "lastRun": "2026-03-18",
  "mode": "file",
  "taxYear": "2025",
  "taxpayer": "Your Name"
}
```

Write checkpoint updates after each major section completes.

### 0c. Load context

1. Load browser tools: use ToolSearch to load `mcp__claude-in-chrome__tabs_context_mcp`, then get the current tab. (Skip for planning commands.)
2. Read `CLAUDE.md` in the current directory if present.
3. Locate the prior year Form 1040 PDF in the tax folder (look for filenames containing "1040", "federal", "return", or the prior tax year). If found, note its path. If not found, remind the user:
   > ⚠️ No prior year Form 1040 found. This is needed for Form 2210 safe harbor (prior AGI + total tax), capital loss carryovers, and year-over-year comparison. Add it to your tax folder or paste the key figures into CLAUDE.md.
4. Read the source documents needed for the requested command.

### 0d. First-Time Setup (no CLAUDE.md present)

If no `CLAUDE.md` exists, run the questionnaire before proceeding:

```
1. What is your name and filing status? (Single / MFJ / MFS / HOH)
2. What tax year are you filing?
3. Do you have your prior year Form 1040 available? (PDF or printed copy)
   → If yes: where is the file, or paste the key figures: Line 11 (AGI) and Line 24 (Total Tax).
   → This is needed for Form 2210 safe harbor, estimated payment calculations, and year-over-year comparison.
   → If no: we'll proceed but some checks will be skipped.
4. List all W-2 employers (person, employer name, EIN if known).
5. Do you have rental properties? If yes, list addresses and ownership %.
6. Are you a Real Estate Professional (>750 hrs/year in real estate)?
7. Do you have investment accounts with 1099-INT, 1099-DIV, or 1099-B?
   → If yes: do you have capital loss carryovers from last year? (Prior Schedule D, Line 6 and Line 14)
8. Do you have a Schedule C business?
9. Any K-1s from partnerships or S-corps?
10. Where are your tax documents stored (local folder path)?
```

Use answers to create a minimal `CLAUDE.md`. Refer to [workspace-template.md](workspace-template.md).

---

## Critical: How to enter values in FreeTaxUSA

FreeTaxUSA is a React SPA. **Standard click-and-type does not work.** You must use the React native input setter:

```javascript
function setReactField(id, value) {
  const el = document.getElementById(id);
  if (!el) return `NOT FOUND: ${id}`;
  const setter = Object.getOwnPropertyDescriptor(
    window.HTMLInputElement.prototype, 'value'
  ).set;
  setter.call(el, String(value));
  el.dispatchEvent(new Event('input', { bubbles: true }));
  el.dispatchEvent(new Event('change', { bubbles: true }));
  return `OK: ${id} = ${value}`;
}
```

Always use `javascript_tool` to run this. Never use `type` action on FreeTaxUSA inputs directly.

To find field IDs: use `javascript_tool` to query `document.querySelectorAll('input')` and inspect `id`, `name`, and `value` attributes.

---

## Critical: How to navigate safely

FreeTaxUSA's nav menu items are buttons, not links. Use JavaScript to click them by exact text:

```javascript
const buttons = Array.from(document.querySelectorAll('a, button'));
const target = buttons.find(b => b.textContent.trim() === 'EXACT TEXT HERE');
target ? (target.click(), 'clicked') : 'not found'
```

**Never** use `buttons[index]` — index-based clicking risks hitting Sign Out or other destructive buttons.

**Safe navigation targets (exact text):**
- Section menus: `Personal`, `Income`, `Deductions / Credits`, `Misc`, `Summary`, `State`, `Final Steps`
- Final Steps submenu: `Federal Payment`, `State Refund`, `Refund / Payment Method Summary`, `Driver's License`, `Tax Return Alerts`
- Income submenu: `W-2 Income`, `Interest Income`, `Dividend Income`, `Your Rental Income (Schedule E)`, `Partnership Income (Schedule E)`, `Business Income (Schedule C)`
- Payments submenu: `Estimated Tax Payments`

---

## Runtime Field Discovery

Field IDs in FreeTaxUSA may change between tax years. If a hardcoded ID returns `NOT FOUND`, use label-proximity discovery:

```javascript
function findFieldByLabel(labelText) {
  const labels = Array.from(document.querySelectorAll('label'));
  const label = labels.find(l => l.textContent.includes(labelText));
  if (label?.htmlFor) return document.getElementById(label.htmlFor);
  const parent = label?.closest('div, tr, li, fieldset');
  return parent?.querySelector('input, select, textarea');
}
const el = findFieldByLabel('Box 14');
console.log(el?.id, el?.value);
```

Always verify the discovered element matches the expected field before writing. Log new IDs to [common-field-ids.md](common-field-ids.md).

To enumerate all inputs on the current page:
```javascript
Array.from(document.querySelectorAll('input, select')).map(el => ({
  id: el.id, name: el.name, type: el.type, value: el.value,
  label: document.querySelector(`label[for="${el.id}"]`)?.textContent?.trim()
}))
```

---

## PDF and Document Parsing

### Preferred: pdftotext (Bash)

```bash
pdftotext -layout "/path/to/W2.pdf" -
```

### Fallback: Read tool (multimodal)

```
Read tool: file_path = "/path/to/document.pdf"
```

### Key box locations

| Form | Key Boxes |
|------|-----------|
| W-2 | Box 1 (wages), Box 2 (fed withheld), Box 12 (codes), Box 14 (other), Box 15-17 (state) |
| 1099-INT | Box 1 (interest), Box 4 (fed withheld), Box 8 (tax-exempt), Box 11 (bond premium) |
| 1099-DIV | Box 1a (ordinary divs), Box 1b (qualified), Box 2a (cap gain dist), Box 7 (foreign tax) |
| 1099-B | Proceeds, Cost basis, Short/Long term, Wash sale disallowed |
| 1098 | Box 1 (mortgage interest), Box 5 (mortgage insurance), Box 10 (real estate taxes) |
| SSA-1099 | Box 3 (gross benefits), Box 4 (repaid), Box 5 (net = Box3 - Box4) |
| K-1 (1065) | Box 1 (ordinary income/loss), Boxes 2-12 (separately stated items) |

---

## Workflow: `/freetaxusa check`

Read-only. Make zero changes to FreeTaxUSA.

1. Load all source documents from the tax folder
2. Navigate to each section in FreeTaxUSA and read current values via `javascript_tool`
3. Compare source document values to FreeTaxUSA values field by field
4. Log every mismatch with: field name, source value, FreeTaxUSA value, severity (blocking / informational)
5. Run Smart Checks (see below)
6. Generate audit report — do not write any values

---

## Workflow: `/freetaxusa file`

Full end-to-end run.

1. Run all section verifications (W-2, 1099s, Schedule E, Schedule C, investments, K-1s)
2. Correct every mismatch using the React setter
3. Write checkpoint after each section
4. Run Pre-Submit Final Validation
5. Navigate to Tax Return Alerts and resolve all red alerts; continue past yellow
6. Run Smart Checks
7. Generate audit report with YoY comparison

---

## Workflow: `/freetaxusa status`

Without opening FreeTaxUSA, read the checkpoint file and CLAUDE.md and report:

- Sections completed vs pending
- Last run date and mode
- Current projected balance due or refund (from CLAUDE.md if available)
- Any outstanding K-1s or documents not yet received
- Days until filing deadline

---

## Workflow: `/freetaxusa w2`

1. Read all W-2 PDFs in the tax folder
2. Navigate to Income → W-2 Income
3. For each W-2, find the Edit button by person name (not index)
4. Compare each box: use `javascript_tool` to read current values
5. In `check` mode: log mismatches. In `file` mode: correct using React setter.

**Box 14 special handling:**
- **NY PFL / FLI** → enters on *two* screens: W-2 Box 14 field AND the SDI/FLI screen under State
- **414(H)** → Box 14 Other; adds back on NY state return automatically
- **IRC 125** → Box 14 Other; check State → NY → Miscellaneous Additions — FreeTaxUSA may not auto-populate
- **SDI (CA)** → California State Disability Insurance; Box 14 or Box 19; enters on CA SDI screen
- **SDI (NJ)** → New Jersey State Disability Insurance; Box 14; dedicated NJ screen under State

**Multi-state:** If W-2 has multiple Box 15-17 rows, enter each state separately.

---

## Workflow: `/freetaxusa schedule-e`

For each rental property:
1. Read income/expense data from spreadsheet or PDF
2. Navigate to Income → Your Rental Income (Schedule E)
3. Verify rents, mortgage interest, taxes, insurance, utilities, repairs, depreciation
4. Check passive loss carryovers match Form 8582 from prior return
5. Verify REP checkbox status

**Standard residential (27.5yr GDS):**
- Asset Type: `Real Estate → Residential Rental Property`

**ADS / non-standard (e.g., 40yr):**
- Asset Type: **Other → "Other (you select the depreciation life, method, and convention)"**
- Method: ADS, Recovery Period: 40yr, Convention: Mid-Month
- Yellow alert "Rental Missing Depreciable Rental Property" is harmless — continue
- Land must be entered as a separate non-depreciable asset

**Partial ownership:** FreeTaxUSA has no ownership % field. Enter only the taxpayer's proportional share pre-divided.

---

## Workflow: `/freetaxusa schedule-c`

1. Read Schedule C income/expense records (spreadsheet, invoices, etc.)
2. Navigate to Income → Business Income (Schedule C)
3. Verify gross income, each expense category, home office if applicable
4. Check QBI eligibility — flag if taxpayer may qualify for the 20% Section 199A deduction
5. Verify SE tax calculation on Schedule 2

---

## Workflow: `/freetaxusa investments`

1. Read all 1099-DIV, 1099-INT, and 1099-B documents
2. Navigate to Interest Income, Dividend Income, and Capital Gains sections
3. Verify ordinary/qualified dividends, foreign tax paid, capital gain distributions
4. For 1099-B: verify proceeds, cost basis, short/long term classification, wash sale disallowed amounts
5. Check capital loss carryovers from prior return match Schedule D
6. Flag if net investment income (dividends + cap gains + rental income) may trigger NIIT

**Notes:**
- Joint accounts: confirm which spouse the 1099 is issued to
- Tax-exempt interest (Box 8): enter separately — some states tax federally-exempt interest
- Bond premium (Box 11): reduces taxable interest

---

## Workflow: `/freetaxusa k1`

Navigate to: Income → Partnership Income (Schedule E)

For each K-1:
1. Enter entity name and EIN (K-1 Box D)
2. Enter ordinary business income/loss (Box 1)
3. Enter separately stated items: Box 2 (rental), Box 5 (interest), Box 6 (dividends), Box 8 (ST cap gain), Box 9a (LT cap gain), Box 12 (Section 179), Box 13 (other deductions), Box 14 (SE earnings)
4. Enter at-risk / passive activity status
5. Check for prior-year passive loss carryovers from Form 8582
6. If K-1 not yet received, note it and recommend `/freetaxusa extension`

---

## Workflow: `/freetaxusa alerts`

Navigate directly to: Final Steps → Tax Return Alerts

**Red alerts** must be fixed. **Yellow alerts** can be continued past.

| Alert | Resolution |
|-------|-----------|
| Rental Missing Depreciable Rental Property | Harmless if using ADS "Other" workaround — continue |
| You might owe an underpayment penalty | Misc → Underpayment Penalty → Form 2210; enter prior year AGI and tax |
| Federal Payment Less/More than Tax Owed | Final Steps → Federal Payment → update direct debit amount |
| Tax Penalty Date Paid | Informational — defaults to April 15, acceptable |
| NYC IRC 125 addition | State → NY → Miscellaneous Additions; update field `amount_472` |

---

## Workflow: `/freetaxusa compare`

Does not open FreeTaxUSA.

**Data sources (in priority order):**
1. Prior year Form 1040 PDF — read directly if available in the tax folder. Extract: AGI (Line 11), Total Tax (Line 24), Taxable Income (Line 15), Wages (Schedule 1 or W-2 total), Federal Withheld (Line 25), refund or amount owed (Line 35a / Line 37).
2. CLAUDE.md prior year figures — use if PDF is not available.
3. If neither exists, ask: *"Do you have your prior year Form 1040 available? I need it for the comparison."*

**Steps:**
1. Load prior year figures from PDF or CLAUDE.md
2. Load current year figures from FreeTaxUSA Summary page or CLAUDE.md
3. Generate a full year-over-year table:

| Item | Prior Year | Current Year | Change | Driver |
|------|-----------|-------------|--------|--------|
| Wages | | | | |
| Other income | | | | |
| AGI | | | | |
| Deduction | | | | |
| Total Tax | | | | |
| Federal Outcome | | | | |
| State Outcome | | | | |
| CPA / prep cost | | | | |
| Errors found | | | | |

3. Identify the top 2-3 drivers of year-over-year change and explain them in plain language
4. Offer to update CLAUDE.md with current year final figures

---

## Workflow: `/freetaxusa estimate`

Calculates Q1–Q4 estimated tax payments for the **next** tax year. Does not open FreeTaxUSA.

**Inputs needed** (read from CLAUDE.md or ask user):
- Current year total tax (Form 1040 Line 24)
- Current year AGI
- Expected income changes for next year (raises, new income sources, property sales)
- Filing status

**Safe harbor calculation:**
- AGI ≤ $150K: pay 100% of current year tax across 4 equal installments
- AGI > $150K: pay 110% of current year tax across 4 equal installments
- Alternative: pay 90% of next year's estimated tax

**Output:**

```
Estimated Tax Payments — [Next Tax Year]

Safe Harbor (based on [current year] tax of $XX,XXX):
  Q1 due April 15:    $X,XXX
  Q2 due June 15:     $X,XXX
  Q3 due September 15: $X,XXX
  Q4 due January 15:  $X,XXX
  Total:              $XX,XXX

Withholding check: If your employer withholds $XX,XXX/year, you owe
$X,XXX in additional estimated payments.

Recommendation: [adjusted amounts if income is expected to change significantly]
```

Offer to save these figures to CLAUDE.md under an "Estimated Payments" section.

---

## Workflow: `/freetaxusa extension`

Files Form 4868 and calculates the payment due. Does not require FreeTaxUSA to be open.

**Extension facts:**
- Form 4868 gives 6 additional months to FILE — not to pay
- Tax owed is still due on the original deadline (April 15)
- Penalty for late payment: 0.5% per month on unpaid balance
- Penalty for late filing (no extension): 5% per month — always file the extension

**Steps:**
1. Read current year projected tax from CLAUDE.md or ask user for estimate
2. Read total withholding to date
3. Calculate: **Amount to pay with extension = projected tax − withholding already paid**
4. If amount ≤ 0, no payment needed with extension — just file the form
5. Navigate to FreeTaxUSA → Final Steps → File Extension (or direct user to IRS Direct Pay if FreeTaxUSA doesn't support it)

**Output:**

```
Extension Calculation

Projected tax:          $XX,XXX
Withholding paid:      -$XX,XXX
Amount due with Form 4868: $X,XXX

New filing deadline: October 15, [year]
Payment still due:   April 15, [year]

[If amount > 0]: Pay via IRS Direct Pay at directpay.irs.gov to avoid penalties.
```

---

## Workflow: `/freetaxusa whatif`

Runs what-if tax scenarios without opening FreeTaxUSA. Uses current year figures from CLAUDE.md as the baseline.

**Supported scenarios** (detect from user's message or offer a menu):

1. **401k / 403b contribution increase** — how much does maxing out reduce tax?
2. **Property sale** — estimated capital gain tax on sale of a property at a given price
3. **Income change** — how does a raise or new income source affect tax bracket and liability?
4. **Rental property addition** — how does adding a property affect Schedule E and passive losses?
5. **Filing status change** — MFJ vs MFS comparison
6. **Additional withholding** — how much extra per paycheck to eliminate estimated payments?

For each scenario:
1. State the assumption clearly
2. Calculate the estimated tax impact using current year marginal rate from CLAUDE.md
3. Show: old liability vs new liability, delta, and the effective rate change
4. Note: "This is an estimate — actual results depend on your full return"

---

## Smart Checks (run automatically in `check` and `file` modes)

After all sections are verified, run these flags before generating the audit report:

### AMT Check
If taxable income exceeds ~$220K (single) or ~$267K (MFJ), flag for potential Alternative Minimum Tax. FreeTaxUSA calculates Form 6251 automatically, but confirm it's not showing unexpected AMT on the Summary page.

### NIIT Check
If AGI exceeds $200K (single) or $250K (MFJ) AND there is investment income (dividends, capital gains, rental income), confirm Net Investment Income Tax (3.8%) appears on Schedule 2, Line 12.

### QBI Check
If there is Schedule C income or K-1 income from a pass-through entity, flag that the taxpayer may qualify for the Section 199A 20% deduction. Check if FreeTaxUSA is calculating it on Schedule 1.

### Withholding Gap Alert
Compare total withholding (W-2 Box 2 + any estimated payments) to total tax liability. If gap exceeds $1,000:
> ⚠️ Withholding shortfall of $X,XXX detected. You may owe an underpayment penalty. Run `/freetaxusa alerts` to address Form 2210.

### Missing Document Detector
Cross-reference CLAUDE.md's list of expected documents against what has been verified. Report any gaps:
> ⚠️ The following expected documents were not verified: [list]

Also flag if no prior year Form 1040 was found:
> ⚠️ Prior year Form 1040 not found. Add it to your tax folder to enable year-over-year comparison, Form 2210 safe harbor calculation, and carryover verification.

---

## Workflow: Form 2210 (Underpayment Penalty)

Inputs needed (from CLAUDE.md or user):
- Prior year AGI (Form 1040, Line 11)
- Prior year total tax
- If AGI > $150K: safe harbor = 110% × prior year total tax

Field IDs on the prior year info screen:
- `prior_agi`
- `prior_tax_after_credits`
- `prior_additional_medicare_tax`
- `prior_net_investment_income_tax`

---

## Pre-Submit Final Validation

Required in `file` mode before proceeding to filing.

```javascript
const fields = Array.from(document.querySelectorAll('input[type="text"], input[type="number"]'));
fields.map(el => ({ id: el.id, value: el.value }))
```

Re-navigate to each completed section and spot-check at least 3 key fields. Document all re-reads in the audit report as "Verified" or "Re-corrected." Do not proceed to Final Steps until all values confirm.

---

## Generate Audit Report

**File path:** `{tax-docs-folder}/audit-YYYY-MM-DD.md`

```markdown
# FreeTaxUSA Audit Report

**Date:** YYYY-MM-DD
**Command:** /freetaxusa [command]
**Mode:** check | file
**Tax Year:** 20XX
**Taxpayer:** [Name]
**Return:** Federal + [State]

## Documents Read
- [list all source documents]

## Field Verification Table

| Field | Section | Source Value | FreeTaxUSA Value | Status |
|-------|---------|-------------|-----------------|--------|

## Smart Check Results

| Check | Result | Notes |
|-------|--------|-------|
| AMT | Clear / ⚠️ Review | |
| NIIT | Clear / ⚠️ Applies | |
| QBI | Clear / ⚠️ May apply | |
| Withholding gap | Clear / ⚠️ $X shortfall | |
| Missing documents | Clear / ⚠️ [list] | |

## Alerts

| Alert | Type | Resolution |
|-------|------|-----------|

## Pre-Submit Validation

All written fields re-verified: YES / PARTIAL / NO

## Year-Over-Year Comparison

| Item | Prior Year | Current Year | Change |
|------|-----------|-------------|--------|

## Notes
```

---

## Workspace File (CLAUDE.md)

See [workspace-template.md](workspace-template.md) for the full template. Lives at the root of the user's tax folder and is loaded automatically each session.

---

## Reference

- [react-inputs.md](react-inputs.md) — React SPA form automation reference
- [common-field-ids.md](common-field-ids.md) — FreeTaxUSA field IDs by section
- [workspace-template.md](workspace-template.md) — CLAUDE.md starter template
