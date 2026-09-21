# Quickstart: Validate PPN Invalid / 9N-Prefix / EPL Compatibility / Custom PZN Embedding Options

**Feature**: `001-epil-pkg-identifier` — validates FR-039, FR-040, FR-041–FR-045, FR-046–FR-049, SC-009, SC-010, SC-011

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

