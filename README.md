# freetaxusa — Claude Code Plugin

> Your accountant charges $800/hr and still got Box 14 wrong.
> This is free and runs in your terminal.

A Claude Code plugin for navigating FreeTaxUSA with browser automation. Read your actual source documents. Compare them field-by-field against what's in your return. Fix mistakes. Resolve alerts. File with confidence.

---

## What it does

- **Audits your return** — reads your W-2s, 1099s, and spreadsheets and compares them to what's entered in FreeTaxUSA, field by field
- **Fixes React form inputs** — FreeTaxUSA won't respond to standard automation; this plugin uses the React native setter to update fields correctly
- **Resolves Tax Return Alerts** — walks through each alert systematically: underpayment penalty (Form 2210), Box 14 mismatches, depreciation warnings
- **Handles ADS depreciation** — documents the "Other" category workaround for non-standard depreciation lives (40yr, etc.)
- **Plans ahead** — calculates estimated payments, extension amounts, and what-if scenarios without touching your return
- **Maintains context** — a `CLAUDE.md` workspace file carries your tax situation across sessions so you never re-explain from scratch

---

## Requirements

- [Claude Code](https://claude.ai/claude-code) CLI
- [claude-in-chrome](https://github.com/nicepkg/claude-in-chrome) MCP browser extension
- A FreeTaxUSA account with your return in progress

---

## Install

```bash
claude plugin marketplace add arashvakil/freetaxusa-skill
claude plugin install freetaxusa
```

---

## Usage

Open Claude Code from your tax documents folder:

```bash
cd ~/Documents/Taxes
claude
```

Then run:

```
/freetaxusa
```

---

## Commands

| Command | What it does |
|---------|-------------|
| `/freetaxusa` | Interactive — asks check vs. file if ambiguous |
| `/freetaxusa check` | Read-only audit — compares every document to FreeTaxUSA, no changes made |
| `/freetaxusa file` | Full run — verify, correct mismatches, resolve alerts, proceed toward filing |
| `/freetaxusa alerts` | Jump directly to Tax Return Alerts and resolve them |
| `/freetaxusa status` | Show what's verified, what's pending, and current balance due or refund |
| `/freetaxusa w2` | Verify W-2 entries only |
| `/freetaxusa schedule-e` | Verify Schedule E rental properties only |
| `/freetaxusa schedule-c` | Verify Schedule C business income only |
| `/freetaxusa investments` | Verify 1099-DIV, 1099-B, capital gains, and carryovers |
| `/freetaxusa k1` | Walk through K-1 partnership entries one by one |
| `/freetaxusa compare` | Generate year-over-year comparison from your CLAUDE.md |
| `/freetaxusa estimate` | Calculate Q1–Q4 estimated tax payments for next year |
| `/freetaxusa extension` | File Form 4868 and calculate what to pay with it |
| `/freetaxusa whatif` | Run what-if scenarios: 401k max, property sale, income change |

### Check vs. File

**`check` mode** — read-only. Reads your documents, reads your return, reports every mismatch. Nothing is written. Use this first.

**`file` mode** — corrects mismatches, resolves alerts, and walks you through to submission.

---

## Setup: workspace file

Copy `plugins/freetaxusa/skills/freetaxusa/workspace-template.md` to your tax folder as `CLAUDE.md` and fill in your details.

This gives Claude persistent context about your situation — properties, depreciation schedules, prior year figures, expected documents — without re-explaining each session. On first run with no `CLAUDE.md`, the skill will walk you through a setup questionnaire.

**Also add your prior year Form 1040 PDF to the same folder.** This unlocks:
- Year-over-year comparison (this year vs. last year, with plain-English explanation of what changed)
- Form 2210 underpayment penalty calculation (requires prior year AGI and total tax)
- Carryover verification — passive losses (Form 8582) and capital losses (Schedule D)

Without it the skill still runs, but these checks will be skipped or require manual input.

---

## Smart checks

Every `check` and `file` run automatically flags:

- **AMT** — taxable income above $220K (single) / $267K (MFJ)? Form 6251 review
- **NIIT** — AGI above $200K / $250K with investment income? 3.8% net investment income tax
- **QBI** — Schedule C or K-1 income? May qualify for the 20% Section 199A deduction
- **Withholding gap** — total withholding vs. total tax liability; flags if shortfall exceeds $1,000
- **Missing documents** — cross-references CLAUDE.md's expected document list against what was verified

---

## The React input trick

FreeTaxUSA is a React SPA. Standard automation (`element.value = x`, click-and-type) doesn't trigger React's state — fields appear to update but the value is never saved.

The fix:

```javascript
const setter = Object.getOwnPropertyDescriptor(
  window.HTMLInputElement.prototype, 'value'
).set;
setter.call(element, value);
element.dispatchEvent(new Event('input', { bubbles: true }));
element.dispatchEvent(new Event('change', { bubbles: true }));
```

This works on any React application — TurboTax, H&R Block, IRS Direct File, state tax portals. See [`react-inputs.md`](plugins/freetaxusa/skills/freetaxusa/react-inputs.md) for the full reference.

---

## Files

```
freetaxusa-skill/
├── .claude-plugin/
│   └── marketplace.json
├── plugins/
│   └── freetaxusa/
│       ├── .claude-plugin/
│       │   └── plugin.json
│       └── skills/
│           └── freetaxusa/
│               ├── SKILL.md                 ← main skill (all workflows, commands, warnings)
│               ├── react-inputs.md          ← React SPA form automation reference
│               ├── common-field-ids.md      ← FreeTaxUSA field IDs by section
│               └── workspace-template.md   ← CLAUDE.md starter template
└── README.md
```

---

## Known limitations

- Field IDs (e.g. `amount_472`) may change between FreeTaxUSA tax years — if something doesn't work, use the browser console to discover the current ID
- The ADS depreciation workaround triggers a harmless yellow alert ("Rental Missing Depreciable Rental Property") — this is expected and documented in the skill
- Browser automation requires claude-in-chrome to be running in Chrome

---

## Contributing

Found new field IDs, additional alert types, or FreeTaxUSA workflow quirks? PRs welcome. `common-field-ids.md` is especially worth keeping current across tax years.

---

## Disclaimer

Not tax or financial advice. For educational and informational purposes only. Consult a licensed tax professional before filing. I didn't — but I'm legally required to tell you that you should.

---

## License

MIT
