# FreeTaxUSA Common Field IDs

> **All IDs documented here are for the 2025 tax year (filed in 2026).**
> Field IDs can change between tax years. At the start of each new tax year session, re-verify IDs before using them — especially for W-2 Box 14, state fields, and Form 2210.

---

## Field Discovery

Use these snippets at runtime when a hardcoded ID fails or you need to find an unknown field.

### Full input inventory
```javascript
// Dump all visible inputs with their id, name, and current value
Array.from(document.querySelectorAll('input')).map(i => ({
  id: i.id,
  name: i.name,
  type: i.type,
  value: i.value,
  placeholder: i.placeholder
}))
```

### Label-proximity search
When you know the label text but not the ID, use this to find the nearest input:
```javascript
// Find an input near a label that contains the given text (case-insensitive)
function findFieldByLabel(labelText) {
  const labels = Array.from(document.querySelectorAll('label'));
  const match = labels.find(l => l.textContent.toLowerCase().includes(labelText.toLowerCase()));
  if (!match) return 'No label found for: ' + labelText;
  // Try label[for] first
  if (match.htmlFor) return { via: 'label[for]', id: match.htmlFor, el: document.getElementById(match.htmlFor) };
  // Try nearest input sibling/child
  const input = match.closest('div,td,li')?.querySelector('input,select,textarea');
  return { via: 'proximity', el: input, id: input?.id };
}
findFieldByLabel('414(H)');
```

### React-controlled field setter
FreeTaxUSA uses React; plain `element.value = x` does not trigger state updates. Always use:
```javascript
function setReactField(id, value) {
  const el = document.getElementById(id);
  if (!el) return `Element not found: ${id}`;
  const nativeInputValueSetter = Object.getOwnPropertyDescriptor(window.HTMLInputElement.prototype, 'value').set;
  nativeInputValueSetter.call(el, value);
  el.dispatchEvent(new Event('input', { bubbles: true }));
  el.dispatchEvent(new Event('change', { bubbles: true }));
  return `Set ${id} = ${value}`;
}
```

### Field ID Verification
After calling `setReactField()`, confirm the value was accepted:
```javascript
// Verify a field value was accepted by React after setReactField()
function verifyField(id, expectedValue) {
  const el = document.getElementById(id);
  const actual = el?.value;
  return actual === String(expectedValue)
    ? `✓ verified ${id} = ${actual}`
    : `⚠ mismatch: expected ${expectedValue}, got ${actual}`;
}
// Example:
// setReactField('other_a2', '3469.44');
// verifyField('other_a2', '3469.44');
```

---

## W-2 Box 14 (2025 tax year)

| Field | ID |
|-------|----|
| NY PFL / FLI (Box 14 FLI field) | `other_a1` (first Box 14 entry) |
| 414(H) (Box 14 Other) | `other_a2` |
| IRC 125 (Box 14 Other) | `other_a3` |

Note: These IDs are per W-2. If entering a second W-2 in the same session, the suffix may increment (e.g., `other_b1`, `other_b2`). Verify with the full input inventory snippet above.

---

## SDI / FLI Screen (State Section)

| Field | ID |
|-------|----|
| FLI amount | `fli_amt0` |

---

## NY Miscellaneous Additions (State Section)

| Field | ID | Notes |
|-------|----|-------|
| IRC 125 NY addition | `amount_472` | Must be set separately even if Box 14 IRC 125 is correctly entered on the W-2 screen. FreeTaxUSA does not auto-populate this from Box 14. |

Navigation path: State Return → New York → Additions to Income → IRC Section 125

---

## Estimated Tax Payments

Navigation path: Federal → Payments → Estimated Tax Payments (Form 1040-ES)

| Field | Notes |
|-------|-------|
| Payment amount | Enter each quarterly payment separately. FreeTaxUSA adds a row per payment. |
| Payment date | Use the actual date paid (not the due date) for accurate Form 2210 installment calculation. |

Field IDs for estimated payments were not captured in 2025 sessions. Use the label-proximity snippet to locate them at runtime:
```javascript
findFieldByLabel('estimated tax');
findFieldByLabel('date paid');
```

Important: Enter estimated payments before reviewing the balance due summary. Payments entered after the summary screen may require navigating back to refresh the total.

---

## K-1 / Partnership Income

Navigation path: Federal → Income → Business Income (Partnerships, S-Corps, Trusts, Estates) → Schedule K-1

Key notes:
- Partnership K-1 (Form 1065): flows to Schedule E page 2
- S-Corp K-1 (Form 1120-S): flows to Schedule E page 2
- Trust/Estate K-1 (Form 1041): flows to Schedule B (interest/dividends) or other income sections depending on the box

When entering a K-1:
1. Select the correct entity type (Partnership / S-Corp / Trust)
2. Enter EIN, entity name, and your ownership share
3. Box 1 (ordinary income/loss) flows directly to Schedule E
4. Separately stated items (Box 2–20) require individual entry — FreeTaxUSA presents each box sequentially

Prior-year passive loss carryovers for each K-1 entity are tracked separately from rental carryovers. Pull from prior Form 8582 Part VII.

Field IDs for K-1 screens were not captured in 2025 sessions. Use the label-proximity snippet to locate fields at runtime.

---

## Schedule A (Itemized Deductions)

Navigation path: Federal → Deductions → Itemized Deductions

### 1098 Mortgage Interest Fields

| Field | Notes |
|-------|-------|
| Mortgage interest (Box 1) | Enter the exact Box 1 amount from Form 1098. |
| Points paid on purchase (Box 6) | Deductible in the year paid for a primary residence purchase. |
| Mortgage insurance premiums (Box 5) | Subject to income phaseout. |
| Property taxes (Box 10) | Flows to SALT deduction (capped at $10,000 combined with state/local income tax). |

Note: Mortgage interest on a rental property goes on Schedule E (not Schedule A). Only the primary residence and one second home qualify for Schedule A mortgage interest deduction.

Field IDs for Schedule A were not captured in 2025 sessions. Use the label-proximity snippet at runtime.

---

## Federal Direct Debit

| Field | ID |
|-------|----|
| Payment amount | `amount` |

---

## Form 2210 — Prior Year Info

| Field | ID |
|-------|----|
| Prior year AGI | `prior_agi` |
| Tax after credits (Line 22) | `prior_tax_after_credits` |
| Self-employment tax | `prior_se_tax` |
| Additional tax on distributions | `prior_early_withdrawal_tax_without_cont` |
| Household employment tax | `prior_household` |
| Homebuyer repayment | `prior_homebuyer_repayment` |
| Additional Medicare Tax | `prior_additional_medicare_tax` |
| Net Investment Income Tax | `prior_net_investment_income_tax` |
| Recapture of low-income housing credit | `prior_recapture_low_housing` |
| Other taxes | `prior_other_taxes` |
| Earned Income Credit | `prior_earned_income_credit` |
| Additional Child Tax Credit | `prior_additional_child_tax_credit` |
| Refundable American Opportunity Credit | `prior_refundable_oppor_credit` |
| Premium Tax Credit | `prior_ptc` |
| Federal fuels credit | `prior_fuels_credit` |
| Qualified sick/family leave credits | `prior_sick_credits` |
| Section 1341 credit | `prior_section_1341` |

### Notes on Form 2210 Entry

When prior year AGI > $150K, safe harbor = 110% × prior year total tax.

To avoid double-counting, split the prior year total tax:
- `prior_tax_after_credits` = total tax − Additional Medicare − NIIT − SE tax − other Schedule 2 items
- Enter each Schedule 2 item in its own field

Example: total tax $48,000, Additional Medicare $900, NIIT $350:
- `prior_tax_after_credits` = 48,000 − 900 − 350 = **46,750**
- `prior_additional_medicare_tax` = **900**
- `prior_net_investment_income_tax` = **350**

---

## CA State Returns

Navigation path: State Return → California

| Field | Notes |
|-------|-------|
| CA SDI (State Disability Insurance) | Reported in W-2 Box 14 or Box 19 depending on employer. In CA, SDI is deductible on Schedule A as a state income tax (subject to SALT cap) but NOT deductible on the CA state return itself. |
| CA SDI Box 14 label | Commonly labeled "CASDI", "CA SDI", or "SDI" on the W-2. FreeTaxUSA should auto-recognize "SDI" as California SDI. |

Note: CA SDI withheld appears on W-2 Box 14 for most CA employers. If the employer uses Box 19 instead, it may appear under "local taxes." Verify by checking the W-2 carefully.

Field IDs for CA state screens were not captured in 2025 sessions (no CA filers in the reference workspace). Use the label-proximity and full input inventory snippets at runtime.

---

## Annual Re-Verification Reminder

At the start of each new tax year session, run the following before using any hardcoded ID:

```javascript
// Quick spot-check: verify a known field still has the expected ID
// Run this on the W-2 entry screen to confirm Box 14 IDs have not shifted
Array.from(document.querySelectorAll('input[id^="other_a"]')).map(i => ({
  id: i.id,
  value: i.value,
  placeholder: i.placeholder
}))
```

If the IDs have changed, use the label-proximity snippet to rediscover them and update this file with the new IDs and tax year.
