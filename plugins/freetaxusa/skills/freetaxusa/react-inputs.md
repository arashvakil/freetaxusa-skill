# React SPA Form Automation Reference

FreeTaxUSA is built on React. React intercepts DOM events and maintains its own internal state tree. When you set `element.value = 'foo'` directly, React's state doesn't update — the field appears to change visually but the value is not saved when the form is submitted.

## The Fix: Native Input Value Setter

React patches `HTMLInputElement.prototype.value`. The original (unpatched) setter bypasses React's interception and properly triggers the synthetic event system:

```javascript
function setReactField(id, value) {
  const el = document.getElementById(id);
  if (!el) return `NOT FOUND: ${id}`;

  const nativeInputValueSetter = Object.getOwnPropertyDescriptor(
    window.HTMLInputElement.prototype,
    'value'
  ).set;

  nativeInputValueSetter.call(el, String(value));
  el.dispatchEvent(new Event('input', { bubbles: true }));
  el.dispatchEvent(new Event('change', { bubbles: true }));

  return `OK: ${id} = ${el.value}`;
}
```

## Setting multiple fields at once

```javascript
const setter = Object.getOwnPropertyDescriptor(
  window.HTMLInputElement.prototype, 'value'
).set;

function setField(id, val) {
  const el = document.getElementById(id);
  if (!el) return `NOT FOUND: ${id}`;
  setter.call(el, String(val));
  el.dispatchEvent(new Event('input', { bubbles: true }));
  el.dispatchEvent(new Event('change', { bubbles: true }));
  return `OK: ${id} = ${val}`;
}

// Set multiple fields
[
  setField('field_id_1', '12345'),
  setField('field_id_2', '67890'),
].join('\n')
```

## Finding field IDs

```javascript
// List all input fields and their current values
Array.from(document.querySelectorAll('input'))
  .filter(i => i.id)
  .map(i => ({ id: i.id, name: i.name, value: i.value, type: i.type }))
```

## Select/dropdown fields

For `<select>` elements (dropdowns), use a different approach:

```javascript
const nativeSelectSetter = Object.getOwnPropertyDescriptor(
  window.HTMLSelectElement.prototype,
  'value'
).set;

function setSelectField(id, value) {
  const el = document.getElementById(id);
  if (!el) return `NOT FOUND: ${id}`;
  nativeSelectSetter.call(el, value);
  el.dispatchEvent(new Event('change', { bubbles: true }));
  return `OK: ${id} = ${el.value}`;
}
```

## Why this works

React uses a synthetic event system and stores state in a fiber tree. When you use the native setter + dispatch real DOM events with `bubbles: true`, React's event delegation picks up the event at the document level, finds the corresponding React fiber node, and updates internal state — exactly as if the user had typed into the field.

## What doesn't work

- `element.value = 'foo'` — sets DOM value but not React state
- `element.setAttribute('value', 'foo')` — sets HTML attribute, not property
- `triple_click` + `type` via browser automation — sometimes works, often doesn't trigger React onChange for currency-formatted fields
- `form_input` MCP tool — inconsistent on React-controlled inputs

## Applies beyond FreeTaxUSA

This technique works on any React application: TurboTax, H&R Block, government portals (IRS Direct File, state tax sites), healthcare forms, banking applications, etc.
