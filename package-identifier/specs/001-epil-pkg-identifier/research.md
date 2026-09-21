# Research: PPN Generation — Independent Invalid / 9N-Prefix Options

**Feature**: `001-epil-pkg-identifier`
**Scope of this research pass**: FR-039, FR-040, SC-009 (user-selectable `9N` prefix on generated PPN values, orthogonal to the existing invalid-generation option)

There were no open `NEEDS CLARIFICATION` markers in Technical Context for this increment — the
surrounding stack, hosting model, and algorithm source are already fixed by the existing feature
(single-file `index.html`, no build step, no external dependencies). The only design questions
were how to expose and wire the new option.

## Decision: Add a third boolean parameter to `generatePpn`

**Decision**: Extend `generatePpn(wantInvalid, embedPzn)` to `generatePpn(wantInvalid, embedPzn,
includePrefix = true)`. The function still always computes the 12-digit body and checksum the
same way; `includePrefix` only controls whether the returned string is prefixed with `9N`.

**Rationale**:
- Keeps the checksum computation (`_ppnCheck` / `_decimalModulo`) completely untouched — prefix
  presence has never been part of the checksum, so it must not influence `wantInvalid` corruption
  logic either.
- Matches the existing pattern used for `embedPzn` (an independent boolean flag alongside
  `wantInvalid`), so the change is consistent with the rest of the algorithm library rather than
  introducing a new options-object convention for only one function.
- `validatePpn` and `extractPznFromPpn` already strip an optional `9N` prefix, so no changes are
  required there — the new option is purely a generation-time concern.
- Defaulting `includePrefix` to `true` preserves current behavior for any existing call site that
  doesn't pass the new argument (none currently exist outside the Generate panel, but this keeps
  the function safe to call from the browser console per Constitution Principle I/II).

**Alternatives considered**:
- *Separate `generatePpnWithoutPrefix()` function*: rejected — duplicates the checksum logic and
  violates the "four-step pattern" (generate/validate/switch/switch) described in the
  constitution's Development Workflow section; a single parameterized function is simpler to keep
  in sync.
- *Options object (`generatePpn({ wantInvalid, embedPzn, includePrefix })`)*: rejected for this
  increment — would require touching every existing call site and diverges from the positional-
  argument style already used by `generateNtin`/`generatePzn`/`generateGtin` in this codebase.
  Not worth the churn for one additional boolean.

## Decision: UI control is a third independent checkbox, not a combined dropdown

**Decision**: Add a `9N` prefix checkbox (`row-ppn-prefix` / `gen-ppn-prefix`), shown only when
`PPN` is selected, defaulting to checked. It is independent of (not mutually exclusive with) the
existing invalid-generation checkbox.

**Rationale**:
- FR-039 explicitly requires the option to be independent of invalid-generation, so a single
  checkbox toggle is the most direct UI expression of "orthogonal boolean flag" and matches the
  existing checkbox-row visual/interaction pattern (`row-embed-pzn`, `row-invalid`).
- Avoids introducing a 4-way `<select>` (valid+prefixed / valid+unprefixed / invalid+prefixed /
  invalid+unprefixed) which would be a new control pattern not used anywhere else in the app and
  would make the existing invalid-generation checkbox partially redundant.

**Alternatives considered**:
- *Single 4-option dropdown*: rejected as a new UI pattern inconsistent with the rest of the
  Generate panel and harder to scan than two independent checkboxes.
- *Always strip the prefix and show it only as a separate "copy without prefix" button*: rejected
  — does not satisfy FR-039's requirement that the *generated* value itself omit the prefix when
  requested (e.g. for pasting directly into a system that expects a bare 12-digit PPN).

## Open questions

None remaining for this increment.
