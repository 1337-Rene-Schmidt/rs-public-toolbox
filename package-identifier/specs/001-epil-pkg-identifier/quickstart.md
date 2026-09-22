# Quickstart: Validate PPN Invalid / 9N-Prefix / EPL Compatibility / Custom PZN Embedding / URL Anchor Options

**Feature**: `001-epil-pkg-identifier` — validates FR-039, FR-040, FR-041–FR-045, FR-042a, FR-046–FR-049, FR-050–FR-057, SC-009, SC-010, SC-011, SC-012, SC-014

## Prerequisites

- No build step. Open [`index.html`](../../index.html) directly in a browser
  (`file://.../index.html`), or serve the repository root with any static file server.
- No login, network access, or setup required.

## Scenario 1 — Prefix option only visible for PPN

1. On the **Generate** tab, select each type in turn: `PZN`, `NTIN`, `GTIN`, `PCID`.
2. **Expect**: no `9N` prefix checkbox is visible for any of these types.
3. Select `PPN`.
4. **Expect**: a `9N` prefix checkbox is visible and checked by default.

## Scenario 2 — Four independent combinations

For each combination of the **invalid** checkbox and the **9N prefix** checkbox below, select
`PPN`, set both checkboxes accordingly, click **Generate**, and check the result:

| Invalid | 9N prefix | Expected value shape | Expected result |
|---------|-----------|------------------------|------------------|
| off | on | `9N` + 12 digits | success / "Generated valid PPN" |
| off | off | exactly 12 digits (no `9N`) | success / "Generated valid PPN" |
| on | on | `9N` + 12 digits | error / "Generated invalid PPN — check digit deliberately corrupted" |
| on | off | exactly 12 digits (no `9N`) | error / "Generated invalid PPN — check digit deliberately corrupted" |

**Expect**: toggling the prefix checkbox never changes whether the result is labelled valid or
invalid — only whether `9N` appears at the start of the displayed value.

## Scenario 3 — Cross-check with Validate tab

1. Generate a `PPN` with **9N prefix** off and **invalid** off. Copy the displayed (unprefixed)
   value.
2. Switch to the **Validate** tab, select `PPN`, paste the value, and press **Validate**.
3. **Expect**: `✓ PPN is structurally valid`.
4. Repeat with the **invalid** checkbox on during generation.
5. **Expect**: `✗ PPN is structurally invalid`.

## Scenario 4 — Embedded PZN unaffected by prefix

1. Generate a `PPN` with **Embed a valid PZN** checked and **9N prefix** off.
2. **Expect**: the embedded-PZN sub-result still renders and shows the embedded PZN as valid,
   identical to when **9N prefix** is on.

## Manual console cross-check (optional)

Open the browser devtools console on the loaded page and run:

```js
generatePpn(false, true, true).startsWith('9N')   // → true
generatePpn(false, true, false).startsWith('9N')  // → false
validatePpn(generatePpn(false, true, false))      // → true
validatePpn(generatePpn(true, true, false))       // → false
```

## Scenario 5 — EPL v2.3.x compatibility mode is visible only for PPN, with a tooltip

1. On the **Generate** tab, select each type in turn: `PZN`, `NTIN`, `GTIN`, `PCID`.
2. **Expect**: no **EPL v2.3.x compatibility mode** checkbox is visible for any of these types.
3. Select `PPN`.
4. **Expect**: an **EPL v2.3.x compatibility mode** checkbox is visible and unchecked by default, with a
   visible hint line and a hover/focus tooltip explaining that it generates PPNs compatible with
   the EPL backend's current checksum expectations.

## Scenario 6 — EPL v2.3.x compatibility mode combinations

For each combination below, select `PPN`, set the checkboxes accordingly, click **Generate**:

| Invalid | EPL compat | Expected result |
|---------|------------|------------------|
| off | off | success / "Generated valid PPN" (standard ISO/IEC 7064 checksum) |
| off | on  | success / "Generated valid PPN" (EPL-legacy checksum) |
| on  | off | error / "Generated invalid PPN — check digit deliberately corrupted" |
| on  | on  | error / "Generated invalid PPN — check digit deliberately corrupted" |

**Expect**: toggling EPL v2.3.x compatibility mode never changes whether the result is labelled valid or
invalid, and works independently of the `9N` prefix and embed-PZN options in any combination.

## Scenario 6a — EPL v2.3.x compatibility mode success shows a distinct caveat note

1. Select `PPN`, turn **EPL v2.3.x compatibility mode** on, leave **invalid** off, and click
   **Generate**.
2. **Expect**: the `Generated valid PPN` success message is shown, plus a separate, distinctly
   styled note reading `Note: valid only in EPL v2.3.x compatibility mode (non-standard checksum).`
   beneath it.
3. Turn **EPL v2.3.x compatibility mode** off and click **Generate** again.
4. **Expect**: the `Generated valid PPN` success message is shown with no caveat note.
5. Turn **EPL v2.3.x compatibility mode** back on, turn **invalid** on, and click **Generate**.
6. **Expect**: the error result (`Generated invalid PPN — check digit deliberately corrupted`) is
   shown with no caveat note.

## Scenario 7 — EPL-compatible values validate in the app and match the backend formula

1. Generate a `PPN` with **EPL v2.3.x compatibility mode** on and **invalid** off. Copy the value.
2. Switch to the **Validate** tab, select `PPN`, paste the value, and press **Validate**.
3. **Expect**: `✓ PPN is structurally valid` (the app accepts either checksum formula).
4. In the devtools console, confirm the checksum matches the legacy formula directly:
   ```js
   const v = generatePpn(false, true, true, true); // prefixed, embed PZN, EPL compat
   const body = v.slice(2, 12);                    // strip '9N', drop the 2-digit check
   _ppnCheckEplLegacy(body) === Number(v.slice(-2)) // → true
   ```

## Scenario 8 — PZN-to-embed field is visible only for NTIN/PPN

1. On the **Generate** tab, select each type in turn: `PZN`, `GTIN`, `PCID`.
2. **Expect**: no PZN-to-embed field is visible for any of these types.
3. Select `NTIN`, then `PPN`.
4. **Expect**: an optional PZN-to-embed text field is visible for both.

## Scenario 9 — Entering a valid PZN embeds it exactly

1. Select `PPN`, type a known valid PZN (e.g. `03879429`) into the PZN-to-embed field, leave
   **Embed a valid PZN** in any state, and click **Generate**.
2. **Expect**: success result; the embedded-PZN sub-result shows `03879429` marked valid.
3. Repeat with `NTIN` selected.
4. **Expect**: the generated NTIN's digits 5–12 equal `03879429`.

## Scenario 10 — Entering an invalid PZN blocks generation

1. Select `PPN`, type a structurally invalid value (e.g. `00000000` or `1234` or `abcdefgh`) into
   the PZN-to-embed field, and click **Generate**.
2. **Expect**: an error result is shown and no PPN value is generated.
3. Repeat with `NTIN` selected.
4. **Expect**: the same error behavior.

## Scenario 11 — Blank field preserves prior behavior

1. Select `PPN`, leave the PZN-to-embed field blank, toggle **Embed a valid PZN** on and off, and
   click **Generate** each time.
2. **Expect**: behavior identical to before this increment — the embedded PZN is random (either a
   valid generated PZN or a random 8-digit number, per the checkbox).

## Scenario 12 — No URL fragment behaves exactly as before

1. Open `index.html` with no URL fragment (or `#` alone).
2. **Expect**: the Generate tab is active and `PZN` is the selected type in both panels, exactly as
   before this increment.

## Scenario 13 — `mode` anchor activates the corresponding tab

1. Open `index.html#mode=validate`.
2. **Expect**: the Validate tab is active on load.
3. Open `index.html#mode=generate`.
4. **Expect**: the Generate tab is active on load.
5. Open `index.html#mode=GENERATE` (mixed case).
6. **Expect**: the Generate tab is active on load (case-insensitive match).

## Scenario 14 — `package-identifier` anchor preselects the type in both panels

1. Open `index.html#package-identifier=ppn`.
2. **Expect**: `PPN` is preselected in the Generate type selector, and switching to the Validate
   tab shows `PPN` preselected there too.
3. Repeat with `#package-identifier=PZN`, `#package-identifier=NTIN`, `#package-identifier=GTIN`,
   and `#package-identifier=PCID` (case-insensitive).
4. **Expect**: the matching type is preselected in both panels each time.

## Scenario 15 — Combined anchor and unrecognized-value fallback

1. Open `index.html#mode=validate&package-identifier=ppn`.
2. **Expect**: the Validate tab is active with `PPN` preselected in both type selectors.
3. Open `index.html#mode=bogus&package-identifier=xyz`.
4. **Expect**: no error is shown; the app falls back to the default Generate tab with `PZN`
   selected, exactly as with no fragment at all.
5. Open `index.html#foo=bar`.
6. **Expect**: the unrelated key is ignored; defaults apply.

## Scenario 16 — Changing the fragment while the app is open re-applies state

1. Open `index.html` with no fragment (defaults apply).
2. Using the browser's address bar (or `history.pushState`/manual edit + Enter), change the URL to
   end in `#mode=validate&package-identifier=gtin` without a full page reload.
3. **Expect**: the Validate tab becomes active and `GTIN` becomes the selected type in both panels,
   without needing to reload the page.

## Scenario 17 — Clicking a tab writes `mode` back to the fragment

1. Open `index.html` with no fragment.
2. Click the **Validate** tab.
3. **Expect**: the address bar now ends in `#mode=validate`, the page did not reload, and clicking
   the browser **Back** button does not undo the tab switch (no new history entry was created).
4. Click the **Generate** tab.
5. **Expect**: the fragment updates to `#mode=generate`.

## Scenario 18 — Changing a type selector writes `package-identifier` back to the fragment

1. On the **Generate** tab, change the identifier-type selector to `PPN`.
2. **Expect**: the address bar's fragment now includes `package-identifier=ppn`.
3. Switch to the **Validate** tab and change its identifier-type selector to `GTIN`.
4. **Expect**: the fragment updates to include `package-identifier=gtin`.

## Scenario 19 — Write-back preserves unrelated fragment keys

1. Open `index.html#foo=bar`.
2. Switch tabs and change a type selector.
3. **Expect**: the fragment updates `mode`/`package-identifier` as expected, and `foo=bar` remains
   present in the fragment throughout.

## Scenario 20 — Write-back does not cause a read/write feedback loop

1. Open `index.html` with no fragment.
2. Click the **Validate** tab, then change its type selector several times in a row.
3. **Expect**: each change updates the fragment correctly and promptly; no visible flicker,
   unexpected tab switch, or console error occurs (confirming write-back does not re-trigger
   `applyAnchorState()`).


