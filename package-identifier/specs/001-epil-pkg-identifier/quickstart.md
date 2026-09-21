# Quickstart: Validate PPN Invalid / 9N-Prefix Options

**Feature**: `001-epil-pkg-identifier` — validates FR-039, FR-040, SC-009

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
