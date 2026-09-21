---

description: "Task list template for feature implementation"
---

# Tasks: Package Identifier Tool

**Input**: Design documents from `/specs/001-epil-pkg-identifier/`

**Prerequisites**: plan.md (required), spec.md (required for user stories), constitution.md

**Tests**: Not requested in spec.md; no test tasks are included.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story. The entire application is a single self-contained `index.html` file (inline CSS + JavaScript, no build step), so nearly all tasks target the same file and generally cannot run in parallel against each other.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **Single file**: everything lives in `index.html` at the repository root (per constitution Article I).

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic document/tab structure

- [x] T001 Create `index.html` document shell: `<!DOCTYPE html>`, `lang="en"`, `<meta charset="UTF-8">`, mobile viewport meta tag, and page title `Package Identifier Tool`
- [x] T002 Add base layout CSS in index.html: system font stack, neutral light background, one centered `.container` with `max-width` around 660px, white card surfaces with subtle shadow and rounded corners
- [x] T003 Add tab button and panel CSS in index.html: styled tab buttons with active underline state, `.panel`/`.panel.active` show/hide rules
- [x] T004 Add tab navigation markup in index.html: two tab buttons labelled `Generate` and `Validate`, and two corresponding panels with Generate active on initial load
- [x] T005 Wire tab switching click handlers in index.html: on click, remove `.active` from all tab buttons and panels, activate the clicked tab, and activate the matching panel via its `data-tab` value

**Checkpoint**: Shell loads as a static `file://` page with working tab switching between two empty panels

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Pure algorithm functions shared by both Validate (US1) and Generate (US2), and required by embedded-PZN inspection (US3). No user story can be completed without these.

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [x] T006 Implement `_pznRawCheck(base7)` and `validatePzn(value)` in index.html: require exactly 8 digits, reject the dummy set `{00000000, 22222222, 33333333, 44444444}`, reject a computed check digit of `10`, and compare against the final digit (weights `1..7` summed, `sum % 11`)
- [x] T007 Implement `generatePzn(wantInvalid)` in index.html: create a random 7-digit base, compute the check digit, and optionally corrupt it by shifting `+1 mod 10` (depends on T006 for verification)
- [x] T008 Implement `_gtinCheck(base)` and `validateGtin(value)` in index.html: strip non-digits, require length in `{8, 12, 13, 14}`, apply GS1 MOD-10 with alternating weights `3, 1` from right to left, and compare the computed check digit against the final digit
- [x] T009 Implement `generateGtin(wantInvalid)` in index.html: create a 13-digit random base, compute the check digit, return a 14-digit GTIN, and optionally corrupt the digit by shifting `+1 mod 10` (depends on T008)
- [x] T010 Implement `validateNtin(value)` in index.html: strip non-digits, require exactly 13 digits, then delegate to `validateGtin` (depends on T008)
- [x] T011 Implement `generateNtin(wantInvalid, embedPzn)` in index.html: build `4150 + inner + check`, where `inner` is either a valid generated PZN (via `generatePzn`) or a random 8-digit number (depends on T007, T009, T010)
- [x] T012 Implement `_decimalModulo(decimalString, divisor)`, `_ppnCheck(body10)`, and `validatePpn(value)` in index.html per ISO/IEC 7064 MOD 97-10: `body10 = "11" + pzn`, `check = 98 - decimalModulo(body10 + "00", 97)` zero-padded to 2 digits; validation removes an optional `9N` prefix, requires exactly 12 digits, and confirms `decimalModulo(value, 97) === 1`
- [x] T013 Implement `generatePpn(wantInvalid, embedPzn)` in index.html: build `9N + 11 + inner + checksum`, where `inner` is either a valid generated PZN (via `generatePzn`) or a random 8-digit number, and corrupt the checksum by shifting `+1 mod 97` when `wantInvalid` (depends on T007, T012)
- [x] T014 Implement `generatePcid()` in index.html: prefer `crypto.randomUUID()` and fall back to an internal UUID-v4-style template when unavailable
- [x] T015 Implement `validatePcid(value)` in index.html: validate against the RFC 4122 versions `1..5` regex `^[0-9a-f]{8}-[0-9a-f]{4}-[1-5][0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$` (case-insensitive)

**Checkpoint**: All five validate/generate function pairs are callable directly in the browser console and satisfy the generator–validator duality (constitution Article IV) — foundation ready for UI wiring

---

## Phase 3: User Story 1 - Validate an Identifier by Type (Priority: P1) 🎯 MVP

**Goal**: Let a user choose an identifier type, enter a value, and see a clear structurally-valid/invalid result.

**Independent Test**: Open the app, switch to Validate, choose a type, enter an identifier string, press Validate, and observe a clear valid/invalid result without using Generate.

### Implementation for User Story 1

- [x] T016 [US1] Add Validate panel markup in index.html: type selector with options `PZN`, `NTIN`, `GTIN`, `PPN`, `PCID`; a freeform text input for the identifier value; a Validate button; and a result block containing the validated value, result label, and an embedded-PZN container
- [x] T017 [US1] Implement the empty-input guard in `runValidation()` in index.html: when the trimmed Validate input is empty, show an error result with an em dash placeholder value and the exact message `Please enter an identifier value.`, without invoking any validator
- [x] T018 [US1] Implement `runValidation()` type-dispatched validation in index.html: read the selected type and dispatch to `validatePzn`, `validateNtin`, `validateGtin`, `validatePpn`, or `validatePcid`, using the selected type label rather than auto-detection (depends on T006, T008, T010, T012, T015)
- [x] T019 [US1] Implement validate result labeling in index.html: valid message `✓ <TYPE> is structurally valid`, invalid message `✗ <TYPE> is structurally invalid`, and always show the submitted input value when validation runs
- [x] T020 [US1] Attach a `keydown` listener on the Validate input in index.html so pressing Enter triggers `runValidation()`

**Checkpoint**: User Story 1 is fully functional and independently testable — validating any of the five types produces a correct success/error result, and Enter triggers validation

---

## Phase 4: User Story 2 - Generate an Identifier by Type (Priority: P2)

**Goal**: Let a user choose an identifier type and generate a valid (or deliberately invalid) example value, with copy-to-clipboard support.

**Independent Test**: Open the app, stay on Generate, choose each supported type in turn, press Generate, and confirm that a corresponding identifier value is shown.

### Implementation for User Story 2

- [x] T021 [US2] Add Generate panel markup in index.html: type selector with the same five options; an embed-PZN checkbox row present in the markup and shown only for `NTIN` and `PPN`; an invalid-generation checkbox row that is disabled and cleared when `PCID` is selected; a Generate button; and a result block containing the generated value, result label, and Copy button
- [x] T022 [US2] Implement `syncGenOptions()` in index.html: read the selected generator type, show the embed-PZN row only for `NTIN` and `PPN`, disable and clear the invalid-generation checkbox for `PCID`, and re-enable invalid generation for all other types
- [x] T023 [US2] Implement the Generate button click handler in index.html: read the selected type, read `wantInvalid` from the checkbox unless disabled, read `embedPzn` from the checkbox, dispatch to `generatePzn`, `generateNtin`, `generateGtin`, `generatePpn`, or `generatePcid`, and catch unexpected errors to render an error result (depends on T007, T009, T011, T013, T014)
- [x] T024 [US2] Implement generate result labeling in index.html: valid label `Generated valid <TYPE>`, invalid label `Generated invalid <TYPE> — check digit deliberately corrupted`, with success/error banner class driven by whether invalid generation was requested
- [x] T025 [US2] Store the generated value in `#gen-copy.dataset.copy` in index.html after successful generation, for later copy invocation
- [x] T026 [US2] Wire the Generate Copy button in index.html: read `dataset.copy`, call `navigator.clipboard?.writeText(value)` when a value is present, silently ignore promise rejection, and change the button label to `Copied!`, reverting to `Copy` after 1.5 seconds (depends on T025)
- [ ] T036 [US2] Extend `generatePpn(wantInvalid, embedPzn, includePrefix = true)` in index.html per [contracts/ppn-generation.md](contracts/ppn-generation.md): prepend `9N` only when `includePrefix` is true; the 12-digit body and its checksum computation (`_ppnCheck`/`_decimalModulo`) MUST NOT change, so `validatePpn` and `extractPznFromPpn` behave identically regardless of `includePrefix` (depends on T013)
- [ ] T037 [US2] Add a `9N` prefix checkbox row in index.html inside `.options-block`, after the invalid-generation row: `id="row-ppn-prefix"` wrapping `<input type="checkbox" id="gen-ppn-prefix" checked>`, hidden by default via the existing `.hidden` class pattern
- [ ] T038 [US2] Update `syncGenOptions()` in index.html: show `row-ppn-prefix` only when the selected type is `ppn`; do not alter the checkbox's checked state when toggling visibility (depends on T022, T037)
- [ ] T039 [US2] Update the Generate button click handler in index.html: when the selected type is `ppn`, read `includePrefix` from `#gen-ppn-prefix.checked` and pass it as the third argument to `generatePpn`; other types are unaffected (depends on T023, T036, T037)

**Checkpoint**: User Stories 1 AND 2 both work independently — generating any of the five types produces a labeled result, Copy provides visible feedback, and PPN generation independently supports the invalid and `9N`-prefix options per FR-039/FR-040

---

## Phase 5: User Story 3 - Inspect Embedded PZN Information (Priority: P3)

**Goal**: For `NTIN` and `PPN` results, show whether the embedded PZN can be extracted and whether that embedded PZN is itself structurally valid.

**Independent Test**: Generate or validate an NTIN or PPN value and confirm that the app renders an additional embedded-PZN block showing valid, invalid, or extraction-failed status.

### Implementation for User Story 3

- [x] T027 [US3] Implement `extractPznFromNtin(value)` in index.html: strip non-digits and return digits `slice(4, 12)` when the value is 13 digits; otherwise return `null`
- [x] T028 [US3] Implement `extractPznFromPpn(value)` in index.html: remove an optional `9N` prefix, then return digits `slice(2, 10)` when the stripped value is 12 digits; otherwise return `null`
- [x] T029 [US3] Implement `buildEmbeddedPznBlock(pzn)` in index.html: if extraction returns `null`, render warn text `Embedded PZN: could not be extracted`; otherwise validate the PZN with `validatePzn(pzn)` and render a success or error sub-block stating whether the embedded PZN is valid (depends on T006, T027, T028)
- [x] T030 [US3] Wire embedded-PZN extraction dispatch in index.html after both validate and generate results: for `NTIN`, call `extractPznFromNtin(value)`; for `PPN`, call `extractPznFromPpn(value)`; for all other types, render no embedded-PZN block (depends on T018, T023, T029)

**Checkpoint**: All user stories are independently functional — NTIN/PPN validate and generate results now include a correctly-stated embedded PZN sub-result

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Visual states and behaviors that span all three user stories

- [x] T031 Style result banners in index.html: hidden by default, `.success` state for valid/generated-valid outcomes, `.error` state for invalid/generated-invalid outcomes, and sub-result variants for embedded PZN (success, error, warn)
- [x] T032 Style interactive controls in index.html: full-width selects, text inputs, and primary buttons; checkbox rows for optional generator behavior; inline secondary button styling for Copy
- [x] T033 Walk through the Delivery Checklist in [plan.md](plan.md) against the running index.html and confirm every item still holds
- [x] T034 Manually verify all constitution Article IV/V invariants (generator–validator duality and exact algorithm specifications) for PZN, GTIN, NTIN, PPN, and PCID
- [x] T035 Verify the corrected PPN algorithm (ISO/IEC 7064 MOD 97-10) in index.html against the worked example: PZN `12345678` → body `1112345678` → checksum `35` → PPN `111234567835` / `9N111234567835`; confirm `validatePpn` accepts both forms and `_decimalModulo(ppn, 97) === 1`
- [ ] T040 Walk through [quickstart.md](quickstart.md) Scenarios 1–4 against the running index.html: confirm the `9N` prefix row is visible only for `PPN`; confirm all four combinations of invalid × prefix generate with the correct label and prefix presence; confirm Validate and embedded-PZN inspection are unaffected by `includePrefix` (depends on T036–T039)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Story 1 (Phase 3)**: Depends on Foundational completion
- **User Story 2 (Phase 4)**: Depends on Foundational completion; independent of User Story 1
- **User Story 3 (Phase 5)**: Depends on Foundational completion AND on the validate/generate dispatch points introduced in Phase 3 (T018) and Phase 4 (T023)
- **Polish (Phase 6)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2) - No dependencies on other stories
- **User Story 2 (P2)**: Can start after Foundational (Phase 2) - No dependencies on other stories
- **User Story 3 (P3)**: Can start after Foundational (Phase 2), but its wiring task (T030) attaches to dispatch points from both US1 (T018) and US2 (T023)

### Within Each User Story

- Markup before behavior wiring
- Dispatch logic before result labeling
- Story complete before moving to next priority

### Parallel Opportunities

- Because the entire app is one `index.html` file, most tasks touch the same file and should be done sequentially to avoid merge conflicts, even where the underlying logic is independent
- Foundational algorithm functions (T006–T015) are logically independent of one another and could be implemented in any order, but should still be applied one at a time against index.html

### PPN Invalid × 9N-Prefix Increment (T036–T040)

- T036 (algorithm) and T037 (markup) touch different parts of index.html but should still be applied one at a time, consistent with the single-file convention above; both must land before T038/T039 (wiring), which must land before T040 (manual verification)
- This increment extends User Story 2 only; User Stories 1 and 3 are unaffected because `validatePpn`/`extractPznFromPpn` already tolerate an optional `9N` prefix

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational (CRITICAL - blocks all stories)
3. Complete Phase 3: User Story 1 (Validate)
4. **STOP and VALIDATE**: Test User Story 1 independently
5. Deploy/demo if ready

### Incremental Delivery

1. Complete Setup + Foundational → Foundation ready
2. Add User Story 1 (Validate) → Test independently → Deploy/Demo (MVP!)
3. Add User Story 2 (Generate) → Test independently → Deploy/Demo
4. Add User Story 3 (Embedded PZN Inspection) → Test independently → Deploy/Demo
5. Each story adds value without breaking previous stories

---

## Notes

- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- No [P] markers are used: the app is a single `index.html` file, so concurrent edits to different tasks would conflict
- Commit after each task or logical group
- Stop at any checkpoint to validate story independently
- The `index.html` file already contains a working implementation derived into this spec/plan/constitution set; these tasks describe the build-up used to verify and (re)produce that implementation
