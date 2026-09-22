# Implementation Plan: Package Identifier Tool

**Feature**: `001-epil-pkg-identifier`
**Spec**: [spec.md](spec.md)
**Created**: 2026-05-21
**Status**: Draft
**Design docs**: [research.md](research.md) · [data-model.md](data-model.md) · [contracts/ppn-generation.md](contracts/ppn-generation.md) · [contracts/custom-pzn-embedding.md](contracts/custom-pzn-embedding.md) · [contracts/url-anchor-deep-linking.md](contracts/url-anchor-deep-linking.md) · [quickstart.md](quickstart.md)

---

## Constitution Check (increment: PPN invalid × 9N-prefix options, FR-039/FR-040)

| Article | Check | Result |
|---------|-------|--------|
| I. Single-File Delivery | New checkbox reuses existing `.checkbox-row` markup/CSS inline in `index.html`; no new files. | Pass |
| II. Zero External Dependencies | No new APIs or libraries; pure boolean state read from a checkbox. | Pass |
| III. Algorithm Fidelity | Checksum computation (`_ppnCheck`/`_decimalModulo`) is untouched; `includePrefix` only affects string concatenation after the checksum is computed. | Pass |
| IV. Standards-First Identifiers | No new identifier type introduced; existing IFA PPN / ISO 7064 MOD 97-10 citation stands. | Pass |
| V. Mobile-Accessible UI | New checkbox row follows the same pattern (and CSS classes) as `row-embed-pzn`/`row-invalid`, so it inherits the 44×44px tap target and contrast rules. | Pass |

No violations; no complexity tracking entries required.

---

## Constitution Check (increment: PPN EPL v2.3.x Compatibility Mode, FR-041–FR-045)

| Article | Check | Result |
|---------|-------|--------|
| I. Single-File Delivery | New checkbox + tooltip help text reuse existing `.checkbox-row` markup/CSS inline in `index.html`; no new files. | Pass |
| II. Zero External Dependencies | Tooltip uses the native `title` attribute plus an always-visible `<small>` hint — no JS popover library, no external assets. | Pass |
| III. Algorithm Fidelity | The new legacy checksum helper is added *alongside*, not instead of, `_ppnCheck`/`_decimalModulo`; it must cite its source (`PpnValidationService.php::calculateCheckDigit`) in a code comment, matching Article III's reference-citation requirement — fidelity is to the actual current backend behavior, which is the cited reference implementation. | Pass, with note (see below) |
| IV. Standards-First Identifiers | The standard checksum (ISO/IEC 7064 MOD 97-10) remains the default (`eplCompat` defaults `false`); EPL v2.3.x compatibility mode is an explicit, opt-in accommodation for a known backend defect, not a replacement standard. | Pass |
| V. Mobile-Accessible UI | The explanatory text is rendered as an always-visible `<small>` hint (not a hover-only tooltip), so it remains usable on touch viewports without relying on `:hover`; the `title` attribute is additive for desktop pointer users. | Pass |

**Note on Article III**: this increment intentionally implements a second, non-standard checksum formula. This is not a deviation from *this app's* reference implementation for PPN (IFA MOD-97 / ISO 7064, still the default); it is a deliberate, clearly labeled, opt-in accommodation of a defect in the EPL backend's `PpnValidationService.php`, tracked separately for a backend fix. No complexity tracking entry is required because both formulas are fully implemented (not approximated) and the standard formula remains the default and sole basis for `9N`-prefix-only generation.

---

## Constitution Check (increment: NTIN/PPN Custom PZN Embedding, FR-046–FR-049)

| Article | Check | Result |
|---------|-------|--------|
| I. Single-File Delivery | New free-text field reuses the existing `.field`/`label.field-label`/`input[type="text"]` markup and CSS already used by the Validate panel's `#val-input`; no new files. | Pass |
| II. Zero External Dependencies | Plain `<input type="text">` read via the DOM; no new APIs or libraries. | Pass |
| III. Algorithm Fidelity | `_pznRawCheck`/`validatePzn` (the existing PZN reference implementation) is reused unchanged to validate the manually entered value before it is trusted as `inner`; no new checksum logic is introduced. | Pass |
| IV. Standards-First Identifiers | No new identifier type; the field only changes the *source* of the 8-digit inner PZN segment already required by the NTIN/PPN structures. | Pass |
| V. Mobile-Accessible UI | The field follows the same `.field` pattern (full-width, 44×44px-equivalent tap/focus target) already used elsewhere in the Generate and Validate panels. | Pass |

No violations; no complexity tracking entries required.

---

## Constitution Check (increment: PPN EPL v2.3.x Compatibility Caveat Note, FR-042a)

| Article | Check | Result |
|---------|-------|--------|
| I. Single-File Delivery | The caveat note reuses the existing result-block markup/CSS already inline in `index.html`; no new files. | Pass |
| II. Zero External Dependencies | Plain conditionally-rendered text node; no new APIs or libraries. | Pass |
| III. Algorithm Fidelity | Purely presentational — no checksum or validation logic changes. | Pass |
| IV. Standards-First Identifiers | Reinforces (rather than weakens) the standards-first posture by making the non-standard, opt-in nature of EPL v2.3.x compatibility mode visible at the point of use, not just in a tooltip. | Pass |
| V. Mobile-Accessible UI | The note is an always-visible text line (not hover-only), so it is reachable on touch viewports like the existing `<small>` hint. | Pass |

No violations; no complexity tracking entries required.

---

## Constitution Check (increment: URL Anchor Deep Linking, FR-050–FR-053)

| Article | Check | Result |
|---------|-------|--------|
| I. Delivery | Parsing uses the native `URLSearchParams` API against `location.hash`; no new files, no build step. | Pass |
| II. Data & Privacy | The URL fragment is read-only, ephemeral browser state (never sent to a server, never written to storage); this is not persistence — it disappears on manual URL edit/clear like any other navigation state. | Pass |
| III. Supported Identifier Types | No new identifier type; `package-identifier` values are validated against the existing five-type set before being applied. | Pass |
| IV. Generator–Validator Duality | Not applicable — this increment only affects which tab/type is pre-selected, not any generate/validate algorithm. | Pass |
| V. Algorithm Specifications | Not applicable — no checksum or algorithm changes. | Pass |
| VI. Embedded PZN | Not applicable — unaffected. | Pass |
| VII. User Interface | Still exactly two top-level modes (Generate/Validate); the anchor only pre-activates one of the two existing tabs via the existing tab-switch mechanism, and only pre-selects among the five existing identifier types — no new mode or type is introduced. Unrecognized values fall back silently to current defaults, consistent with §25's "explicit visible outcome" rule applying only to generate/validate operations, not passive page-load state. | Pass |
| VIII. Clipboard | Not applicable — unaffected. | Pass |

No violations; no complexity tracking entries required.

---

## Overview

A single self-contained `index.html` SPA that generates and validates five identifier types entirely in the browser: PZN, NTIN, GTIN, PPN, and PCID. The UI provides exactly two top-level modes, Generate and Validate, plus additional embedded-PZN inspection for NTIN and PPN.

**Requirements covered**: FR-001 – FR-049, FR-042a, FR-050 – FR-053
**User stories**: US-1 (Validate by type), US-2 (Generate by type), US-3 (Inspect embedded PZN), US-4 (Deep link via URL anchor)

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
*Satisfies: FR-021, FR-022, FR-033*

| Sub-task | Detail |
|----------|--------|
| `_pznRawCheck(base7)` | Multiply the first 7 digits by weights `1..7`, sum, and return `sum % 11`. |
| `validatePzn(value)` | Require exactly 8 digits, reject the dummy set, reject computed check digit `10`, and compare the final digit to the computed result. |
| `generatePzn(wantInvalid)` | Create a random 7-digit base, compute the check digit, and optionally corrupt it by shifting `+1 mod 10`. |

**Done when**: Valid issued-style PZNs pass; the dummy values fail; invalid generation produces a value that fails `validatePzn`.

---

### T-1.2 · GTIN algorithm pair
*Satisfies: FR-023, FR-024, FR-025, FR-033*

| Sub-task | Detail |
|----------|--------|
| `_gtinCheck(base)` | Apply GS1 MOD-10 with alternating weights `3, 1` from right to left across the body digits. |
| `validateGtin(value)` | Strip non-digits, require length in `{8, 12, 13, 14}`, compute the expected check digit, and compare against the final digit. |
| `generateGtin(wantInvalid)` | Create a 13-digit random base, compute the check digit, and return a 14-digit GTIN; optionally corrupt the digit by shifting `+1 mod 10`. |

**Done when**: `validateGtin('03400926006476')` is true; changing the last digit makes it false; generated invalid output fails validation.

---

### T-1.3 · NTIN algorithm pair
*Satisfies: FR-026, FR-027, FR-033, FR-046, FR-048, FR-049*

| Sub-task | Detail |
|----------|--------|
| `validateNtin(value)` | Strip non-digits, require exactly 13 digits, then delegate to `validateGtin`. |
| `generateNtin(wantInvalid, embedPzn, customPzn = null)` | Build `4150 + inner + check`, where `inner` is `customPzn` verbatim when provided (already validated by the UI layer per FR-047), otherwise either a valid generated PZN or a random 8-digit number per `embedPzn` exactly as before. |
| `extractPznFromNtin(value)` | Strip non-digits and return digits `slice(4, 12)` when the value is 13 digits; otherwise return `null`. |

**Done when**: Generated valid NTINs pass validation; generated invalid NTINs fail; extracted embedded PZN matches the inner 8-digit segment; when `customPzn` is supplied, the extracted embedded PZN equals it exactly.

---

### T-1.4 · PPN algorithm pair
*Satisfies: FR-028, FR-029, FR-033, FR-039, FR-040, FR-041, FR-043, FR-044, FR-045*

Implements the IFA PPN specification: ISO/IEC 7064 MOD 97-10 over `"11" + 8-digit PZN`. Also
implements a second, non-standard checksum formula solely for EPL v2.3.x compatibility mode (see
Constitution Check above and [research.md](research.md)).

| Sub-task | Detail |
|----------|--------|
| `_decimalModulo(decimalString, divisor)` | Fold a decimal digit string into a modulo result one digit at a time, avoiding integer-size limits. |
| `_ppnCheck(body10)` | `remainder = _decimalModulo(body10 + "00", 97)`; `check = 98 - remainder`, zero-padded to 2 digits. Standard ISO/IEC 7064 MOD 97-10 formula; used whenever `eplCompat` is `false` (the default). |
| `_ppnCheckEplLegacy(body10)` | Mirrors the EPL backend's current (non-standard) formula: sum each character's `charCodeAt(0)` (not its numeric digit value) weighted `2..11` in position order, then take `% 97`. Cite the source as a code comment, e.g. `// Source: PpnValidationService.php::calculateCheckDigit (legacy, pre-fix)`. Zero-pad the result to 2 digits. |
| `validatePpn(value)` | Remove an optional `9N` data-identifier prefix, require exactly 12 digits, and confirm the trailing 2-digit checksum satisfies *either* `_ppnCheck` (`_decimalModulo(value, 97) === 1`) *or* `_ppnCheckEplLegacy` on the first 10 digits — accepting either recognized checksum formula. |
| `generatePpn(wantInvalid, embedPzn, includePrefix, eplCompat, customPzn = null)` | Build `11 + inner + checksum`, where `inner` is `customPzn` verbatim when provided (already validated by the UI layer per FR-047), otherwise either a valid generated PZN (leading zeroes retained) or a random 8-digit number per `embedPzn`, exactly as before; compute the checksum via `_ppnCheckEplLegacy` when `eplCompat` is `true`, otherwise via `_ppnCheck`; corrupt by shifting the checksum `+1 mod 97` when `wantInvalid`; prepend `9N` only when `includePrefix` is true. Loop (regenerate/recorrupt) until `validatePpn(ppn) === !wantInvalid`, so an invalid result is guaranteed to fail *both* checksum formulas, not just the one that was corrupted. Default `includePrefix` to `true`, `eplCompat` to `false`, and `customPzn` to `null` for any caller that omits them. |
| `extractPznFromPpn(value)` | Remove an optional `9N` prefix, then return digits `slice(2, 10)` when the stripped value is 12 digits; otherwise return `null`. Unaffected by `eplCompat`. |

**Done when**: `generatePpn` for PZN `12345678` yields body `1112345678`, checksum `35`, PPN `111234567835` / `9N111234567835` depending on `includePrefix` (standard mode); all four combinations of `wantInvalid` × `includePrefix` validate consistently via `validatePpn`; with `eplCompat` true, the checksum instead matches `_ppnCheckEplLegacy`'s output and the resulting value still validates via `validatePpn` (which accepts either formula); with `eplCompat` true and `wantInvalid` true, the value fails `validatePpn` (i.e. fails both formulas); embedded PZN extraction yields the 8-digit inner payload in every combination.

---

### T-1.5 · PCID algorithm pair
*Satisfies: FR-030, FR-031, FR-032*

| Sub-task | Detail |
|----------|--------|
| `generatePcid()` | Prefer `crypto.randomUUID()` and fall back to an internal UUID-v4-style template when unavailable. |
| `validatePcid(value)` | Validate against the RFC 4122 versions `1..5` regex. |

**Done when**: Generated PCIDs match validation; there is no path that produces a deliberately invalid PCID.

---

## Phase 2 — HTML Shell And Controls

Write the full markup in a single document with inline CSS and JavaScript only.

### T-2.1 · Document shell
*Satisfies: FR-001, FR-038*

- `<!DOCTYPE html>`, `lang="en"`, `charset="UTF-8"`, and a mobile viewport meta tag.
- Page title `Package Identifier Tool`.
- One `.container` wrapping the header, tab controls, and both panels.

### T-2.2 · Tab navigation
*Satisfies: FR-001, FR-038*

- Two tab buttons labelled `Generate` and `Validate`.
- Two corresponding panels, with Generate active on initial load.

### T-2.3 · Generate panel structure
*Satisfies: FR-002, FR-003, FR-014, FR-015, FR-016, FR-017, FR-039, FR-041, FR-042, FR-042a, FR-044*

Inside the Generate panel, provide:
- A type selector with options for `PZN`, `NTIN`, `GTIN`, `PPN`, and `PCID`.
- An embed-PZN checkbox row that is present in the markup and shown only for `NTIN` and `PPN`.
- An invalid-generation checkbox row that is disabled and cleared when `PCID` is selected.
- A `9N` prefix checkbox row (`row-ppn-prefix` / `gen-ppn-prefix`), checked by default, present in the markup and shown only for `PPN`. This option is independent of the invalid-generation checkbox.
- An **EPL v2.3.x compatibility mode** checkbox row (`row-epl-compat` / `gen-epl-compat`), unchecked by default, present in the markup and shown only for `PPN`. This option is independent of the invalid-generation, `9N`-prefix, and embed-PZN checkboxes. Include a help affordance (e.g. a small info marker with a `title` attribute) plus an always-visible `<small>` hint line explaining: "Generates PPNs using the checksum the EPL backend currently expects, instead of the standard ISO/IEC 7064 formula."
- A Generate button.
- A result block containing generated value, result label, Copy button, and (for `PPN` generated under EPL v2.3.x compatibility mode) a distinct caveat note element, separate from the success message, stating the value is valid only under EPL v2.3.x compatibility mode and uses a non-standard checksum.

### T-2.4 · Validate panel structure
*Satisfies: FR-002, FR-004, FR-008, FR-009, FR-018, FR-019, FR-020*

Inside the Validate panel, provide:
- A type selector with the same five options.
- A freeform text input for the identifier value.
- A Validate button.
- A result block containing the validated value, result label, and an embedded-PZN container for NTIN/PPN sub-results.

### T-2.5 · URL anchor parsing (deep linking)
*Satisfies: FR-050, FR-051, FR-052, FR-053*

Implement `applyAnchorState()` per [contracts/url-anchor-deep-linking.md](contracts/url-anchor-deep-linking.md):
1. Read `location.hash` (strip the leading `#`) and parse it with `new URLSearchParams(...)`.
2. Read the `mode` key; if its value case-insensitively equals `generate` or `validate`, activate that tab via the existing tab-switch logic (T-7.1); otherwise leave the current tab as-is.
3. Read the `package-identifier` key; if its value case-insensitively matches `pzn`, `ntin`, `gtin`, `ppn`, or `pcid`, set both `#gen-type` and `#val-type` to that value and call `syncGenOptions()` so option-row visibility stays consistent; otherwise leave both selects unchanged.
4. Ignore any other keys/values without error.

Call `applyAnchorState()` once on initial script execution, and again on every `window.addEventListener('hashchange', applyAnchorState)` event, per FR-053.

---

## Phase 3 — Styling And Result States

Apply only the visual rules required by the current implementation. Do not invent extra responsive or accessibility behavior not present in the code.

### T-3.1 · Base layout
*Satisfies: FR-038*

- System font stack.
- Neutral light background.
- One centered container with `max-width` around 660 px.
- White card surfaces with subtle shadow and rounded corners.

### T-3.2 · Interactive controls
*Satisfies: FR-038*

- Styled tab buttons with active underline state.
- Full-width selects, text inputs, and primary buttons.
- Checkbox rows for optional generator behavior.

### T-3.3 · Result banners
*Satisfies: FR-006, FR-007, FR-037, FR-042a*

- Hidden by default.
- `.success` state for valid/generated-valid outcomes.
- `.error` state for invalid/generated-invalid outcomes.
- Sub-result variants for embedded PZN: success, error, and warn.
- A `.caveat` sub-line variant (distinct from `.success`/`.error`) for the EPL v2.3.x compatibility mode note, rendered beneath the success message rather than merged into it.

### T-3.4 · Copy button feedback styling
*Satisfies: FR-010, FR-013*

- Inline secondary button styling for Copy.
- No separate toast mechanism is required by the current implementation.

---

## Phase 4 — Generate Flow

Wire the Generate panel to the selected type and options.

### T-4.1 · Option synchronization
*Satisfies: FR-014, FR-015, FR-016, FR-017, FR-039, FR-041*

Implement `syncGenOptions()`:
1. Read the selected generator type.
2. Show the embed-PZN row only for `NTIN` and `PPN`.
3. Disable and clear the invalid-generation checkbox for `PCID`.
4. Re-enable invalid generation for all other types.
5. Show the `9N` prefix row only for `PPN`; leave its checked state untouched when shown/hidden so the user's last choice is remembered across type switches.
6. Show the **EPL v2.3.x compatibility mode** row only for `PPN`; leave its checked state untouched when shown/hidden.

### T-4.2 · Type-dispatched generation
*Satisfies: FR-002, FR-003, FR-014, FR-030, FR-039, FR-040, FR-041, FR-044*

On Generate button click:
1. Read the selected type.
2. Read `wantInvalid` from the checkbox unless that checkbox is disabled.
3. Read `embedPzn` from the checkbox.
4. When the type is `PPN`, read `includePrefix` from the `9N` prefix checkbox and `eplCompat` from the **EPL v2.3.x compatibility mode** checkbox.
5. Dispatch to `generatePzn`, `generateNtin`, `generateGtin`, `generatePpn` (passing `includePrefix` and `eplCompat`), or `generatePcid`.
6. Catch unexpected errors and render an error result.

### T-4.3 · Generate result labeling
*Satisfies: FR-006, FR-014, FR-042a*

- Valid generation label: `Generated valid <TYPE>`.
- Invalid generation label: `Generated invalid <TYPE> — check digit deliberately corrupted`.
- Success/error banner class is driven by whether the user requested invalid generation.
- When `type === 'ppn'`, `eplCompat === true`, and the result is valid (`wantInvalid === false`), additionally render a distinct caveat note beneath the `Generated valid PPN` message: `Note: valid only in EPL v2.3.x compatibility mode (non-standard checksum).` This note is omitted when `eplCompat === false` or when the result is invalid (the value already fails validation in that case).

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
*Satisfies: FR-001, FR-038*

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

### T-7.3 · URL anchor deep linking
*Satisfies: FR-050, FR-051, FR-052, FR-053*

Run `applyAnchorState()` (T-2.5) once after all Generate/Validate wiring (T-4.1–T-4.4, T-5.1–T-5.4) is in place, so that activating a tab or changing a type selector via the anchor exercises the exact same code paths (`syncGenOptions()`, tab-switch handler) as a manual click. Attach the `hashchange` listener at the same point.

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
- [x] `generatePpn(wantInvalid, embedPzn, includePrefix)` produces the correct 12-digit body/checksum for all four `wantInvalid` × `includePrefix` combinations, with `9N` present only when `includePrefix` is true

**Generate flow**
- [x] The type selector offers exactly PZN, NTIN, GTIN, PPN, and PCID
- [x] The embed-PZN option is shown only for NTIN and PPN
- [x] The invalid-generation option is disabled and unchecked for PCID
- [x] Generating with invalid mode on PZN, NTIN, GTIN, or PPN yields an error-styled result with the deliberate-corruption label
- [x] Generated values are stored for the Copy button
- [x] The `9N` prefix option is shown only for PPN and defaults to checked
- [x] Toggling the `9N` prefix option is independent of the invalid-generation option and affects only prefix presence, not validity
- [x] All four PPN combinations (valid+prefixed, valid+unprefixed, invalid+prefixed, invalid+unprefixed) validate consistently with `validatePpn`
- [x] `generatePpn(wantInvalid, embedPzn, includePrefix, eplCompat)` computes the checksum via `_ppnCheckEplLegacy` when `eplCompat` is true, and `validatePpn` accepts values under either checksum formula
- [x] The **EPL v2.3.x compatibility mode** option is shown only for `PPN`, defaults unchecked, and works independently of the invalid-generation, `9N`-prefix, and embed-PZN options
- [x] The **EPL v2.3.x compatibility mode** option has a tooltip (`title` attribute) and an always-visible `<small>` hint explaining its purpose
- [x] Combining **EPL v2.3.x compatibility mode** with invalid-generation produces a value that fails `validatePpn` (i.e. fails both checksum formulas)
- [ ] Generating a valid `PPN` with **EPL v2.3.x compatibility mode** on shows a distinct caveat note beneath `Generated valid PPN` stating the value is valid only under EPL v2.3.x compatibility mode and uses a non-standard checksum; the note is absent when the mode is off or the result is invalid
- [x] The optional PZN-to-embed field is shown only for `NTIN` and `PPN` and has no effect for other types
- [x] `generateNtin`/`generatePpn` accept an optional `customPzn` argument that, when a valid PZN string, is embedded verbatim (leading zeroes preserved) instead of a randomly generated inner PZN
- [x] Entering a structurally invalid value in the PZN-to-embed field blocks generation and shows an error result instead
- [x] Leaving the PZN-to-embed field blank preserves the exact prior behavior of the embed-a-valid-PZN checkbox

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

**URL anchor deep linking**
- [ ] Opening the app with no URL fragment behaves exactly as before this increment
- [ ] `#mode=generate` / `#mode=validate` (case-insensitive) activates the corresponding tab on load
- [ ] `#package-identifier=<type>` (case-insensitive, one of the five supported codes) preselects that type in both `#gen-type` and `#val-type` on load
- [ ] An unrecognized `mode` or `package-identifier` value falls back to the default (Generate tab, PZN type) without an error state
- [ ] Changing the URL fragment while the app is open (`hashchange`) re-applies the recognized `mode`/`package-identifier` state


