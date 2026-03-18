# Tax Workspace Template (CLAUDE.md)

Copy this file to the root of your tax folder as `CLAUDE.md`. Fill in your details. Claude Code loads this automatically at the start of every session.

---

```markdown
<!-- SETUP CHECKLIST
Complete this section when starting a new tax year. Delete or keep as reference.

[ ] Prior year Form 1040 PDF available? Add to tax folder — needed for Form 2210, YoY comparison, carryover verification.
[ ] Filing status: Single / MFJ / MFS / HOH
[ ] Real Estate Professional? Y/N (if yes, all rental losses are deductible against ordinary income)
[ ] Standard or itemized deduction? (compare Schedule A total vs. standard deduction)
[ ] Rentals? How many properties, ownership % for each?
[ ] K-1s expected? From which entities? (partnership, S-corp, trust)
[ ] Estimated tax payments made during the year? Y/N (enter before calculating balance due)
[ ] Prior year passive loss carryovers? Pull from prior Form 8582.
[ ] Capital loss carryovers? Pull from prior Schedule D (Line 6 ST, Line 14 LT).
[ ] State return: NY / CA / other? Multi-state?
-->

# Tax Workspace — [YOUR NAME]

## Me
**[Full Name]** — [email]. Working on [personal / family / business] taxes.

## People
| Who | Role |
|-----|------|
| **[Spouse name]** | Spouse |
| **[Parent name]** | Parent (if helping with their return) |

## Entities
| Name | What |
|------|------|
| **[Name] & [Spouse]** | Joint personal return (MFJ) |
| **[LLC Name]** | Business entity (Schedule C) |

## Properties
| Property | Lender | Ownership | Notes |
|----------|--------|-----------|-------|
| **[Address]** | [Bank] | 100% | Primary residence |
| **[Address]** | [Bank] | [%] | Rental |

## Filing Status History
| Year | Status | Date Filed | Method |
|------|--------|------------|--------|
| [YEAR] | MFJ / Single / MFS / HOH | [date or "extended"] | FreeTaxUSA / CPA / other |

## Prior Years
| Year | AGI | Total Tax | Federal Outcome | State Outcome | CPA/Tool Cost | Errors Found |
|------|-----|-----------|-----------------|---------------|---------------|--------------|
| [YEAR] | $[amount] | $[amount] | Owed $[amount] / Refund $[amount] | Owed $[amount] / Refund $[amount] | $[amount] | [brief note or none] |

Notes on prior year figures:
- Pull AGI and Total Tax from Line 11 and Line 24 of the prior year 1040.
- These are required for Form 2210 safe harbor (110% × prior year total tax if AGI > $150K).

## W-2s ([TAX YEAR])
| Person | Employer | EIN | Box 1 Wages | Box 2 Fed Withheld |
|--------|----------|-----|------------|---------------------|
| [Name] | [Employer] | [EIN] | $[amount] | $[amount] |

### Box 14 ([Name])
| Item | Amount | FreeTaxUSA Field |
|------|--------|-----------------|
| NY PFL | $[amount] | FLI Box 14 field |
| 414(H) | $[amount] | Box 14 Other |
| IRC 125 | $[amount] | Box 14 Other |

## Interest Income ([TAX YEAR])
| Source | Recipient | Box 1 Amount |
|--------|-----------|-------------|
| [Institution] | [Name] | $[amount] |

## Dividends & Capital Gains ([TAX YEAR])
| Source | Ordinary Divs | Qualified Divs | Cap Gain Dist | Foreign Tax |
|--------|--------------|----------------|---------------|-------------|
| [Institution] | $[amount] | $[amount] | $[amount] | $[amount] |

### Capital Loss Carryovers (into [TAX YEAR])
- Short-term: $[amount]
- Long-term: $[amount]
- Source: [prior year Schedule D, line 6 / line 14]

## Estimated Tax Payments ([TAX YEAR])
Enter these before calculating balance due — they directly reduce the amount owed.

| Date | Amount |
|------|--------|
| [Q1 date] | $[amount] |
| [Q2 date] | $[amount] |
| [Q3 date] | $[amount] |
| [Q4 date] | $[amount] |

Note: If no estimated payments were made, confirm explicitly so Claude does not ask repeatedly.

## K-1s ([TAX YEAR])
| Entity Name | EIN | Ordinary Income | Separately Stated Items | Notes |
|-------------|-----|-----------------|------------------------|-------|
| [Partnership / S-Corp / Trust name] | [EIN] | $[amount] | [e.g., §179, depletion, foreign tax] | [e.g., "awaiting K-1", "no prior carryover"] |

Note: K-1s from partnerships and S-corps flow to Schedule E page 2. Trust K-1s (Form 1041) go to Schedule B / other. Always note whether there are prior-year passive loss carryovers for each K-1 entity.

## Rental Properties — Schedule E
### [Address] — [ownership %]
- Mortgage interest: $[amount] | Taxes: $[amount] | Insurance: $[amount]
- Depreciation: [asset], in service [date], basis $[amount], [life]yr [method]
- Prior accumulated depreciation thru [year]: $[amount]
- Prior passive loss carryover: $[amount]

## Current Year Return — Final Numbers
| Return | Amount |
|--------|--------|
| **Federal Due/Refund** | **$[amount]** |
| **NY Refund/Due** | **$[amount]** |

Key figures: AGI $[amount] | Total Tax $[amount] | Federal Withheld $[amount] | Standard/Itemized Deduction $[amount]

## Workarounds & Known Issues
Document FreeTaxUSA-specific quirks here so they do not need to be re-discovered each session.

| Issue | Workaround | Verified |
|-------|-----------|---------|
| ADS 40yr depreciation (non-standard life) | Use category "Other" → type "Other (you select the depreciation life, method, and convention)" → Method: ADS, Recovery: 40yr, Convention: Mid-Month. Do NOT use "Residential rental property" (locks in 27.5yr). | [date] |
| "Rental Missing Depreciable Rental Property" alert for ADS properties | Harmless — appears because FreeTaxUSA does not recognize the "Other" category as rental. Continue filing. | [date] |
| NY IRC 125 addition not auto-populated from Box 14 | Must set `amount_472` separately in the NY State Miscellaneous Additions screen even if Box 14 IRC 125 is correctly entered. | [date] |
| [Add new issues here] | [workaround] | [date] |

## Corrections Log
Claude appends to this table after each session when a field value is corrected.

| Date | Field / Section | Old Value | New Value | Source |
|------|----------------|-----------|-----------|--------|
| [date] | [e.g., "Spouse Box 14 NY PFL"] | $[old] | $[new] | [e.g., "W-2 re-read YYYY-MM-DD"] |

## Folder Structure
\`\`\`
Taxes/
  [PRIOR YEAR] Tax Documents/
    [PRIOR YEAR]_FEDERAL_RETURN.pdf   ← prior year Form 1040 — required for YoY comparison and Form 2210
  [YEAR] Tax Documents/
    [Name] W-2.pdf
    [Name] 1099-INT.pdf
    Rental Expenses.xlsx
  CLAUDE.md
\`\`\`
```

---

## Tips

- **Add your prior year Form 1040 PDF to the tax folder** — this single document unlocks year-over-year comparison, Form 2210 safe harbor calculation, and carryover verification (passive losses from Form 8582, capital losses from Schedule D). Without it, Claude will ask for these figures manually.
- **Complete the setup checklist** at the top before starting a new year — missing items (K-1s, estimated payments) discovered late can require re-entering downstream figures
- **Keep prior year figures in CLAUDE.md** as a fallback — if the PDF isn't available, paste Line 11 (AGI) and Line 24 (Total Tax) at minimum
- **Document depreciation assets** in detail — basis, date in service, method, life, accumulated depreciation. The ADS workaround (see [SKILL.md](SKILL.md)) requires these to be correct
- **Record corrections** in the Corrections Log — when Claude finds and fixes errors, appending them here prevents re-checking the same fields in future sessions
- **Document K-1 timing** — partnerships often issue K-1s in March or later; note "awaiting K-1" and file an extension (Form 4868) if needed
- **Confirm estimated payments explicitly** — if none were made, say so; otherwise Claude will ask at the start of every session
- **Note FreeTaxUSA-specific workarounds** in the Workarounds section — e.g., "123 Main St uses ADS/40yr via 'Other' category; Rental Missing alert is harmless"
- **Update Filing Status History** each year — useful for spotting filing method changes and confirming extension dates
