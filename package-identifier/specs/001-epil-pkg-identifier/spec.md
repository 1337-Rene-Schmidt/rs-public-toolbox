# Feature Specification: Package Identifier Tool

**Feature Branch**: `001-epil-pkg-identifier`

**Created**: 2026-05-21

**Status**: Draft

**Input**: User description: "Build a simple package identifier tool for generating and validating identifiers without requiring authentication."

## Clarifications

### Session 2026-09-22

- Q: Where should the "only valid under EPL v2.3.x compatibility mode, non-standard checksum" caveat appear when a PPN is generated with EPL v2.3.x compatibility mode enabled? → A: Keep `Generated valid PPN` as-is and show a separate, clearly-labeled caveat note/line beneath it whenever EPL v2.3.x compatibility mode produced the value.

## User Scenarios & Testing *(mandatory)*

### User Story 1 — Validate an Identifier by Type (Priority: P1)

A pharmacist, QA analyst, or supply-chain operator needs to check whether a selected identifier value is structurally valid for a known identifier type.

**Why this priority**: Validation is the fastest way to confirm whether a value conforms to the expected structure before using it in testing or operational workflows.

**Independent Test**: Open the app, switch to Validate, choose a type, enter an identifier string, press Validate, and observe a clear valid/invalid result without using Generate.

**Acceptance Scenarios**:

1. **Given** the user selects `PZN`, `NTIN`, `GTIN`, `PPN`, or `PCID`, **When** they enter a structurally valid value for that type and press Validate, **Then** the app shows a success result stating that the selected type is structurally valid.
2. **Given** the user selects one of the supported types, **When** they enter a structurally invalid value and press Validate, **Then** the app shows an error result stating that the selected type is structurally invalid.
3. **Given** the Validate input is empty, **When** the user presses Validate, **Then** the app shows the message `Please enter an identifier value.` and does not attempt algorithmic validation.
4. **Given** the user is focused in the Validate input, **When** they press Enter, **Then** validation is triggered.

---

### User Story 2 — Generate an Identifier by Type (Priority: P2)

A developer, tester, or packaging operator needs to generate example identifiers for supported types without relying on backend services or external tools.

**Why this priority**: Generation makes the tool useful for demos, integration testing, and quick reference data creation.

**Independent Test**: Open the app, stay on Generate, choose each supported type in turn, press Generate, and confirm that a corresponding identifier value is shown.

**Acceptance Scenarios**:

1. **Given** the user selects `PZN`, `NTIN`, `GTIN`, `PPN`, or `PCID`, **When** they press Generate, **Then** the app produces a value for that selected type and displays it in the result area.
2. **Given** the selected type is checksum-based (`PZN`, `NTIN`, `GTIN`, or `PPN`) and the user enables the invalid option, **When** they press Generate, **Then** the app produces a deliberately invalid identifier and labels it as invalid.
3. **Given** the selected type is `PCID`, **When** the user views Generate options, **Then** the invalid-generation option is disabled and cleared.
4. **Given** a value has been generated, **When** the user presses Copy, **Then** the app attempts to copy the generated value to the clipboard and temporarily changes the button label to `Copied!`.
5. **Given** the selected Generate type is `PPN`, **When** the user toggles the `9N` prefix option, **Then** the generated value includes or omits the visible `9N` prefix accordingly, independent of the invalid-generation option.
6. **Given** the selected Generate type is `PPN`, **When** the user enables **EPL v2.3.x compatibility mode** and presses Generate, **Then** the app produces a PPN whose check digit is computed so that the EPL backend's current PPN validation accepts it, whether or not the embed-PZN option is enabled, and the result area shows a distinct caveat note (separate from the `Generated valid PPN` success message) stating that the value is valid only under EPL v2.3.x compatibility mode and uses a non-standard checksum.
7. **Given** the selected Generate type is `NTIN` or `PPN`, **When** the user enters a structurally valid PZN into the optional PZN-to-embed field and presses Generate, **Then** the app produces a value that embeds exactly that PZN.
8. **Given** the selected Generate type is `NTIN` or `PPN`, **When** the user enters a structurally invalid value into the optional PZN-to-embed field and presses Generate, **Then** the app shows an error result and does not produce a generated value.

---

### User Story 3 — Inspect Embedded PZN Information (Priority: P3)

A user working with NTIN or PPN values needs to see whether the embedded PZN can be extracted and whether that embedded PZN is itself structurally valid.

**Why this priority**: NTIN and PPN carry a nested PZN, and the tool exposes that additional result to aid inspection.

**Independent Test**: Generate or validate an NTIN or PPN value and confirm that the app renders an additional embedded-PZN block showing valid, invalid, or extraction-failed status.

**Acceptance Scenarios**:

1. **Given** the user generates or validates an `NTIN` or `PPN` whose embedded PZN can be extracted and is valid, **When** the result is shown, **Then** the app shows an embedded PZN sub-result marked valid.
2. **Given** the user generates or validates an `NTIN` or `PPN` whose embedded PZN can be extracted but is invalid, **When** the result is shown, **Then** the app shows an embedded PZN sub-result marked invalid.
3. **Given** the user generates or validates an `NTIN` or `PPN` that is too malformed to extract an embedded PZN, **When** the result is shown, **Then** the app shows the message `Embedded PZN: could not be extracted`.
4. **Given** the selected Generate type is `NTIN` or `PPN`, **When** the Generate panel is shown, **Then** the app shows the option to embed a valid PZN inside the generated identifier.
5. **Given** the selected Generate type is not `NTIN` or `PPN`, **When** the Generate panel is shown, **Then** the embed-PZN option is hidden.

---

### User Story 4 — Deep Link to a Mode and Identifier Type via URL Anchor (Priority: P4)

A user shares or bookmarks a link that opens the tool directly in a specific mode with a specific identifier type already selected, instead of always landing on the default Generate/PZN view.

**Why this priority**: Deep linking is a convenience layer over the existing modes and types; it changes nothing about the underlying generation/validation behavior, so it is the lowest-priority increment.

**Independent Test**: Open the app with a URL fragment such as `#mode=validate&package-identifier=ppn` and confirm the Validate tab is active with PPN preselected; open the app with no fragment and confirm it behaves exactly as before.

**Acceptance Scenarios**:

1. **Given** the URL fragment contains no `mode` and no `package-identifier` key, **When** the app loads, **Then** the app behaves exactly as it does today (Generate tab active, `PZN` selected).
2. **Given** the URL fragment contains `mode=generate`, **When** the app loads, **Then** the Generate tab is active.
3. **Given** the URL fragment contains `mode=validate`, **When** the app loads, **Then** the Validate tab is active.
4. **Given** the URL fragment contains `package-identifier=<type>` where `<type>` case-insensitively matches `pzn`, `ntin`, `gtin`, `ppn`, or `pcid`, **When** the app loads, **Then** that identifier type is preselected in both the Generate and Validate type selectors.
5. **Given** the URL fragment contains an unrecognized `mode` or `package-identifier` value, **When** the app loads, **Then** the corresponding default (Generate tab, `PZN` type) is used instead, without any error state being shown.
6. **Given** the app is already open, **When** the URL fragment is changed (e.g. via browser back/forward navigation or a manual edit) to a recognized `mode`/`package-identifier` combination, **Then** the app re-applies the corresponding tab activation and type preselection.
7. **Given** the app is on the default view with no URL fragment, **When** the user clicks the `Validate` tab, **Then** the URL fragment updates to include `mode=validate` without reloading the page or adding a new browser-history entry.
8. **Given** the app is showing the Generate panel, **When** the user changes the Generate identifier-type selector to a different supported type, **Then** the URL fragment updates to include `package-identifier=<type>` reflecting that selection.
9. **Given** the app is showing the Validate panel, **When** the user changes the Validate identifier-type selector to a different supported type, **Then** the URL fragment updates to include `package-identifier=<type>` reflecting that selection.
10. **Given** the URL fragment contains an unrelated key (e.g. `#foo=bar`), **When** the user switches tabs or changes an identifier-type selector, **Then** the app updates only the `mode`/`package-identifier` keys and leaves `foo=bar` intact.

---

### Edge Cases

- Formatting characters may be tolerated for some numeric identifier types.
- PPN values may be entered with or without their visible prefix.
- PPN generation may deliberately omit the `9N` prefix even when embedding a valid PZN or generating an invalid check digit.
- **EPL v2.3.x compatibility mode** may be combined with the invalid-generation option; the resulting value MUST still fail structural validation.
- The optional PZN-to-embed field has no effect for identifier types other than `NTIN` and `PPN`.
- Leading/trailing whitespace in the PZN-to-embed field is trimmed before validation; a whitespace-only value is treated as blank.
- Additional unrecognized keys or malformed `key=value` pairs in the URL fragment MUST be ignored without affecting any recognized keys.
- A URL fragment containing only one of `mode`/`package-identifier` MUST apply only that recognized part and leave the other at its default.
- Anchor-derived state application (reading the fragment on load/`hashchange`) MUST NOT itself trigger a write-back update, avoiding an infinite read/write loop.
- PCID invalid generation is not supported.
- Copy may be unavailable in some environments; failures are silent.
- The app does not auto-detect identifier type; the user must choose the intended type before generating or validating.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST provide exactly two top-level modes, `Generate` and `Validate`.
- **FR-002**: The system MUST support exactly five identifier types in both modes: `PZN`, `NTIN`, `GTIN`, `PPN`, and `PCID`.
- **FR-003**: In Generate mode, the system MUST generate a value for the selected identifier type without requiring a backend workflow.
- **FR-004**: In Validate mode, the system MUST validate the entered value according to the selected identifier type without requiring a backend workflow.
- **FR-005**: The system MUST require the user to select the identifier type explicitly; it MUST NOT infer the type automatically from the input value.
- **FR-006**: The system MUST display generation results in a visible result area containing the generated value and a descriptive label.
- **FR-007**: The system MUST display validation results in a visible result area containing the submitted value and a descriptive structural-validity label.
- **FR-008**: The system MUST show the message `Please enter an identifier value.` when Validate is triggered with an empty input.
- **FR-009**: The system MUST allow the Enter key in the Validate input field to trigger validation.
- **FR-010**: The system MUST provide a copy control for generated values.
- **FR-011**: On copy, the system MUST attempt to copy the generated value when copy support is available.
- **FR-012**: On copy, the system MUST ignore copy failures silently.
- **FR-013**: On copy invocation, the copy button label MUST change to `Copied!` temporarily and later revert to `Copy`.
- **FR-014**: The system MUST provide an option to deliberately generate an invalid identifier for checksum-based types.
- **FR-015**: The system MUST disable and clear the invalid-generation option when the selected type is `PCID`.
- **FR-016**: The system MUST provide an `Embed a valid PZN inside this identifier` option only when the selected Generate type is `NTIN` or `PPN`.
- **FR-017**: The system MUST hide the embed-PZN option for `PZN`, `GTIN`, and `PCID`.
- **FR-018**: When generating or validating `NTIN` or `PPN`, the system MUST attempt to extract an embedded PZN and render an additional sub-result.
- **FR-019**: When embedded-PZN extraction fails for `NTIN` or `PPN`, the system MUST show `Embedded PZN: could not be extracted`.
- **FR-020**: When embedded-PZN extraction succeeds for `NTIN` or `PPN`, the system MUST validate the extracted PZN and show whether it is valid or invalid.

### Identifier Rules

- **FR-021**: `PZN` validation MUST accept only structurally valid PZN values and reject known dummy placeholder values.
- **FR-022**: `PZN` generation MUST be able to produce both valid and deliberately invalid examples.
- **FR-023**: `GTIN` validation MUST accept the supported standard GTIN lengths.
- **FR-024**: `GTIN` generation MUST produce valid GTIN examples in the current supported output form.
- **FR-025**: `GTIN` generation MUST also be able to produce deliberately invalid examples.
- **FR-026**: `NTIN` validation MUST accept only structurally valid NTIN values.
- **FR-027**: `NTIN` generation MUST create values that follow the current NTIN structure and may optionally contain a valid embedded PZN.
- **FR-028**: `PPN` validation MUST accept only structurally valid PPN values and tolerate the visible PPN prefix when present.
- **FR-029**: `PPN` generation MUST create values that follow the current PPN structure and may optionally contain a valid embedded PZN.
- **FR-030**: `PCID` generation MUST create structurally valid PCID values.
- **FR-031**: `PCID` validation MUST accept structurally valid PCID values.
- **FR-032**: The system MUST NOT support deliberately invalid `PCID` generation.
- **FR-033**: For identifier types that support invalid generation, a deliberately invalid example MUST fail validation for its own type.

### Non-Functional Requirements

- **FR-034**: The system MUST run without authentication or login.
- **FR-035**: The system MUST not require persistent client storage; no saved state between sessions is required.
- **FR-036**: The system MUST operate without requiring external service integration for its core generation and validation flows.
- **FR-037**: The interface MUST present clear success and error states for every generate and validate operation.
- **FR-038**: The interface MUST present the workflow in a compact, single-page layout with tab-based navigation.

### PPN Prefix Generation

- **FR-039**: In Generate mode, when the selected type is `PPN`, the system MUST provide an option, independent of the invalid-generation option, to include or omit the visible `9N` data-identifier prefix on the generated value.
- **FR-040**: `PPN` generation MUST support all four combinations of the invalid-generation option and the `9N`-prefix option (valid+prefixed, valid+unprefixed, invalid+prefixed, invalid+unprefixed); each combination MUST validate consistently with `validatePpn`, whose result MUST NOT depend on prefix presence.

### PPN EPL v2.3.x Compatibility Mode

- **FR-041**: In Generate mode, when the selected type is `PPN`, the system MUST provide an **EPL v2.3.x compatibility mode** option, independent of the invalid-generation and `9N`-prefix options, that computes the PPN's check digit using the checksum formula currently expected by the EPL backend service rather than the standard ISO/IEC 7064 MOD 97-10 checksum. This formula selection applies as the basis for the check digit regardless of whether the invalid-generation option is also enabled (see FR-045 for the invalid case).
- **FR-042**: The system MUST provide explanatory help text (a tooltip) for the **EPL v2.3.x compatibility mode** option describing that it generates PPNs compatible with the EPL backend's current, non-standard PPN checksum validation, for interoperability until that backend defect is fixed.
- **FR-042a**: When **EPL v2.3.x compatibility mode** produces a structurally valid PPN, the generation result area MUST show a distinct caveat note, separate from the `Generated valid PPN` success message, stating that the value is valid only under EPL v2.3.x compatibility mode and uses a non-standard checksum.
- **FR-043**: `PPN` structural validation MUST accept a value whose checksum satisfies either the standard ISO/IEC 7064 MOD 97-10 formula or the EPL-compatible legacy formula, so that a PPN generated in EPL v2.3.x compatibility mode also validates successfully when checked in Validate mode.
- **FR-044**: **EPL v2.3.x compatibility mode** MUST work with or without the embed-a-valid-PZN option enabled.
- **FR-045**: When **EPL v2.3.x compatibility mode** is combined with the invalid-generation option, the produced value MUST fail structural validation under both the standard and the EPL-compatible checksum formulas.

### NTIN/PPN Custom PZN Embedding

- **FR-046**: In Generate mode, when the selected type is `NTIN` or `PPN`, the system MUST provide an optional free-text field where the user may specify the exact PZN value to embed, instead of relying on the embed-a-valid-PZN checkbox to choose one automatically.
- **FR-047**: When the optional PZN-to-embed field is non-blank, the system MUST validate its value using the same rules as `PZN` validation before generating; if the value is not a structurally valid PZN, the system MUST show an error result and MUST NOT produce a generated `NTIN`/`PPN` value.
- **FR-048**: When the optional PZN-to-embed field is non-blank and structurally valid, `NTIN`/`PPN` generation MUST embed that exact value (including leading zeroes) regardless of the embed-a-valid-PZN checkbox's state.
- **FR-049**: When the optional PZN-to-embed field is blank (or contains only whitespace), `NTIN`/`PPN` generation MUST behave exactly as previously specified (FR-016, FR-027, FR-029), governed solely by the embed-a-valid-PZN checkbox.

### URL Anchor / Deep Linking

- **FR-050**: On initial load, if the URL fragment contains a `mode` key whose value case-insensitively matches `generate` or `validate`, the system MUST activate the corresponding tab instead of the default `Generate` tab.
- **FR-051**: On initial load, if the URL fragment contains a `package-identifier` key whose value case-insensitively matches one of the five supported identifier codes (`pzn`, `ntin`, `gtin`, `ppn`, `pcid`), the system MUST preselect that identifier type in both the Generate and Validate type selectors.
- **FR-052**: The system MUST fall back to its default behavior (`Generate` tab active, `PZN` type selected) whenever the URL fragment is absent, empty, or contains a `mode` or `package-identifier` value that does not match a recognized value; unrecognized keys or values MUST be ignored without producing an error state.
- **FR-053**: The system MUST re-apply the anchor-derived mode/type state (per FR-050/FR-051) whenever the URL fragment changes during the session (e.g. via browser back/forward navigation or a manually edited fragment), not only at initial page load.

### URL Anchor Write-Back

- **FR-054**: When the user activates a tab via its tab button (`Generate` or `Validate`), the system MUST update the URL fragment's `mode` key to the newly active tab's lowercase name, using a mechanism that neither reloads the page nor creates a new browser-history entry.
- **FR-055**: When the user changes the value of either identifier-type selector (the Generate panel's or the Validate panel's), the system MUST update the URL fragment's `package-identifier` key to the newly selected type's lowercase code, using the same non-reloading, non-history-creating mechanism as FR-054.
- **FR-056**: URL fragment write-back (FR-054/FR-055) MUST preserve any other existing `key=value` pairs already present in the fragment that are not `mode` or `package-identifier`.
- **FR-057**: URL fragment write-back MUST NOT itself trigger the fragment-change re-application described in FR-053 (no feedback loop between reading and writing anchor state).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A user can generate a value for each of the five supported identifier types from the Generate panel without leaving the page.
- **SC-002**: A user can validate a value for each of the five supported identifier types from the Validate panel and receive an explicit structurally valid or structurally invalid result.
- **SC-003**: For `NTIN` and `PPN`, generation and validation produce an embedded-PZN sub-result in one of three states: valid, invalid, or could not be extracted.
- **SC-004**: For `PZN`, `NTIN`, `GTIN`, and `PPN`, enabling invalid generation produces a value that fails validation for the same type.
- **SC-005**: For `PCID`, invalid generation cannot be selected by the user.
- **SC-006**: Copying a generated value changes the copy button label to `Copied!` and then back to `Copy`.
- **SC-007**: A user can use the tool without signing in or connecting to a backend account.
- **SC-008**: The app remains usable in its default compact single-page layout on typical desktop and mobile widths.
- **SC-009**: For `PPN`, generating with any combination of the invalid-generation and `9N`-prefix options produces a value whose structural validity matches the requested invalid state, regardless of whether the `9N` prefix is present.
- **SC-010**: For `PPN`, generating with **EPL v2.3.x compatibility mode** enabled produces a value that validates successfully in the app's own Validate mode and uses the checksum formula the EPL backend currently expects; combining this option with invalid-generation always produces a value that fails validation under both recognized checksum formulas.
- **SC-011**: For `NTIN` and `PPN`, entering a structurally valid PZN into the optional PZN-to-embed field produces a generated value that embeds exactly that PZN; entering a structurally invalid value produces an error result and no generated value.
- **SC-012**: Opening the app with a URL fragment such as `#mode=validate&package-identifier=ppn` activates the Validate tab with `PPN` preselected in both type selectors without any manual clicks; opening the app with no fragment, or with an unrecognized fragment, behaves exactly as it did before this increment.
- **SC-013**: Generating a valid `PPN` with **EPL v2.3.x compatibility mode** enabled always shows a distinct caveat note beneath the success message; the note is absent whenever the mode is off or the generated result is invalid.
- **SC-014**: Switching tabs or changing either identifier-type selector updates the URL fragment's `mode`/`package-identifier` keys to reflect the current UI state, without a page reload, without adding a browser-history entry, and without disturbing unrelated fragment keys already present.

## Assumptions

- Users know which identifier type they intend to work with and will select that type explicitly before generating or validating.
- Structural validation is sufficient; no semantic, registry, issuance, or business-rule verification is performed.
- `GTIN` generation is limited to 14-digit output; the tool does not generate GTIN-8, GTIN-12, or GTIN-13 values directly.
- `NTIN` and `PPN` embedded-PZN inspection is an additional informational result and does not replace the outer identifier's own validation result.
- Copy support depends on platform capabilities and may not be available in every environment.
- The interface is intended to remain usable in a compact single-page layout without separate device-specific modes.
- The EPL backend service currently validates PPN check digits using a legacy formula that differs from the ISO/IEC 7064 MOD 97-10 standard; **EPL v2.3.x compatibility mode** is a temporary interoperability accommodation for that discrepancy, not a redefinition of the PPN standard. Accepting either checksum formula in Validate mode marginally increases the chance of a false-positive structural validation for arbitrary pasted input, which is an accepted trade-off for this tool's structural-testing purpose.
- A manually entered PZN-to-embed value is trusted only after passing the same structural validation as PZN Validate mode; no additional semantic, registry, or issuance verification is performed on it.
- The URL fragment (`#...`) is parsed as query-string-style `key=value` pairs separated by `&`, mirroring `URLSearchParams` semantics; this is a presentation/deep-linking convenience only and introduces no new persistent state (Constitution Article II).
- Anchor-derived preselection affects both the Generate and Validate type selectors simultaneously, since a single `package-identifier` value cannot address only one panel.
- URL fragment write-back (FR-054–FR-057) is a convenience mirror of current UI state for shareable/bookmarkable links; it does not persist anything server-side or in storage (Constitution Article II), and uses a history-neutral update mechanism (e.g. `history.replaceState`) so back/forward navigation is not cluttered with a per-click entry.
