# Feature Specification: ePIL Package Identifier Tool

**Feature Branch**: `001-epil-pkg-identifier`

**Created**: 2026-05-21

**Status**: Draft

**Input**: User description: "Build a simple HTML5 SPA without any external dependencies for generating and validation ePIL package identifiers. Add mobile support. Users do not need any authentication."

## User Scenarios & Testing *(mandatory)*

### User Story 1 — Validate an Existing Identifier (Priority: P1)

A pharmacist or supply-chain professional has received a medicine package and wants to verify whether the identifier printed on it (or scanned from its DataMatrix code) is structurally valid before accepting or reporting it.

**Why this priority**: Validation is the most frequent daily task; it directly addresses patient-safety concerns by catching malformed or obviously fraudulent identifiers.

**Independent Test**: Open the app, paste or type a raw GS1 DataMatrix string such as `(01)03400926006476(17)260630(10)AB1234(21)SN000000001` into the validate field, press Validate, and receive a clear pass/fail report with field-level detail — no generation required.

**Acceptance Scenarios**:

1. **Given** a valid GS1-encoded identifier string, **When** the user submits it for validation, **Then** the app reports "Valid" and displays each parsed field (GTIN, expiry date, batch number, serial number) with its decoded value.
2. **Given** a string with an incorrect GTIN check digit, **When** the user submits it, **Then** the app reports "Invalid" and highlights that the product code check digit is wrong.
3. **Given** a string with an expiry date already in the past, **When** the user submits it, **Then** the app reports "Invalid" and indicates the package is expired.
4. **Given** a string that is missing a mandatory Application Identifier (e.g., no serial number AI 21), **When** the user submits it, **Then** the app reports "Invalid" and names the missing field.
5. **Given** an empty input field, **When** the user presses Validate, **Then** the app shows an inline error prompting for input rather than an empty result.

---

### User Story 2 — Generate a New Identifier (Priority: P2)

A QA engineer or packaging-line operator needs to create a test or reference ePIL package identifier for integration testing, label proofing, or demonstration.

**Why this priority**: Generation enables practical testing and demonstration without requiring access to a live serialisation system; it is the natural companion to validation.

**Independent Test**: Open the app, enter a 13-digit GTIN, a batch number, and an expiry date, press Generate, and receive a fully formed GS1 identifier string with an auto-generated serial number — without ever using the validate section.

**Acceptance Scenarios**:

1. **Given** the user has entered a valid 13-digit GTIN, a non-empty batch number, and a future expiry date, **When** they press Generate, **Then** the app outputs a complete GS1 DataMatrix string with a randomly generated serial number, and the full human-readable breakdown is shown below.
2. **Given** the user provides an invalid GTIN (wrong length or check digit), **When** they press Generate, **Then** the app highlights the GTIN field with an error message before generating anything.
3. **Given** the user provides an expiry date in the past, **When** they press Generate, **Then** the app shows a warning but still allows generation (expiry dates on existing stock may be in the past).
5. **Given** a successfully generated identifier, **When** the user presses **Copy bracket notation** or **Copy raw string**, **Then** the respective identifier string is copied to the clipboard and a brief confirmation toast is shown.
5. **Given** a successfully generated identifier, **When** the user presses Validate (quick-validate action on the result card), **Then** the generated string is pre-filled in the validate section and immediately validated.

---

### User Story 3 — Use the Tool on a Mobile Device (Priority: P3)

A warehouse inspector or field pharmacist accessing the tool on a smartphone needs the same generate/validate capabilities without pinching to zoom or losing fields off-screen.

**Why this priority**: Mobile usability is explicitly required; the tool must remain fully operable on small screens without a separate build or app install.

**Independent Test**: Open the app in a mobile browser at a viewport of 375 × 667 px (iPhone SE equivalent), complete both a generation and a validation flow entirely by touch, confirm no horizontal scrolling occurs, and verify all tap targets are reachable.

**Acceptance Scenarios**:

1. **Given** the app is loaded on a 375 px-wide viewport, **When** any section is rendered, **Then** all controls are visible without horizontal scrolling; all primary interactive elements (form inputs, primary action buttons, tab controls) are at least 44 × 44 CSS pixels; secondary action chips within result cards (copy-format and quick-action buttons) are at least 36 px tall.
2. **Given** a generated identifier string, **When** the user taps Copy on a mobile device, **Then** the clipboard API (or graceful fallback) copies the string and shows a confirmation.
3. **Given** the user navigates between Generate and Validate sections on mobile, **When** switching tabs or sections, **Then** the transition is instantaneous and no layout shift breaks the visible content.

---

### Edge Cases

- What happens when the identifier contains FNC1 separator characters (GS character, ASCII 0x1D) in addition to human-readable AI brackets?
- How does the system handle batch numbers or serial numbers containing special characters allowed by GS1 (alphanumeric plus `-`, `.`, `/`, `+`, `%`, `*`, `_`, `$`, ` `)?
- What if the GTIN is provided as GTIN-13 (13 digits) vs GTIN-14 (14 digits, padded with leading zero)?
- What happens when the user's browser does not support the Clipboard API (older browsers, non-HTTPS)?
- What if the user pastes a DataMatrix raw byte string (with GS separators) rather than the human-readable bracket notation?

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST accept a GS1 DataMatrix identifier string (human-readable bracket notation or FNC1-delimited raw format) and determine whether it is structurally valid.
- **FR-002**: Validation MUST check: GTIN-13/14 check digit (Luhn/mod-10), all four mandatory Application Identifiers present (AI 01, AI 17, AI 10, AI 21), expiry date is a valid calendar date in YYMMDD format, serial number length is 1–20 characters, batch number length is 1–20 characters.
- **FR-003**: The system MUST allow a user to enter a GTIN-13 or GTIN-14, batch number, expiry date (via date picker or text field), and an optional serial number to generate a complete GS1 DataMatrix identifier string; if no serial number is provided the system MUST auto-generate a random 10-character alphanumeric serial.
- **FR-004**: For a generated identifier, the system MUST display it in both human-readable GS1 bracket notation and as a plain string ready for DataMatrix encoding. For a validated identifier, the system MUST display a field-level breakdown (field name, AI, raw value, decoded value where applicable) rather than repeating the full identifier string.
- **FR-005**: The system MUST provide one-tap/one-click copy-to-clipboard actions for generated identifier strings in both bracket notation and raw format. Validation results do not require a dedicated copy action, as the original identifier string remains available in the input field.
- **FR-006**: The system MUST display field-level validation feedback (per-field pass/fail with descriptive error text) rather than a single pass/fail flag.
- **FR-007**: The system MUST work entirely in the browser with no server-side requests, no external libraries, and no build step required; a single HTML file opened locally or served statically MUST be fully functional.
- **FR-008**: The layout MUST be responsive and usable on viewports as narrow as 320 px. Primary interactive elements (form inputs, primary action buttons, tab controls) MUST have a minimum touch-target size of 44 × 44 CSS pixels. Secondary action chips within result cards (copy-format buttons, quick-action buttons) MAY use a reduced minimum height of 36 px.
- **FR-009**: The system MUST NOT require user authentication or any login step.
- **FR-010**: The system MUST accept GTIN-13 input and internally convert it to GTIN-14 by prepending a leading zero for encoding. The system MUST display the normalised GTIN-14 in the output; a separate GTIN-13 representation is not required.

### Key Entities

- **Package Identifier**: The complete set of serialisation data for one medicine pack. Attributes: GTIN-14 (14 digits), serial number (1–20 alphanumeric chars), batch/lot number (1–20 chars), expiry date (YYMMDD).
- **Application Identifier (AI)**: A GS1-defined numeric prefix that identifies the semantic meaning of a data field within a DataMatrix payload. Relevant AIs: `01` (GTIN), `17` (expiry date), `10` (batch/lot), `21` (serial number).
- **Validation Result**: The outcome of checking a Package Identifier. Attributes: overall status (valid/invalid), list of field-level findings (field name, status, decoded value, error description if applicable).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A user can complete a full validation of a correctly formatted identifier string in under 10 seconds from opening the app.
- **SC-002**: A user can generate a valid identifier for a given GTIN, batch, and expiry in under 15 seconds from opening the app.
- **SC-003**: The app loads and is fully interactive in under 1 second on a modern mobile browser over a local or cached connection (single-file, no network requests at runtime).
- **SC-004**: All validation rules correctly accept 100 % of structurally valid GS1 FMD-compliant identifiers and correctly reject identifiers with any one of the defined error conditions (wrong check digit, missing AI, invalid date, out-of-range field length).
- **SC-005**: No horizontal scrolling occurs on a 320 px-wide viewport; all controls remain reachable by touch.
- **SC-006**: Copy-to-clipboard succeeds in current versions of Chrome, Firefox, Safari, and Edge on both desktop and mobile.

## Assumptions

- The target standard is the EU Falsified Medicines Directive (FMD) serialisation format as defined by GS1 (GTIN-14 + AI 17 + AI 10 + AI 21 encoded in a GS1 DataMatrix).
- GTIN-13 input is accepted as a convenience alias and the app prepends a `0` to produce the required GTIN-14; GTIN-8 and GTIN-12 are out of scope.
- The app is delivered as a single self-contained `index.html` file with embedded CSS and JavaScript; no build tooling, bundler, or package manager is required.
- No data is persisted between sessions; the app maintains no local storage, cookies, or server-side state.
- Mobile support targets current-minus-one versions of iOS Safari and Android Chrome; older browsers (IE 11) are out of scope.
- The Clipboard API with a `document.execCommand` fallback covers the copy-to-clipboard requirement sufficiently for the stated browser targets.
- Serial number auto-generation uses `crypto.getRandomValues` for randomness; the format is uppercase alphanumeric (A–Z, 0–9), 10 characters.
- Expiry date input via a `<input type="date">` element is the primary mechanism; manual YYMMDD text entry is a secondary / paste-in path (validation only).
