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
  the function safe to call from the browser console per Constitution Article I §2).

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

---

# Research: PPN Generation — EPL v2.3.x Compatibility Mode

**Scope of this research pass**: FR-041–FR-045, SC-010 (an opt-in generation-time option that
produces PPN check digits accepted by the current EPL backend, plus explanatory help text)

## Background

Cross-checking `index.html` against the EPL backend's `PpnValidationService.php` revealed that
`calculateCheckDigit()` does not implement ISO/IEC 7064 MOD 97-10 (the standard cited in this
project's constitution, Article IV). It instead sums each body digit's `ord()` (ASCII code, not
its numeric value) weighted by position (`2..11`), then takes `% 97`. This is a backend defect,
not an alternate valid standard — but until the backend is fixed, PPNs generated with the correct
standard formula are rejected by EPL with "invalid PPN format". A backend fix has been proposed
separately; this research covers a frontend-side interim accommodation.

## Decision: Add a second checksum function and a 4th boolean parameter, not a mode-specific clone

**Decision**: Add `_ppnCheckEplLegacy(body10)`, a faithful reproduction of the backend's current
(buggy) formula, alongside the existing `_ppnCheck(body10)`. Extend
`generatePpn(wantInvalid, embedPzn, includePrefix)` to
`generatePpn(wantInvalid, embedPzn, includePrefix, eplCompat = false)`, which selects which
checksum function computes the check digit. `validatePpn` is extended to accept a value under
*either* formula.

**Rationale**:
- Mirrors the same additive-boolean-parameter pattern already established for `includePrefix`
  (see the PPN-prefix research above) — consistent with this codebase's existing conventions
  rather than introducing a new options-object or a parallel `generatePpnEplCompat()` function
  that would duplicate the inner/embed-PZN logic.
- Keeping both checksum functions as named, independently testable functions (rather than
  branching arithmetic inline) satisfies Constitution Article III's requirement that the
  reference implementation be cited in a code comment at the point of implementation — the
  legacy formula's comment cites `PpnValidationService.php::calculateCheckDigit`.
- `validatePpn` must accept both formulas, because a caller checking a value in Validate mode
  cannot know which mode it was generated with — the two checksum spaces are large enough
  (1-in-97 collision chance per formula) that trying both is the only way to make "generate, then
  paste into Validate" work for EPL-compatible values without adding a second, redundant
  "EPL v2.3.x compatibility mode" toggle to the Validate panel (out of scope: FR-041 only requires the
  option in Generate mode).
- Defaulting `eplCompat` to `false` preserves the existing standard behavior for every current
  call site (Generate panel default, console use, and the prior PPN-prefix increment's tests).

**Consequence for invalid generation**: previously, the do-while loop in `generatePpn` only
re-checked validity when generating a *valid* value (`while (!wantInvalid && !validatePpn(ppn))`).
Because `validatePpn` now accepts two formulas, a value corrupted under one formula could
coincidentally still satisfy the other (≈1-in-97 chance). The loop condition is generalized to
`while (wantInvalid ? validatePpn(ppn) : !validatePpn(ppn))`, so invalid generation regenerates
until the result fails *both* formulas. This is a strict robustness improvement with no behavior
change for existing non-`eplCompat` call sites (the standard formula was already deterministic).

**Alternatives considered**:
- *Separate `generatePpnEplCompat()` function*: rejected — duplicates the inner-PZN/embed logic
  and violates the same "four-step pattern" reasoning already used to reject a separate
  prefix-only function in the previous increment.
- *A Validate-mode "EPL v2.3.x compatibility mode" toggle instead of dual-formula acceptance*: rejected
  — the user only requested the option for Generate mode; requiring a matching toggle in Validate
  would force users to know in advance which formula produced a given value, which defeats the
  purpose of "compatibility" (accepting it transparently).
- *Only mutate `_ppnCheck` to always match the backend's current (buggy) formula*: rejected —
  would abandon the standard ISO/IEC 7064 formula entirely, violating Constitution Article IV
  (Standards-First) and silently regressing the correct behavior most callers rely on.

## Decision: Tooltip via native `title` attribute + always-visible `<small>` hint, no JS popover

**Decision**: The **EPL v2.3.x compatibility mode** row includes a small inline help marker with a
`title` attribute (native browser tooltip on hover/focus for pointer users) and an always-visible
`<small class="hint">` line directly under the row containing the same explanatory text.

**Rationale**:
- Touch-friendly, mobile-accessible controls are a general design goal for this app (not a numbered
  constitution rule); a hover-only tooltip is not reliably reachable by touch, so the explanation
  must also be visible without any interaction.
- Constitution Article I §3 (no external resources loaded at any time) rules out a JS tooltip/popover
  library; the `title` attribute requires no script and degrades gracefully (Operational
  Constraints: graceful degradation) if unsupported.
- This mirrors the plain, semantic-HTML-first style already used throughout `index.html` (no
  custom widget components).

**Alternatives considered**:
- *Custom JS-driven popover triggered by click/tap*: rejected — adds interaction-handling code and
  a new UI pattern for a single help string; unnecessary given the always-visible hint already
  satisfies the accessibility requirement.
- *`title` attribute only, no visible hint*: rejected — fails Article V for touch-only users who
  cannot trigger a native tooltip via hover.

## Open questions

None remaining for this increment.

---

# Research: NTIN/PPN Generation — Custom PZN Embedding

**Scope of this research pass**: FR-046–FR-049, SC-011 (an optional free-text field letting the
user specify the exact PZN to embed in a generated `NTIN`/`PPN`, in place of the automatic
random-PZN behavior already governed by the embed-a-valid-PZN checkbox)

## Background

The existing `embedPzn` boolean only chooses *how* the inner 8-digit segment is produced (a
freshly generated valid PZN, or a random 8-digit number) — it never lets the caller supply a
*specific* PZN value. Some users want to generate an `NTIN`/`PPN` around a PZN they already have
(e.g. a real product's PZN) rather than an arbitrary one.

## Decision: A new optional free-text field, validated at the UI layer before generation, taking precedence over `embedPzn` when non-blank

**Decision**: Add a text input (`#gen-pzn-input`, shown only for `NTIN`/`PPN`) alongside the
existing embed-a-valid-PZN checkbox. On Generate:
1. Read and trim the field's value as `customPzn`.
2. If `customPzn` is non-blank and `!validatePzn(customPzn)`: show an error result (mirroring the
   existing empty-Validate-input guard pattern) and do not call any generator function.
3. Otherwise, pass `customPzn` (or `null` if blank) as a new trailing argument to `generateNtin`/
   `generatePpn`; when non-null, the generator uses it verbatim as `inner`, ignoring `embedPzn`.
   When `null`, behavior is byte-for-byte identical to today (governed by `embedPzn`).

**Rationale**:
- Keeps `validatePzn` as the single source of truth for "is this a valid PZN" — no new validation
  logic is introduced, satisfying Constitution Article V (no approximated/duplicated checksum
  rules).
- Validating *before* calling the generator functions mirrors the codebase's existing convention
  of doing user-input error handling in the UI dispatch layer (see the empty-input guard in
  `runValidation()`) rather than inside the pure algorithm functions — keeps `generateNtin`/
  `generatePpn` simple and directly console-callable per Constitution Article I §2.
- Making `customPzn` take precedence over `embedPzn` (rather than requiring them to be mutually
  exclusive in the UI) avoids adding new interaction states (e.g. disabling the checkbox); a
  blank field is an unambiguous "not provided" signal, so precedence is simple to reason about
  and document (FR-048/FR-049).
- Adding `customPzn` as a new *trailing* parameter (rather than replacing `embedPzn`) preserves
  every existing call site's behavior with no change (default `null`), consistent with how
  `includePrefix` and `eplCompat` were added in prior increments.

**Alternatives considered**:
- *Replace the embed-a-valid-PZN checkbox with the text field entirely*: rejected — removes the
  existing "quickly embed *some* valid PZN without typing one" behavior (FR-016/FR-027/FR-029),
  which is still useful and already covered by existing tests/scenarios.
- *Validate inside `generateNtin`/`generatePpn` and throw on an invalid `customPzn`*: rejected —
  inconsistent with how this codebase handles user-input errors (UI-layer guard + rendered error
  result, not exceptions from pure algorithm functions); would also require every console caller
  to handle a thrown error just to generate a value.
- *Disable/clear the embed-a-valid-PZN checkbox when the field is non-blank*: rejected as
  unnecessary UI-state complexity — the field's precedence over the checkbox is simple enough to
  document and does not require mutually-exclusive widget states.

## Open questions

None remaining for this increment.

---

# Research: PPN Generation — EPL v2.3.x Compatibility Caveat Note

**Scope of this research pass**: FR-042a (a distinct caveat note shown in the generation result
area whenever EPL v2.3.x compatibility mode produces a structurally valid PPN)

## Background

FR-042 already requires a tooltip/hint on the **EPL v2.3.x compatibility mode** *option* explaining
its non-standard, opt-in nature before generation. That explanation is easy to miss (it lives next
to the checkbox, not next to the result). Clarification confirmed the result area itself should
also carry a caveat, so a user reading only the result still learns the value is non-standard.

## Decision: A distinct caveat sub-line beneath the success message, not merged into it

**Decision**: Keep the existing `Generated valid PPN` success message unchanged. When
`eplCompat === true` and the generated result is valid, render an additional, visually distinct
line (`.caveat` class, not `.success`/`.error`) reading:
`Note: valid only in EPL v2.3.x compatibility mode (non-standard checksum).`

**Rationale**:
- Preserves Constitution Article VII §24 (every operation shows an unambiguous success/failure
  outcome) — the caveat is additive context, not a redefinition of the pass/fail state.
- Mirrors the existing pattern for the embedded-PZN sub-result (Article VI): a secondary,
  independently-styled block alongside the primary result rather than concatenated text.
- Keeping the message text `Generated valid PPN` byte-for-byte unchanged avoids breaking any
  existing string-matching test/documentation reference to that exact success message.

**Alternatives considered**:
- *Append the caveat into the same success message string*: rejected per clarification — conflates
  two concerns (pass/fail outcome vs. interoperability caveat) into one string, and would break the
  existing exact-match references to `Generated valid PPN` in quickstart.md and prior tests.
- *Only show the caveat via the existing tooltip/hint on the checkbox*: rejected — does not satisfy
  the request, since a user who already generated a value and is looking only at the result area
  would not see it.

## Open questions

None remaining for this increment.

---

# Research: URL Anchor Deep Linking for Mode and Identifier Type

**Scope of this research pass**: FR-050–FR-053, SC-012 (deep-linking the app's initial tab and
identifier-type selection via the URL fragment)

## Background

Users want to share/bookmark a link that opens the tool already in a specific mode (Generate or
Validate) with a specific identifier type preselected, instead of always landing on the default
Generate/PZN view. The request specifies a `key=value` fragment syntax: `#mode=generate` /
`#mode=validate`, and `#package-identifier=<type>`, combinable in one fragment.

## Decision: Parse the fragment with `URLSearchParams`, applied via the existing tab-switch and `syncGenOptions()` code paths

**Decision**: On script load (and again on every `hashchange` event), read
`new URLSearchParams(location.hash.replace(/^#/, ''))`:
1. If the `mode` param case-insensitively equals `generate` or `validate`, invoke the exact same
   logic the tab-button click handler uses to activate that tab (not a parallel implementation).
2. If the `package-identifier` param case-insensitively matches `pzn`, `ntin`, `gtin`, `ppn`, or
   `pcid`, set `#gen-type.value` and `#val-type.value` to that value and call `syncGenOptions()` so
   option-row visibility (embed-PZN, `9N` prefix, EPL compat, PZN-to-embed) stays correct for the
   preselected type.
3. Any other key, or an unrecognized value for a recognized key, is ignored silently — the
   corresponding default (Generate tab active, `PZN` selected) is left untouched.

**Rationale**:
- `URLSearchParams` is a native, zero-dependency browser API already `key=value`/`&`-oriented,
  matching the user's requested syntax exactly with no bespoke parsing code — satisfies
  Constitution Article I (single-file delivery, no new files) and keeps the parser trivially
  correct for edge cases (extra `&`, missing `=`, URL-encoded values).
- Reusing the existing tab-activation and `syncGenOptions()` code paths (rather than writing a
  second, parallel state-setting routine) guarantees the anchor-driven state is indistinguishable
  from a manual click, so no new UI states/bugs are introduced.
- Applying `package-identifier` to *both* `#gen-type` and `#val-type` avoids ambiguity about which
  panel a single URL parameter should target, and matches the natural reading of "preselect this
  identifier type" regardless of which tab ends up active.
- Falling back silently (no error banner) for unrecognized values matches Constitution Article VII
  §24's scope — that rule governs generate/validate *operations*, not passive page-load state; a
  malformed or foreign URL fragment is not a user-initiated operation.
- Re-running the same logic on `hashchange` (FR-053) costs one `addEventListener` call and covers
  browser back/forward navigation and manual fragment edits for free, without introducing any
  two-way binding (the app never *writes* to `location.hash`).

**Alternatives considered**:
- *A bespoke `#/generate/ppn`-style path syntax*: rejected — the user explicitly requested
  `key=value` fragment syntax (`#mode=generate`, `#package-identifier=ppn`), and `URLSearchParams`
  already implements that syntax correctly.
- *Query string (`?mode=...`) instead of the fragment (`#mode=...`)*: rejected — the user explicitly
  asked for URL *anchors*; the fragment also avoids any possibility of being sent to a server if the
  file is ever served from one, which is marginally more consistent with Article II (client-only
  computation), though both forms are equally valid for a `file://`-served single-file app.
- *Writing the current mode/type back into `location.hash` as the user interacts (two-way binding)*:
  rejected — not requested; the user only asked for the anchor to *drive* initial/changed state, not
  for the app to rewrite the URL on every click. Adding write-back would be scope creep beyond
  FR-050–FR-053.
- *A single combined key (e.g. `#state=validate,ppn`)*: rejected — diverges from the two distinct
  keys (`mode`, `package-identifier`) the user explicitly specified.

## Open questions

None remaining for this increment.



