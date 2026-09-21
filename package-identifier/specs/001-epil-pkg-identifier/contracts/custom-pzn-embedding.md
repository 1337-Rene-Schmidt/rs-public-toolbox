# Contract: Custom PZN Embedding for NTIN/PPN Generation

**Feature**: `001-epil-pkg-identifier`

This app has no network/API surface (Constitution: *Zero External Dependencies*, *No network
requests at runtime*). Its "public interface" is the set of pure functions in `index.html` that
are directly callable from the browser console (Constitution Principle I). This contract covers
the new `customPzn` parameter added to `generateNtin` and `generatePpn`, plus the UI-layer
validation guard that precedes calling them.

## `generateNtin(wantInvalid, embedPzn, customPzn = null)`

| Parameter | Type | Default | Effect |
|-----------|------|---------|--------|
| `wantInvalid` | `boolean` | — (required) | Unchanged from existing behavior. |
| `embedPzn` | `boolean` | — (required) | Unchanged from existing behavior. Ignored when `customPzn` is non-`null`. |
| `customPzn` | `string \| null` | `null` | When non-`null`, used verbatim as the 8-digit inner segment instead of a generated one. Caller MUST ensure `validatePzn(customPzn) === true` before passing a non-`null` value — this function does not re-validate it. |

**Returns**: `string` — 13-digit NTIN, unchanged shape.

## `generatePpn(wantInvalid, embedPzn, includePrefix, eplCompat, customPzn = null)`

| Parameter | Type | Default | Effect |
|-----------|------|---------|--------|
| `wantInvalid` | `boolean` | — (required) | Unchanged from existing behavior. |
| `embedPzn` | `boolean` | — (required) | Unchanged from existing behavior. Ignored when `customPzn` is non-`null`. |
| `includePrefix` | `boolean` | `true` | Unchanged from existing behavior. |
| `eplCompat` | `boolean` | `false` | Unchanged from existing behavior. |
| `customPzn` | `string \| null` | `null` | When non-`null`, used verbatim as the 8-digit inner segment instead of a generated one. Caller MUST ensure `validatePzn(customPzn) === true` before passing a non-`null` value — this function does not re-validate it. |

**Returns**: `string` — same shape as before (`9N` + 12 digits, or 12 digits).

**Postconditions (both functions)**:
1. When `customPzn` is non-`null`, `extractPznFromNtin(result)` / `extractPznFromPpn(result)`
   returns exactly `customPzn`.
2. When `customPzn` is `null`, behavior is byte-for-byte identical to the prior signature (as if
   the parameter did not exist).
3. `customPzn` never affects the outer checksum *computation* — it only changes which 8-digit
   string is folded into the body before the checksum is computed.

## UI-layer contract (not a function signature — the Generate button click handler)

Before dispatching to `generateNtin`/`generatePpn` when `type` is `'ntin'` or `'ppn'`:

1. Read `#gen-pzn-input`'s value and trim it → `customPznRaw`.
2. If `customPznRaw === ''`: pass `customPzn = null` to the generator (existing `embedPzn`-driven
   behavior applies, unchanged).
3. Else if `!validatePzn(customPznRaw)`: **do not call any generator function.** Render an error
   result (same result-banner mechanism used elsewhere) and stop — mirrors the existing empty-
   Validate-input guard pattern (`runValidation()`).
4. Else: pass `customPzn = customPznRaw` to the generator.

**This guard is the only place `customPzn` is validated.** `generateNtin`/`generatePpn` trust
their caller and do not re-validate — consistent with this app having no other internal input-
sanitization boundary between the UI and its pure algorithm functions.

## Consumers

| Caller | customPzn source |
|--------|-------------------|
| Generate panel (`btn-generate` click handler) | `#gen-pzn-input` value, trimmed; read only when `type ∈ {'ntin', 'ppn'}` |
| Browser console / manual testing | Omit for default (`null`) behavior, or pass a pre-validated PZN string explicitly |

No other function signatures change. `extractPznFromNtin`/`extractPznFromPpn` are unaffected
because the inner PZN segment occupies the same fixed-position slice regardless of its origin.
