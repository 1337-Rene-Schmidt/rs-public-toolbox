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
2. **Bidirectional, non-persistent.** The fragment drives initial/changed UI state (read), and the
   UI mirrors its current state back into the fragment as the user interacts (write), via
   `history.replaceState` — never `location.hash =` assignment, and never a network round-trip or
   storage write. This is not persistence: it disappears on manual URL edit/clear like any other
   navigation state (Constitution Article II).
3. **Silent fallback.** An absent, empty, or fully unrecognized fragment is functionally identical
   to no fragment at all — no error banner, no console warning surfaced to the user.
4. **Re-applied on change.** The same parse-and-apply routine (`applyAnchorState()`) runs once at
   load and again on every `hashchange` event, so browser back/forward navigation and manual
   fragment edits during the session take effect without a full page reload.
5. **Applied through existing code paths only.** `mode` reuses the tab-button click handler's
   activation logic; `package-identifier` reuses `syncGenOptions()`. No parallel rendering/state
   logic is introduced.
6. **Write-back does not create history entries or reload the page.** `writeAnchorState()` uses
   `history.replaceState(null, '', '#' + params.toString())`, which updates the address bar in
   place without adding a browser-history entry and without firing `hashchange` — so write-back
   can never re-trigger `applyAnchorState()` (no read/write feedback loop).
7. **Write-back preserves unrelated keys.** Any `key=value` pair in the fragment other than `mode`
   or `package-identifier` is carried over unchanged when the app updates its own keys.

## Write-back triggers

| User action | Effect on the fragment |
|-------------|-------------------------|
| Clicking a tab button | `mode` is set to the newly active tab's lowercase name (`generate` or `validate`). |
| Changing `#gen-type` | `package-identifier` is set to the newly selected type's lowercase code. |
| Changing `#val-type` | `package-identifier` is set to the newly selected type's lowercase code. |

Each write-back re-serializes the fragment from a `URLSearchParams` seeded with whatever was
already parsed from `location.hash`, so any other existing key is left untouched — only `mode`
and/or `package-identifier` are added or overwritten.

## Examples

| Fragment | Resulting state |
|----------|------------------|
| *(none)* | Generate tab active, `PZN` selected — unchanged default. |
| `#mode=validate` | Validate tab active, type selectors unchanged. |
| `#package-identifier=ppn` | Current tab unchanged, `PPN` selected in both type selectors. |
| `#mode=validate&package-identifier=ppn` | Validate tab active, `PPN` selected in both type selectors. |
| `#mode=GENERATE&package-identifier=PCID` | Generate tab active, `PCID` selected in both type selectors (case-insensitive). |
| `#mode=bogus&package-identifier=xyz` | No effect — defaults apply, no error shown. |
| `#foo=bar` | No effect — unrelated key ignored on read, but preserved verbatim on the next write-back. |

## Consumers

| Consumer | Notes |
|----------|-------|
| Anyone sharing/bookmarking a link into the app | The only real "external" consumer of this app's behavior; the address bar always reflects the current tab/type after any interaction. |
| `applyAnchorState()` (internal) | Sole reader of `location.hash`; sole caller of the tab-activation and `syncGenOptions()` logic for this purpose. |
| `writeAnchorState()` (internal) | Sole writer of `location.hash`, via `history.replaceState`; invoked from the tab-button click handler and from the `#gen-type`/`#val-type` `change` handlers. |

No other function signatures change. `generatePzn`/`generateNtin`/`generateGtin`/`generatePpn`/
`generatePcid` and their validators are entirely unaffected — this contract only concerns initial
(and hash-change-triggered) UI selection state, plus mirroring that state back into the fragment.
