# Data Model: Package Identifier Tool

**Feature**: `001-epil-pkg-identifier`

This app has no persisted data store (Constitution: *No persistence*). "Data model" here means the
in-memory value objects and UI state read/written by the Generate and Validate flows. This file
documents the full current model; the PPN-prefix addition is called out where it changes existing
state.

## Generate Panel State

| Field | Type | Source | Notes |
|-------|------|--------|-------|
| `type` | `'pzn' \| 'ntin' \| 'gtin' \| 'ppn' \| 'pcid'` | `#gen-type` select | Drives which generator function is dispatched and which option rows are visible. |
| `wantInvalid` | `boolean` | `#gen-invalid` checkbox | Disabled and forced `false` when `type === 'pcid'`. |
| `embedPzn` | `boolean` | `#gen-embed-pzn` checkbox | Only shown/read when `type` is `'ntin'` or `'ppn'`. Ignored when `customPzn` is non-blank (see below). |
| `includePrefix` | `boolean` | `#gen-ppn-prefix` checkbox *(new)* | Only shown/read when `type === 'ppn'`. Defaults to `true` (checked). Independent of `wantInvalid`. |
| `eplCompat` | `boolean` | `#gen-epl-compat` checkbox *(new)* | Only shown/read when `type === 'ppn'`. Defaults to `false` (unchecked). Independent of `wantInvalid`, `includePrefix`, and `embedPzn`. When `true`, the outer PPN checksum is computed with the EPL-legacy formula instead of the standard ISO/IEC 7064 formula. |
| `customPzn` | `string \| null` | `#gen-pzn-input` text field *(new)* | Only shown/read when `type` is `'ntin'` or `'ppn'`. Trimmed; blank/whitespace-only is treated as `null`. When non-null, MUST be a structurally valid PZN (checked with `validatePzn` before generation); an invalid non-blank value blocks generation and shows an error result instead. Takes precedence over `embedPzn` when non-null. |

State transitions on `type` change (`syncGenOptions()`):
- `row-embed-pzn` visibility ⇄ `type ∈ {ntin, ppn}`
- `row-invalid` enabled ⇄ `type !== 'pcid'`
- `row-ppn-prefix` visibility ⇄ `type === 'ppn'` *(new)*
- `row-epl-compat` visibility ⇄ `type === 'ppn'` *(new)*
- `row-pzn-input` visibility ⇄ `type ∈ {ntin, ppn}` *(new)*

## Generated Identifier Value Objects

These are plain strings returned by the generator functions — there is no wrapping object.

| Type | Shape | Checksum scheme |
|------|-------|------------------|
| PZN | 8 digits | MOD-11, weights 1–7 |
| GTIN | 14 digits | GS1 MOD-10, alternating weights 3/1 |
| NTIN | 13 digits (`4150` + 8-digit inner + 1-digit check) | delegates to GTIN MOD-10 |
| **PPN** | `("9N" \| "") + "11" + 8-digit inner + 2-digit check` | ISO/IEC 7064 MOD 97-10 over `"11" + inner"` (standard), or the EPL-legacy formula over the same body when `eplCompat` is true |
| PCID | RFC 4122 UUID v1–v5 string | n/a (no checksum) |

**PPN shape change for this increment**: the leading `"9N"` segment becomes conditional on the new
`includePrefix` flag instead of always being present. The 12-digit body (`"11" + inner + check`)
and its checksum computation are unchanged — `includePrefix` only affects string concatenation at
the very end of `generatePpn`, after the checksum has already been computed.

**PPN checksum formula selection (EPL v2.3.x compatibility mode)**: `generatePpn`'s 4th parameter,
`eplCompat`, selects which of two checksum functions computes the 2-digit check: `_ppnCheck`
(standard, default) or `_ppnCheckEplLegacy` (mirrors the EPL backend's current formula). Both
produce a value in the same `[00, 96]` range requiring the same zero-padding. `validatePpn`
accepts a value if *either* formula's checksum matches the trailing 2 digits, so the caller does
not need to know which formula produced a given value when validating it.

**PPN EPL v2.3.x compatibility caveat note** *(new)*: when the Generate panel renders a result for
`type === 'ppn'` with `eplCompat === true` and the result is valid, an additional, distinctly
styled caveat line is shown beneath the `Generated valid PPN` success message stating the value is
valid only under EPL v2.3.x compatibility mode and uses a non-standard checksum. This is presentation
state only — it does not change `generatePpn`'s return value, `validatePpn`'s behavior, or any
other field in this data model.

## Embedded PZN Sub-Result

| Field | Type | Notes |
|-------|------|-------|
| `extractedPzn` | `string \| null` | `null` means "could not be extracted". |
| `isValid` | `boolean` | Only meaningful when `extractedPzn !== null`. |

Unaffected by this increment — `extractPznFromPpn` already strips an optional `9N` prefix before
slicing, so it behaves identically whether the outer PPN was generated with or without the prefix.

## Custom PZN Embedding

When `customPzn` is non-null, `generateNtin`/`generatePpn` use it verbatim as `inner` instead of
calling `generatePzn(false)` or generating a random 8-digit number. This changes only the *source*
of `inner`; the outer NTIN/PPN checksum computation is otherwise identical. `extractPznFromNtin`/
`extractPznFromPpn` then return exactly `customPzn` when applied to the result, since `inner`
occupies the same fixed-position slice regardless of its origin.

Validation of `customPzn` (via `validatePzn`) happens once, at the UI dispatch layer, before the
generator function is called — not inside `generateNtin`/`generatePpn` themselves — consistent with
how the existing empty-Validate-input guard is handled outside `validatePzn`/etc.

## URL Anchor State *(new)*

Ephemeral state derived from and mirrored to `location.hash`. Read at script load and re-derived
on every `hashchange` event; written back to the fragment (via `history.replaceState`, which does
not fire `hashchange`) whenever the user switches tabs or changes an identifier-type selector. Not
persisted anywhere (no storage, no server round-trip) — it is simply an alternate way of setting,
and a mirror of, the same `type`/tab state that a manual click would set.

| Field | Type | Source | Notes |
|-------|------|--------|-------|
| `mode` | `'generate' \| 'validate' \| null` | `mode` key of `new URLSearchParams(location.hash.slice(1))`, case-insensitive | **Read**: when recognized, activates the matching tab via the existing tab-click code path. `null`/unrecognized leaves the current (default `generate`) tab untouched. **Write**: set to the newly active tab's lowercase name whenever the user clicks a tab button. |
| `packageIdentifier` | `'pzn' \| 'ntin' \| 'gtin' \| 'ppn' \| 'pcid' \| null` | `package-identifier` key of the same parsed fragment, case-insensitive | **Read**: when recognized, sets **both** `#gen-type` and `#val-type` to this value and calls `syncGenOptions()`. `null`/unrecognized leaves both selects at their current (default `pzn`) value. **Write**: set to whichever type selector (`#gen-type` or `#val-type`) the user just changed, using its new lowercase value. |

This state is applied through the exact same code paths as manual interaction (tab-button click
handler, `syncGenOptions()`) — there is no separate rendering logic for anchor-derived state, and
no field in the Generate Panel State or Validate panel table above is otherwise affected.

**Write-back mechanism** *(new)*: `writeAnchorState()` reads the currently active tab and the
just-changed type selector, merges `mode`/`package-identifier` into the existing parsed
`URLSearchParams` (preserving any other keys already present), and calls
`history.replaceState(null, '', '#' + params.toString())`. Because `replaceState` does not fire
`hashchange`, calling it from the tab-click/type-`change` handlers cannot re-trigger
`applyAnchorState()` — read and write paths do not feed back into each other (FR-057).


