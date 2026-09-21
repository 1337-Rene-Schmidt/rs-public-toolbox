# Contract: PPN Generation Function

**Feature**: `001-epil-pkg-identifier`

This app has no network/API surface (Constitution: *Zero External Dependencies*, *No network
requests at runtime*). Its "public interface" is the set of pure functions in `index.html` that
are directly callable from the browser console (Constitution Principle I). This contract covers
the one function signature changed by this increment.

## `generatePpn(wantInvalid, embedPzn, includePrefix)`

| Parameter | Type | Default | Effect |
|-----------|------|---------|--------|
| `wantInvalid` | `boolean` | — (required) | When `true`, the 2-digit checksum is corrupted by `+1 mod 97` so the result fails `validatePpn`. |
| `embedPzn` | `boolean` | — (required) | When `true`, the 8-digit inner segment is a freshly generated valid PZN; otherwise a random 8-digit number. |
| `includePrefix` | `boolean` | `true` | When `true`, the returned string is prefixed with `9N`; when `false`, the returned string is the bare 12-digit body. |

**Returns**: `string` — either `9N` + 12 digits (`includePrefix === true`) or exactly 12 digits
(`includePrefix === false`).

**Postconditions**:
1. `_decimalModulo(<12-digit body>, 97) === 1` if and only if `wantInvalid === false`.
2. The presence or absence of the `9N` prefix has no effect on postcondition 1 — validating the
   value via `validatePpn` (which strips an optional `9N` prefix before checking) MUST produce the
   same valid/invalid result regardless of `includePrefix`.
3. `extractPznFromPpn(result)` returns the same 8-digit inner segment regardless of `includePrefix`.

**Backward compatibility**: existing call sites that omit the third argument keep the current
behavior (`9N` prefix always present), since `includePrefix` defaults to `true`.

## Consumers

| Caller | includePrefix source |
|--------|----------------------|
| Generate panel (`btn-generate` click handler) | `#gen-ppn-prefix` checkbox, read only when `type === 'ppn'` |
| Browser console / manual testing | Omit for default (`true`) behavior, or pass `false` explicitly |

No other function signatures change. `validatePpn` and `extractPznFromPpn` are unaffected because
they already treat the `9N` prefix as optional on input.
