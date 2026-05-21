# Implementation Plan: Package Identifier Tool

**Source file**: `identifier-tool.html`

**Reverse-engineered**: 2026-05-21

**Status**: Draft

---

## Overview

A single self-contained HTML5 SPA for generating and validating five pharmaceutical package identifier formats. No external dependencies, no build step. Delivered as one `identifier-tool.html` file with embedded CSS and JavaScript.

---

## Supported Identifier Types

| Type | Full name | Structure |
|------|-----------|-----------|
| PZN  | Pharmazentralnummer | 8 digits |
| NTIN | National Trade Item Number | 13 digits (GS1 DE prefix + PZN + check) |
| GTIN | Global Trade Item Number | 8 / 12 / 13 / 14 digits |
| PPN  | Pharmacy Product Number | `9N` + 12 digits |
| PCID | Packaged Medicinal Product ID | UUID (v1–5) |

---

## Phase 1 — Core Algorithms

All logic is pure JavaScript with no DOM dependencies so it can be unit-tested in isolation.

### 1.1 PZN — Pharmazentralnummer

**Validation**

1. Reject if input is not exactly 8 digits.
2. Reject dummy values: `00000000`, `22222222`, `33333333`, `44444444`.
3. Compute check digit over the first 7 digits using weights `[1, 2, 3, 4, 5, 6, 7]`:
   ```
   sum = Σ digit[i] × weight[i]   (i = 0..6)
   cd  = sum % 11
   ```
4. Reject if `cd === 10` (slot not allocated by IFAH).
5. Valid if `cd === digit[7]`.

**Generation**

- Use rejection sampling: draw a random 7-digit base, compute check digit, discard if `cd === 10` or the resulting 8-digit string is a dummy value.
- For *invalid* mode: add 1 to the check digit modulo 10 before appending (guarantees wrong check digit without producing `cd === 10`).

---

### 1.2 GTIN — Global Trade Item Number

**Validation**

1. Strip non-digit characters.
2. Reject if length is not in `{8, 12, 13, 14}`.
3. Compute GS1 MOD-10 check digit over all digits except the last:
   ```
   For i = 0 to len-2 (right-to-left indexing from position 0):
     weight = 3 if i is even, else 1
     sum   += digit[len-2-i] × weight
   cd = (10 − (sum % 10)) % 10
   ```
4. Valid if `cd === last digit`.

**Generation**

- Generate a random 13-digit base, compute check digit, append to form GTIN-14.
- For *invalid* mode: add 1 to check digit modulo 10.

---

### 1.3 NTIN — National Trade Item Number

**Structure**

```
[4150 (4 digits)][PZN (8 digits)][MOD-10 check (1 digit)]  =  13 digits total
```

**Validation**

1. Strip non-digit characters; reject if length ≠ 13.
2. Apply GTIN MOD-10 validation (NTIN is a GTIN-13).

**Generation**

- Base = `'4150'` + either a freshly generated valid PZN (if *embed PZN* is checked) or a random 8-digit string.
- Compute MOD-10 check digit over the 12-digit base; append.
- For *invalid* mode: add 1 to check digit modulo 10.

**Embedded PZN extraction**

```
pzn = digits.slice(4, 12)
```
After extraction, validate the extracted value as a PZN.

---

### 1.4 PPN — Pharmacy Product Number

**Structure**

```
9N[IFA PRA code (2 digits)][PZN (8 digits)][MOD-97 check (2 digits)]  =  14 chars total
```

**Validation**

1. Strip leading `9N` if present.
2. Reject if the remaining string is not exactly 12 digits.
3. Compute MOD-97 check over the first 10 digits:
   ```
   multiplier = 1
   sum = 0
   for each char in digits[0..9]:
     multiplier++                   // starts at 2 for the first digit
     sum += charCodeAt(char) × multiplier
   cd = sum % 97
   ```
4. Valid if `cd === parseInt(digits[10..11])`.

**Generation**

- `base10 = '11'` (IFA PRA company code) + either a valid PZN or a random 8-digit string.
- Compute MOD-97 check; zero-pad to 2 digits; prepend `9N`.
- For *invalid* mode: add 1 to check digit modulo 97.

**Embedded PZN extraction**

```
// After stripping '9N':
pzn = digits.slice(2, 10)
```
After extraction, validate the extracted value as a PZN.

---

### 1.5 PCID — Packaged Medicinal Product ID

**Validation**

- Match against the UUID v1–5 regex:
  ```
  /^[0-9a-f]{8}-[0-9a-f]{4}-[1-5][0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$/i
  ```

**Generation**

- Use `crypto.randomUUID()` when available (modern browsers).
- Fallback: manual UUID v4 construction using `Math.random()`.
- *Invalid* generation is not supported — any structurally valid UUID is a valid PCID; the "generate invalid" option is disabled for this type.

---

## Phase 2 — UI Shell

Build the static HTML/CSS skeleton before wiring any JavaScript.

### 2.1 Layout

- Single-column, centred, max-width 660 px.
- Body background `#f0f2f5`; card background `#fff`; rounded corners (12 px).
- System font stack; no web fonts.

### 2.2 Tab switcher

- Two tabs: **Generate** and **Validate**.
- Active tab has a solid bottom border (`#2563eb`); inactive tabs have a transparent bottom border.
- Clicking a tab hides all panels and shows the matching panel via `display: none / block`.

### 2.3 Shared result banner

Two states: `success` (green) and `error` (red), toggled by adding the respective class. Displays:
- **result-value**: monospace, bold, word-break enabled.
- **result-label**: small descriptive text (`✓ / ✗ TYPE is structurally valid/invalid`).

---

## Phase 3 — Generate Panel

### 3.1 Identifier type selector

- `<select>` with options: PZN, NTIN, GTIN, PPN, PCID.
- `change` event triggers `syncGenOptions()` to show/hide conditional UI.

### 3.2 Options panel

A bordered checkbox group; shown beneath the type selector.

| Option | Shown for | Notes |
|--------|-----------|-------|
| Embed valid PZN | NTIN, PPN | Hidden for other types; checked by default |
| Generate invalid | All except PCID | Disabled + unchecked for PCID |

`syncGenOptions()` logic:
1. If type is `ntin` or `ppn`, show "Embed PZN" row; otherwise hide it.
2. If type is `pcid`, disable the "Generate invalid" checkbox and clear it; otherwise re-enable.

### 3.3 Generate button

On click:
1. Read type, wantInvalid, embedPzn from the form.
2. Dispatch to the matching generation function.
3. Populate the result banner with the generated value and a descriptive label.
4. Store the value in `dataset.copy` on the copy button.

### 3.4 Copy button

- Calls `navigator.clipboard.writeText()`.
- On success, briefly changes label to "Copied!" for 1.5 s then reverts to "Copy".
- No error handling needed — the button is only present when a value has been generated.

---

## Phase 4 — Validate Panel

### 4.1 Identifier type selector

Identical select control to the generate panel (independent state).

### 4.2 Text input

- Plain `<input type="text">`.
- `keydown` listener: trigger validation on **Enter**.

### 4.3 Validate button

On click (or Enter):
1. Guard: if input is empty, show error banner "Please enter an identifier value."
2. Dispatch to the matching validation function.
3. Show success or error banner with `✓/✗ TYPE is structurally valid/invalid`.
4. If type is `ntin` or `ppn`, append an embedded-PZN sub-result block.

### 4.4 Embedded PZN sub-result

A smaller bordered block appended inside the result banner:
- Extract the PZN using the type-specific extractor.
- Validate the extracted PZN.
- Show `success` (green), `error` (red), or `warn` (amber if extraction failed) state.
- Display `Embedded PZN: <value> — ✓/✗ valid/invalid`.

---

## Phase 5 — Integration & Edge Cases

| Concern | Handling |
|---------|----------|
| Empty generate result container | Hide `.result` div by default (`display: none`); add `.show` class only after generation |
| PCID "invalid" state | Disable checkbox, prevent invalid generation path entirely |
| NTIN/PPN embed-PZN toggle | `row-embed-pzn` row hidden by default; shown only when type is `ntin` or `ppn` |
| Clipboard API unavailable | `navigator.clipboard?.writeText(v).catch(() => {})` — silent fail (acceptable given modern browser targets) |
| Non-digit characters in GTIN/NTIN | Strip with `.replace(/\D/g, '')` before length/check-digit tests |

---

## Delivery Checklist

- [ ] All five `validate*` functions pass edge-case inputs (correct, wrong check digit, wrong length, dummy PZN)
- [ ] All five `generate*` functions produce values that pass their own `validate*` function
- [ ] `generatePzn(false)` never returns a dummy or cd-10 value in 10 000 trials
- [ ] "Generate invalid" produces a value that fails validation for PZN, NTIN, GTIN, PPN
- [ ] PCID "Generate invalid" checkbox is disabled and cannot be enabled
- [ ] "Embed PZN" row is hidden for PZN, GTIN, PCID types
- [ ] Validate panel shows embedded PZN sub-result for NTIN and PPN only
- [ ] Copy button label resets after 1.5 s
- [ ] File opens correctly as a local `file://` URL with no network requests
