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
