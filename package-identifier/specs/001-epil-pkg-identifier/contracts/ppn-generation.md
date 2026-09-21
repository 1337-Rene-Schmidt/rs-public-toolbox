# Contract: PPN Generation Function

**Feature**: `001-epil-pkg-identifier`

This app has no network/API surface (Constitution: *Zero External Dependencies*, *No network
requests at runtime*). Its "public interface" is the set of pure functions in `index.html` that
are directly callable from the browser console (Constitution Principle I). This contract covers
the one function signature changed by this increment.

> **Note**: A later increment adds a 5th parameter, `customPzn`, to `generatePpn`. See
> [contracts/custom-pzn-embedding.md](custom-pzn-embedding.md) for that parameter and the
> function's current full signature.

## `generatePpn(wantInvalid, embedPzn, includePrefix, eplCompat)`

| Parameter | Type | Default | Effect |
|-----------|------|---------|--------|
| `wantInvalid` | `boolean` | — (required) | When `true`, the 2-digit checksum is corrupted by `+1 mod 97` so the result fails `validatePpn` under both recognized checksum formulas. |
| `embedPzn` | `boolean` | — (required) | When `true`, the 8-digit inner segment is a freshly generated valid PZN; otherwise a random 8-digit number. |
| `includePrefix` | `boolean` | `true` | When `true`, the returned string is prefixed with `9N`; when `false`, the returned string is the bare 12-digit body. |
| `eplCompat` | `boolean` | `false` | When `true`, the 2-digit checksum is computed with `_ppnCheckEplLegacy` (mirrors the EPL backend's current formula) instead of `_ppnCheck` (standard ISO/IEC 7064 MOD 97-10). |

**Returns**: `string` — either `9N` + 12 digits (`includePrefix === true`) or exactly 12 digits
(`includePrefix === false`).

**Postconditions**:
1. When `eplCompat === false`: `_decimalModulo(<12-digit body>, 97) === 1` if and only if
   `wantInvalid === false`.
2. When `eplCompat === true` and `wantInvalid === false`: the trailing 2-digit checksum equals
   `_ppnCheckEplLegacy(<first 10 digits>)`, zero-padded to 2 digits.
3. Regardless of `eplCompat`, `validatePpn(result) === !wantInvalid` — i.e. a value generated with
   `wantInvalid === true` fails validation under *both* checksum formulas, not only the one used to
   compute it before corruption.
4. The presence or absence of the `9N` prefix has no effect on postconditions 1–3 — validating the
   value via `validatePpn` (which strips an optional `9N` prefix before checking) MUST produce the
   same valid/invalid result regardless of `includePrefix`.
5. `extractPznFromPpn(result)` returns the same 8-digit inner segment regardless of `includePrefix`
   or `eplCompat`.

**Backward compatibility**: existing call sites that omit the third and/or fourth argument keep
the current standard behavior (`9N` prefix present, standard ISO/IEC 7064 checksum), since both
`includePrefix` and `eplCompat` default to their current-behavior values (`true` and `false`
respectively).

## `validatePpn(value)`

Accepts a value as structurally valid PPN when, after stripping an optional `9N` prefix and
confirming exactly 12 digits, the trailing 2-digit checksum matches *either*:
- the standard ISO/IEC 7064 MOD 97-10 formula (`_decimalModulo(value, 97) === 1`), or
- the EPL-legacy formula (`_ppnCheckEplLegacy` over the first 10 digits).

This dual acceptance lets a value generated with `eplCompat === true` validate successfully when
later pasted into Validate mode, without requiring the caller to know which formula produced it.

## `_ppnCheckEplLegacy(body10)`

Reproduces the EPL backend's current (non-standard) checksum formula for interoperability:
sum each of the 10 body-digit characters' `charCodeAt(0)` (ASCII code, not numeric digit value)
weighted by position (`2` through `11` in order), then take `% 97`. Zero-pad the result to 2
digits. Must cite its source in a code comment, e.g.
`// Source: PpnValidationService.php::calculateCheckDigit (legacy formula, pre-fix)`, per
Constitution Article III.

**This function exists solely to accommodate a known backend defect.** It is not a valid
implementation of the IFA PPN standard and MUST NOT become the default checksum function.

## Consumers

| Caller | includePrefix source | eplCompat source |
|--------|----------------------|-------------------|
| Generate panel (`btn-generate` click handler) | `#gen-ppn-prefix` checkbox, read only when `type === 'ppn'` | `#gen-epl-compat` checkbox, read only when `type === 'ppn'` |
| Browser console / manual testing | Omit for default (`true`) behavior, or pass `false` explicitly | Omit for default (`false`) behavior, or pass `true` explicitly |

No other function signatures change. `extractPznFromPpn` is unaffected because it already treats
the `9N` prefix as optional on input, and the inner PZN segment's position is unaffected by which
checksum formula was used for the outer PPN.

