# Contract: URL Anchor Deep Linking

**Feature**: `001-epil-pkg-identifier`

Unlike the other contracts in this folder, this one documents an **external-facing interface**:
the URL fragment (`#...`) syntax that anyone may use to construct a link into this app. It is the
one part of this single-file, no-backend tool that is consumed by something other than the
browser console — namely, whoever writes or shares a URL.

## Fragment syntax

The fragment is parsed as `key=value` pairs separated by `&`, identical to `URLSearchParams`
semantics (the implementation MUST use `new URLSearchParams(location.hash.replace(/^#/, ''))`,
not a bespoke parser):

```
#mode=<generate|validate>&package-identifier=<pzn|ntin|gtin|ppn|pcid>
```

Both keys are optional and order-independent. Either may appear alone. Values are matched
**case-insensitively**. Any other key is ignored. Percent-encoding is handled transparently by
`URLSearchParams`.

## `mode` key

| Value (case-insensitive) | Effect |
|---------------------------|--------|
| `generate` | Activates the Generate tab on load (and on `hashchange`). |
| `validate` | Activates the Validate tab on load (and on `hashchange`). |
| anything else, or key absent | No effect — the current/default tab (Generate) is left as-is. |

## `package-identifier` key

| Value (case-insensitive) | Effect |
|---------------------------|--------|
| `pzn`, `ntin`, `gtin`, `ppn`, or `pcid` | Sets **both** `#gen-type` and `#val-type` to this value and re-runs `syncGenOptions()`. |
| anything else, or key absent | No effect — both selects are left at their current/default value (`pzn`). |

## Behavioral guarantees

1. **No new modes or types.** This contract only pre-activates one of the two existing tabs and
   pre-selects one of the five existing identifier types — it cannot express any state that was
   not already reachable via manual clicks (Constitution Article VII).
2. **Read-only, non-persistent.** The app never writes to `location.hash`; there is no two-way
   binding. Nothing here introduces persisted state (Constitution Article II).
3. **Silent fallback.** An absent, empty, or fully unrecognized fragment is functionally identical
   to no fragment at all — no error banner, no console warning surfaced to the user.
4. **Re-applied on change.** The same parse-and-apply routine (`applyAnchorState()`) runs once at
   load and again on every `hashchange` event, so browser back/forward navigation and manual
   fragment edits during the session take effect without a full page reload.
5. **Applied through existing code paths only.** `mode` reuses the tab-button click handler's
   activation logic; `package-identifier` reuses `syncGenOptions()`. No parallel rendering/state
   logic is introduced.

## Examples

| Fragment | Resulting state |
|----------|------------------|
| *(none)* | Generate tab active, `PZN` selected — unchanged default. |
| `#mode=validate` | Validate tab active, type selectors unchanged. |
| `#package-identifier=ppn` | Current tab unchanged, `PPN` selected in both type selectors. |
| `#mode=validate&package-identifier=ppn` | Validate tab active, `PPN` selected in both type selectors. |
| `#mode=GENERATE&package-identifier=PCID` | Generate tab active, `PCID` selected in both type selectors (case-insensitive). |
| `#mode=bogus&package-identifier=xyz` | No effect — defaults apply, no error shown. |
| `#foo=bar` | No effect — unrelated key ignored. |

## Consumers

| Consumer | Notes |
|----------|-------|
| Anyone sharing/bookmarking a link into the app | The only real "external" consumer of this app's behavior. |
| `applyAnchorState()` (internal) | Sole reader of `location.hash`; sole caller of the tab-activation and `syncGenOptions()` logic for this purpose. |

No other function signatures change. `generatePzn`/`generateNtin`/`generateGtin`/`generatePpn`/
`generatePcid` and their validators are entirely unaffected — this contract only concerns initial
(and hash-change-triggered) UI selection state.
