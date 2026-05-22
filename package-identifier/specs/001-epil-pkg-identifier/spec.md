# Feature Specification: Package Identifier Tool

**Feature Branch**: `001-epil-pkg-identifier`

**Created**: 2026-05-21

**Status**: Draft

**Input**: User description: "Build a simple HTML5 SPA without any external dependencies for generating and validating package identifiers. Users do not need authentication."

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

---

### User Story 3 — Inspect Embedded PZN Information (Priority: P3)

A user working with NTIN or PPN values needs to see whether the embedded PZN can be extracted and whether that embedded PZN is itself structurally valid.

**Why this priority**: NTIN and PPN carry a nested PZN in the current implementation, and the tool exposes that additional result to aid inspection.

**Independent Test**: Validate an NTIN or PPN value and confirm that the app renders an additional embedded-PZN block showing valid, invalid, or extraction-failed status.

**Acceptance Scenarios**:

1. **Given** the user validates an `NTIN` or `PPN` whose embedded PZN can be extracted and is valid, **When** validation completes, **Then** the app shows an embedded PZN sub-result marked valid.
2. **Given** the user validates an `NTIN` or `PPN` whose embedded PZN can be extracted but is invalid, **When** validation completes, **Then** the app shows an embedded PZN sub-result marked invalid.
3. **Given** the user validates an `NTIN` or `PPN` that is too malformed to extract an embedded PZN, **When** validation completes, **Then** the app shows the message `Embedded PZN: could not be extracted`.
4. **Given** the selected Generate type is `NTIN` or `PPN`, **When** the Generate panel is shown, **Then** the app shows the option to embed a valid PZN inside the generated identifier.
5. **Given** the selected Generate type is not `NTIN` or `PPN`, **When** the Generate panel is shown, **Then** the embed-PZN option is hidden.

---

### Edge Cases

- Non-digit characters in GTIN and NTIN inputs are stripped before validation logic is applied.
- PPN validation accepts an optional leading `9N` prefix.
- PCID invalid generation is not supported.
- Clipboard writes may fail silently; copy feedback is limited to the temporary button-label change.
- The app does not auto-detect identifier type; the user must choose the intended type before generating or validating.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST provide exactly two top-level modes, `Generate` and `Validate`.
- **FR-002**: The system MUST support exactly five identifier types in both modes: `PZN`, `NTIN`, `GTIN`, `PPN`, and `PCID`.
- **FR-003**: In Generate mode, the system MUST generate a value for the selected identifier type entirely in the browser.
- **FR-004**: In Validate mode, the system MUST validate the entered value according to the selected identifier type entirely in the browser.
- **FR-005**: The system MUST require the user to select the identifier type explicitly; it MUST NOT infer the type automatically from the input value.
- **FR-006**: The system MUST display generation results in a visible result area containing the generated value and a descriptive label.
- **FR-007**: The system MUST display validation results in a visible result area containing the submitted value and a descriptive structural-validity label.
- **FR-008**: The system MUST show the message `Please enter an identifier value.` when Validate is triggered with an empty input.
- **FR-009**: The system MUST allow the Enter key in the Validate input field to trigger validation.
- **FR-010**: The system MUST provide a copy control for generated values.
- **FR-011**: On copy, the system MUST attempt to write the generated value using the browser Clipboard API when available.
- **FR-012**: On copy, the system MUST ignore clipboard-write failures silently.
- **FR-013**: On copy invocation, the copy button label MUST change to `Copied!` temporarily and later revert to `Copy`.
- **FR-014**: The system MUST provide an option to deliberately generate an invalid identifier for checksum-based types.
- **FR-015**: The system MUST disable and clear the invalid-generation option when the selected type is `PCID`.
- **FR-016**: The system MUST provide an `Embed a valid PZN inside this identifier` option only when the selected Generate type is `NTIN` or `PPN`.
- **FR-017**: The system MUST hide the embed-PZN option for `PZN`, `GTIN`, and `PCID`.
- **FR-018**: When validating `NTIN` or `PPN`, the system MUST attempt to extract an embedded PZN and render an additional sub-result.
- **FR-019**: When embedded-PZN extraction fails for `NTIN` or `PPN`, the system MUST show `Embedded PZN: could not be extracted`.
- **FR-020**: When embedded-PZN extraction succeeds for `NTIN` or `PPN`, the system MUST validate the extracted PZN and show whether it is valid or invalid.

### Identifier Rules

- **FR-021**: `PZN` validation MUST require exactly 8 digits.
- **FR-022**: `PZN` validation MUST reject the dummy values `00000000`, `22222222`, `33333333`, and `44444444`.
- **FR-023**: `PZN` validation MUST use MOD-11 over the first 7 digits with weights `1..7`.
- **FR-024**: `PZN` values whose computed check digit is `10` MUST be treated as invalid.
- **FR-025**: `PZN` invalid generation MUST be produced by shifting the computed check digit by `+1 mod 10`.

- **FR-026**: `GTIN` validation MUST strip non-digit characters before evaluation.
- **FR-027**: `GTIN` validation MUST accept only lengths `8`, `12`, `13`, and `14`.
- **FR-028**: `GTIN` validation MUST use the GS1 MOD-10 check-digit algorithm with alternating weights `3, 1` from right to left over the body digits.
- **FR-029**: `GTIN` generation MUST produce a 14-digit GTIN from a random 13-digit base plus computed check digit.
- **FR-030**: `GTIN` invalid generation MUST be produced by shifting the computed check digit by `+1 mod 10`.

- **FR-031**: `NTIN` validation MUST strip non-digit characters and require exactly 13 digits.
- **FR-032**: `NTIN` validation MUST reuse GTIN validation logic on the resulting 13-digit value.
- **FR-033**: `NTIN` generation MUST construct the value from the fixed prefix `4150`, an 8-digit inner value, and a computed check digit.
- **FR-034**: When the embed-PZN option is enabled for `NTIN`, the inner 8-digit value MUST be a generated valid PZN.
- **FR-035**: When the embed-PZN option is disabled for `NTIN`, the inner 8-digit value MUST be a random 8-digit number.
- **FR-036**: `NTIN` embedded-PZN extraction MUST use digits at positions 5 through 12 of the 13-digit value.

- **FR-037**: `PPN` validation MUST accept an optional leading `9N` prefix.
- **FR-038**: After optional prefix removal, `PPN` validation MUST require exactly 12 digits.
- **FR-039**: `PPN` validation MUST compute the checksum over the first 10 digits using character codes and multipliers `2..11`, with the result taken modulo `97`.
- **FR-040**: `PPN` generation MUST construct the value from the literal prefix `9N`, the fixed company code `11`, an 8-digit inner value, and a zero-padded 2-digit checksum.
- **FR-041**: When the embed-PZN option is enabled for `PPN`, the inner 8-digit value MUST be a generated valid PZN.
- **FR-042**: When the embed-PZN option is disabled for `PPN`, the inner 8-digit value MUST be a random 8-digit number.
- **FR-043**: `PPN` embedded-PZN extraction MUST use digits at positions 3 through 10 after optional `9N` removal.

- **FR-044**: `PCID` generation MUST use `crypto.randomUUID()` when available and otherwise fall back to an internally generated UUID-like value.
- **FR-045**: `PCID` validation MUST accept UUIDs matching RFC 4122 versions `1` through `5`.
- **FR-046**: The system MUST NOT support deliberately invalid `PCID` generation.

### Non-Functional Requirements

- **FR-047**: The system MUST be delivered as a single self-contained HTML file with inline CSS and JavaScript.
- **FR-048**: The system MUST run without authentication or login.
- **FR-049**: The system MUST not depend on any external JavaScript, CSS, fonts, images, or network services.
- **FR-050**: The system MUST keep computation client-side in the browser.
- **FR-051**: The system MUST not require persistent client storage; no saved state between sessions is required.
- **FR-052**: The interface MUST use a single-column, centered layout with tab-based navigation and visible success/error result states.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A user can generate a value for each of the five supported identifier types from the Generate panel without leaving the page.
- **SC-002**: A user can validate a value for each of the five supported identifier types from the Validate panel and receive an explicit structurally valid or structurally invalid result.
- **SC-003**: For `NTIN` and `PPN`, validation always produces an embedded-PZN sub-result in one of three states: valid, invalid, or could not be extracted.
- **SC-004**: For `PZN`, `NTIN`, `GTIN`, and `PPN`, enabling invalid generation produces a value that fails validation for the same type.
- **SC-005**: For `PCID`, invalid generation cannot be selected by the user.
- **SC-006**: Copying a generated value changes the copy button label to `Copied!` and then back to `Copy`.
- **SC-007**: The app loads and operates as a standalone local HTML file with no required backend communication.
- **SC-008**: The app remains usable in its default centered single-column layout on typical desktop and mobile browser widths supported by fluid CSS.

## Assumptions

- The implementation is the source of truth for current behavior.
- Users know which identifier type they intend to work with and will select that type explicitly before generating or validating.
- Structural validation is sufficient; no semantic, registry, issuance, or business-rule verification is performed.
- `GTIN` generation is limited to 14-digit output in the current implementation; the tool does not generate GTIN-8, GTIN-12, or GTIN-13 values directly.
- `NTIN` and `PPN` embedded-PZN inspection is an additional informational result and does not replace the outer identifier's own validation result.
- Clipboard behavior depends on browser support for `navigator.clipboard.writeText`; the current implementation does not provide a fallback copy mechanism.
- The UI is responsive only through fluid layout rules already present in the single HTML document; there are no explicit breakpoint-specific or touch-target guarantees in the current implementation.
