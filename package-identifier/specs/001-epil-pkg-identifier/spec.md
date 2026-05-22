# Feature Specification: Package Identifier Tool

**Feature Branch**: `001-epil-pkg-identifier`

**Created**: 2026-05-21

**Status**: Draft

**Input**: User description: "Build a simple package identifier tool for generating and validating identifiers without requiring authentication."

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

**Why this priority**: NTIN and PPN carry a nested PZN, and the tool exposes that additional result to aid inspection.

**Independent Test**: Generate or validate an NTIN or PPN value and confirm that the app renders an additional embedded-PZN block showing valid, invalid, or extraction-failed status.

**Acceptance Scenarios**:

1. **Given** the user generates or validates an `NTIN` or `PPN` whose embedded PZN can be extracted and is valid, **When** the result is shown, **Then** the app shows an embedded PZN sub-result marked valid.
2. **Given** the user generates or validates an `NTIN` or `PPN` whose embedded PZN can be extracted but is invalid, **When** the result is shown, **Then** the app shows an embedded PZN sub-result marked invalid.
3. **Given** the user generates or validates an `NTIN` or `PPN` that is too malformed to extract an embedded PZN, **When** the result is shown, **Then** the app shows the message `Embedded PZN: could not be extracted`.
4. **Given** the selected Generate type is `NTIN` or `PPN`, **When** the Generate panel is shown, **Then** the app shows the option to embed a valid PZN inside the generated identifier.
5. **Given** the selected Generate type is not `NTIN` or `PPN`, **When** the Generate panel is shown, **Then** the embed-PZN option is hidden.

---

### Edge Cases

- Formatting characters may be tolerated for some numeric identifier types.
- PPN values may be entered with or without their visible prefix.
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

## Assumptions

- Users know which identifier type they intend to work with and will select that type explicitly before generating or validating.
- Structural validation is sufficient; no semantic, registry, issuance, or business-rule verification is performed.
- `GTIN` generation is limited to 14-digit output; the tool does not generate GTIN-8, GTIN-12, or GTIN-13 values directly.
- `NTIN` and `PPN` embedded-PZN inspection is an additional informational result and does not replace the outer identifier's own validation result.
- Copy support depends on platform capabilities and may not be available in every environment.
- The interface is intended to remain usable in a compact single-page layout without separate device-specific modes.
