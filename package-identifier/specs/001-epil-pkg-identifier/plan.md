# Implementation Plan: Package Identifier Tool

**Feature**: `001-epil-pkg-identifier`
**Spec**: [spec.md](spec.md)
**Created**: 2026-05-21
**Status**: Draft

---

## Overview

A single self-contained `index.html` SPA that generates and validates five identifier types entirely in the browser: PZN, NTIN, GTIN, PPN, and PCID. The UI provides exactly two top-level modes, Generate and Validate, plus additional embedded-PZN inspection for NTIN and PPN.

**Requirements covered**: FR-001 – FR-052
**User stories**: US-1 (Validate by type), US-2 (Generate by type), US-3 (Inspect embedded PZN)

---

## Dependency order

```
Phase 1 — Algorithm library
    ↓
Phase 2 — HTML shell and controls
    ↓
Phase 3 — Styling and result states
    ↓
Phase 4 — Generate flow
    ↓
Phase 5 — Validate flow
    ↓
Phase 6 — Embedded PZN inspection
    ↓
Phase 7 — Clipboard and interaction polish
```

---

## Phase 1 — Algorithm Library

All functions in this phase are pure JavaScript with no DOM dependency. They should be directly callable in the browser console.

### T-1.1 · PZN algorithm pair
*Satisfies: FR-021, FR-022, FR-023, FR-024, FR-025*

| Sub-task | Detail |
|----------|--------|
| `_pznRawCheck(base7)` | Multiply the first 7 digits by weights `1..7`, sum, and return `sum % 11`. |
| `validatePzn(value)` | Require exactly 8 digits, reject the dummy set, reject computed check digit `10`, and compare the final digit to the computed result. |
| `generatePzn(wantInvalid)` | Create a random 7-digit base, compute the check digit, and optionally corrupt it by shifting `+1 mod 10`. |

**Done when**: Valid issued-style PZNs pass; the dummy values fail; invalid generation produces a value that fails `validatePzn`.

---

### T-1.2 · GTIN algorithm pair
*Satisfies: FR-026, FR-027, FR-028, FR-029, FR-030*

| Sub-task | Detail |
|----------|--------|
| `_gtinCheck(base)` | Apply GS1 MOD-10 with alternating weights `3, 1` from right to left across the body digits. |
| `validateGtin(value)` | Strip non-digits, require length in `{8, 12, 13, 14}`, compute the expected check digit, and compare against the final digit. |
| `generateGtin(wantInvalid)` | Create a 13-digit random base, compute the check digit, and return a 14-digit GTIN; optionally corrupt the digit by shifting `+1 mod 10`. |

**Done when**: `validateGtin('03400926006476')` is true; changing the last digit makes it false; generated invalid output fails validation.

---

### T-1.3 · NTIN algorithm pair
*Satisfies: FR-031, FR-032, FR-033, FR-034, FR-035, FR-036*

| Sub-task | Detail |
|----------|--------|
| `validateNtin(value)` | Strip non-digits, require exactly 13 digits, then delegate to `validateGtin`. |
| `generateNtin(wantInvalid, embedPzn)` | Build `4150 + inner + check`, where `inner` is either a valid generated PZN or a random 8-digit number. |
| `extractPznFromNtin(value)` | Strip non-digits and return digits `slice(4, 12)` when the value is 13 digits; otherwise return `null`. |

**Done when**: Generated valid NTINs pass validation; generated invalid NTINs fail; extracted embedded PZN matches the inner 8-digit segment.

---

### T-1.4 · PPN algorithm pair
*Satisfies: FR-037, FR-038, FR-039, FR-040, FR-041, FR-042, FR-043*

| Sub-task | Detail |
|----------|--------|
| `_ppnCheck(base10)` | Compute MOD-97 over the first 10 digits using character codes and multipliers `2..11`. |
| `validatePpn(value)` | Remove an optional `9N` prefix, require exactly 12 digits, and compare the computed 2-digit checksum with the trailing digits. |
| `generatePpn(wantInvalid, embedPzn)` | Build `9N + 11 + inner + checksum`, where `inner` is either a valid generated PZN or a random 8-digit number. |
| `extractPznFromPpn(value)` | Remove an optional `9N` prefix, then return digits `slice(2, 10)` when the stripped value is 12 digits; otherwise return `null`. |

**Done when**: Generated valid PPNs pass validation; generated invalid PPNs fail; embedded PZN extraction yields the 8-digit inner payload.

---

### T-1.5 · PCID algorithm pair
*Satisfies: FR-044, FR-045, FR-046*

| Sub-task | Detail |
|----------|--------|
| `generatePcid()` | Prefer `crypto.randomUUID()` and fall back to an internal UUID-v4-style template when unavailable. |
| `validatePcid(value)` | Validate against the RFC 4122 versions `1..5` regex. |

**Done when**: Generated PCIDs match validation; there is no path that produces a deliberately invalid PCID.

---

## Phase 2 — HTML Shell And Controls

Write the full markup in a single document with inline CSS and JavaScript only.

### T-2.1 · Document shell
*Satisfies: FR-001, FR-047, FR-048, FR-049*

- `<!DOCTYPE html>`, `lang="en"`, `charset="UTF-8"`, and a mobile viewport meta tag.
- Page title `Package Identifier Tool`.
- One `.container` wrapping the header, tab controls, and both panels.

### T-2.2 · Tab navigation
*Satisfies: FR-001, FR-052*

- Two tab buttons labelled `Generate` and `Validate`.
- Two corresponding panels, with Generate active on initial load.

### T-2.3 · Generate panel structure
*Satisfies: FR-002, FR-003, FR-014, FR-015, FR-016, FR-017*

Inside the Generate panel, provide:
- A type selector with options for `PZN`, `NTIN`, `GTIN`, `PPN`, and `PCID`.
- An embed-PZN checkbox row that is present in the markup and shown only for `NTIN` and `PPN`.
- An invalid-generation checkbox row that is disabled and cleared when `PCID` is selected.
- A Generate button.
- A result block containing generated value, result label, and Copy button.

### T-2.4 · Validate panel structure
*Satisfies: FR-002, FR-004, FR-008, FR-009, FR-018, FR-019, FR-020*

Inside the Validate panel, provide:
- A type selector with the same five options.
- A freeform text input for the identifier value.
- A Validate button.
- A result block containing the validated value, result label, and an embedded-PZN container for NTIN/PPN sub-results.

---

## Phase 3 — Styling And Result States

Apply only the visual rules required by the current implementation. Do not invent extra responsive or accessibility behavior not present in the code.

### T-3.1 · Base layout
*Satisfies: FR-052*

- System font stack.
- Neutral light background.
- One centered container with `max-width` around 660 px.
- White card surfaces with subtle shadow and rounded corners.

### T-3.2 · Interactive controls
*Satisfies: FR-052*

- Styled tab buttons with active underline state.
- Full-width selects, text inputs, and primary buttons.
- Checkbox rows for optional generator behavior.

### T-3.3 · Result banners
*Satisfies: FR-006, FR-007, FR-052*

- Hidden by default.
- `.success` state for valid/generated-valid outcomes.
- `.error` state for invalid/generated-invalid outcomes.
- Sub-result variants for embedded PZN: success, error, and warn.

### T-3.4 · Copy button feedback styling
*Satisfies: FR-010, FR-013*

- Inline secondary button styling for Copy.
- No separate toast mechanism is required by the current implementation.

---

## Phase 4 — Generate Flow

Wire the Generate panel to the selected type and options.

### T-4.1 · Option synchronization
*Satisfies: FR-014, FR-015, FR-016, FR-017*

Implement `syncGenOptions()`:
1. Read the selected generator type.
2. Show the embed-PZN row only for `NTIN` and `PPN`.
3. Disable and clear the invalid-generation checkbox for `PCID`.
4. Re-enable invalid generation for all other types.

### T-4.2 · Type-dispatched generation
*Satisfies: FR-002, FR-003, FR-014, FR-046*

On Generate button click:
1. Read the selected type.
2. Read `wantInvalid` from the checkbox unless that checkbox is disabled.
3. Read `embedPzn` from the checkbox.
4. Dispatch to `generatePzn`, `generateNtin`, `generateGtin`, `generatePpn`, or `generatePcid`.
5. Catch unexpected errors and render an error result.

### T-4.3 · Generate result labeling
*Satisfies: FR-006, FR-014*

- Valid generation label: `Generated valid <TYPE>`.
- Invalid generation label: `Generated invalid <TYPE> — check digit deliberately corrupted`.
- Success/error banner class is driven by whether the user requested invalid generation.

### T-4.4 · Copy target storage
*Satisfies: FR-010, FR-011*

After successful generation, store the generated value in `#gen-copy.dataset.copy` for later copy invocation.

---

## Phase 5 — Validate Flow

Wire validation to the selected type and render a single structural-validity result.

### T-5.1 · Empty input guard
*Satisfies: FR-008*

If the Validate input is empty after trimming:
- Show an error result.
- Set the displayed value to an em dash placeholder.
- Use the exact message `Please enter an identifier value.`
- Do not invoke any validator.

### T-5.2 · Type-dispatched validation
*Satisfies: FR-002, FR-004, FR-005*

Implement `runValidation()` to:
1. Read the selected type.
2. Dispatch to `validatePzn`, `validateNtin`, `validateGtin`, `validatePpn`, or `validatePcid`.
3. Use the selected type label, not auto-detection, to build the result text.

### T-5.3 · Result labeling
*Satisfies: FR-007*

- Valid message: `✓ <TYPE> is structurally valid`.
- Invalid message: `✗ <TYPE> is structurally invalid`.
- The result area always shows the submitted input value when validation runs.

### T-5.4 · Enter key shortcut
*Satisfies: FR-009*

Attach a `keydown` listener to the Validate input so pressing Enter triggers `runValidation()`.

---

## Phase 6 — Embedded PZN Inspection

This phase covers the NTIN/PPN-specific secondary result block.

### T-6.1 · NTIN and PPN extraction dispatch
*Satisfies: FR-018, FR-019, FR-020*

After validation:
- If the selected type is `NTIN`, call `extractPznFromNtin(value)`.
- If the selected type is `PPN`, call `extractPznFromPpn(value)`.
- For all other types, render no embedded-PZN block.

### T-6.2 · Embedded PZN sub-result rendering
*Satisfies: FR-018, FR-019, FR-020*

Implement `buildEmbeddedPznBlock(pzn)`:
- If extraction returns `null`, render warn text `Embedded PZN: could not be extracted`.
- If a PZN is extracted, validate it with `validatePzn(pzn)`.
- Render a success or error sub-block stating whether the embedded PZN is valid.

---

## Phase 7 — Clipboard And Interaction Polish

Keep clipboard behavior aligned with the current code rather than the older GTIN-only plan.

### T-7.1 · Tab switching
*Satisfies: FR-001, FR-052*

One click handler per tab button should:
1. Remove `.active` from all tab buttons and panels.
2. Activate the clicked tab.
3. Activate the matching panel using its `data-tab` value.

### T-7.2 · Copy behavior
*Satisfies: FR-010, FR-011, FR-012, FR-013*

Wire the Generate copy button to:
- Read `dataset.copy`.
- Call `navigator.clipboard?.writeText(value)` when a value is present.
- Silently ignore promise rejection.
- Change the button label to `Copied!` and revert it to `Copy` after 1.5 seconds.

---

## Delivery Checklist

Verify each item manually before marking the feature complete.

**Algorithms**
- [x] `validatePzn` rejects dummy PZNs and unallocated check digit `10`
- [x] `generatePzn(true)` produces a value that fails `validatePzn`
- [x] `validateGtin` accepts valid GTIN-8, GTIN-12, GTIN-13, and GTIN-14 lengths after stripping non-digits
- [x] `generateGtin(true)` produces a value that fails `validateGtin`
- [x] `validateNtin` enforces 13 digits and delegates to GTIN validation
- [x] `extractPznFromNtin` returns the inner 8-digit segment for a well-formed NTIN
- [x] `validatePpn` accepts optional `9N` prefix and checks the trailing 2-digit checksum
- [x] `extractPznFromPpn` returns the inner 8-digit segment after optional prefix removal
- [x] `generatePcid()` returns a value that passes `validatePcid`

**Generate flow**
- [x] The type selector offers exactly PZN, NTIN, GTIN, PPN, and PCID
- [x] The embed-PZN option is shown only for NTIN and PPN
- [x] The invalid-generation option is disabled and unchecked for PCID
- [x] Generating with invalid mode on PZN, NTIN, GTIN, or PPN yields an error-styled result with the deliberate-corruption label
- [x] Generated values are stored for the Copy button

**Validate flow**
- [x] Empty input shows `Please enter an identifier value.`
- [x] Enter key triggers validation from the input field
- [x] Valid values show `✓ <TYPE> is structurally valid`
- [x] Invalid values show `✗ <TYPE> is structurally invalid`
- [x] Validation uses the user-selected type rather than type auto-detection

**Embedded PZN**
- [x] NTIN validation renders an embedded-PZN sub-result
- [x] PPN validation renders an embedded-PZN sub-result
- [x] Extraction failure renders `Embedded PZN: could not be extracted`
- [x] Extracted embedded PZNs are separately marked valid or invalid

**Clipboard and non-functional**
- [x] Copy attempts use `navigator.clipboard.writeText()` when available
- [x] Clipboard write failures are silently ignored
- [x] Pressing Copy changes the label to `Copied!` and later restores `Copy`
- [x] The app is a single self-contained `index.html` file with inline CSS and JavaScript
- [x] The app runs without authentication, external assets, or persistent storage requirements


