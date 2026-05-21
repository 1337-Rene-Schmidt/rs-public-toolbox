<!--
  SYNC IMPACT REPORT
  ==================
  Version change:   (new) → 1.0.0
  Modified principles: N/A (initial ratification — reverse-engineered from identifier-tool.html)
  Added sections:   Core Principles (I–V), Operational Constraints, Development Workflow, Governance
  Removed sections: N/A
  Templates checked:
    ✅ .specify/templates/plan-template.md  — no conflicting content
    ✅ .specify/templates/spec-template.md  — no conflicting content
    ✅ .specify/templates/tasks-template.md — no conflicting content
  Follow-up TODOs:  none
-->

# ePIL Package Identifier Tool — Constitution

## Core Principles

### I. Single-File Delivery (NON-NEGOTIABLE)

The entire tool MUST be deliverable as a single self-contained `index.html` file with all CSS
and JavaScript embedded inline. No build step, bundler, package manager, or web server is
required to open and use the tool — opening the file locally in any modern browser MUST be
sufficient for full functionality.

- Every feature MUST be implementable inside this single-file constraint.
- Splitting into separate `.css` / `.js` files is only permitted when the project is served from
  a static host and the deployment artifact is a directory (not a single file). The single-file
  form remains the canonical artifact.
- New UI sections MUST reuse the existing embedded CSS design tokens (colours, spacing, border
  radii) before adding new styles.

### II. Zero External Dependencies

The tool MUST NOT load any resource from an external origin at runtime. No CDN imports, no npm
packages, no external fonts, no analytics scripts.

- Every algorithm and UI component MUST be implemented using the platform's native browser APIs
  only (`crypto`, `navigator.clipboard`, DOM, CSS).
- When a native API is unavailable in the target browser, a pure-JavaScript fallback MUST be
  provided inline (e.g. `crypto.randomUUID` → manual UUID v4 template fallback).
- Third-party libraries are permanently out of scope, regardless of bundle size.

### III. Algorithm Fidelity (NON-NEGOTIABLE)

Every identifier algorithm MUST produce results identical to the authoritative reference
implementation for that standard. Deviations — even cosmetic ones — are considered defects.

- The reference implementation for each identifier type MUST be cited in a code comment at the
  point of implementation (e.g. `// Source: PpnValidationService.php`).
- Algorithm correctness MUST be verified against the reference before merging: at minimum, one
  known-valid and one known-invalid example per identifier type MUST be confirmed.
- Check-digit logic MUST NOT be simplified or approximated. The exact weight sequence, modulus,
  and edge-case handling (e.g. PZN check digit = 10 → unallocated slot) MUST be preserved.

### IV. Standards-First Identifiers

Only officially standardised pharmaceutical or healthcare identifier types are in scope. Informal
or proprietary identifier formats MUST NOT be added without a corresponding published standard
or regulatory mandate.

- Each supported type MUST correspond to a named standard (GS1 MOD-10 for GTIN/NTIN, IFA MOD-97
  for PPN, German IFA MOD-11 for PZN, RFC 4122 for PCID/UUID).
- The standard name and the relevant algorithmic source MUST be documented in the code.
- Adding a new identifier type requires updating: the generate switch, the validate switch, the
  type `<select>` elements, and the source citation comment block.

### V. Mobile-Accessible UI

Every interactive element MUST be fully usable on a 320 px-wide touch viewport without
horizontal scrolling or zooming.

- All tap targets MUST have a minimum rendered size of 44 × 44 CSS pixels.
- The layout MUST be single-column on narrow viewports; multi-column arrangements are only
  permitted at viewports wider than 480 px.
- The colour scheme MUST maintain a contrast ratio of at least 4.5:1 for body text against its
  background (WCAG AA).
- No JavaScript-based layout framework is permitted; all responsiveness MUST use CSS only
  (flexbox, CSS Grid, media queries, relative units).

## Operational Constraints

- **No authentication**: The tool MUST NOT require any login, token, or session. It is
  intentionally public and stateless.
- **No persistence**: No data is written to `localStorage`, `sessionStorage`, cookies, or any
  server. Each page load starts from a blank state.
- **No network requests at runtime**: After the initial page load, the tool MUST make zero
  network requests. All computation is local.
- **Graceful degradation**: When a browser API is unavailable (e.g. Clipboard API on non-HTTPS),
  the feature MUST degrade silently or show a user-friendly fallback; it MUST NOT throw an
  uncaught error.

## Development Workflow

- **Verify in-browser, no build**: Every change is validated by opening `index.html` directly in
  a browser. There is no compile step to run.
- **Algorithm changes require reference confirmation**: Any modification to a check-digit or
  parsing routine MUST be cross-checked against the cited reference before committing.
- **New identifier types follow the four-step pattern**: generate function → validate function →
  generate switch case → validate switch case. The UI `<select>` entries are updated last.
- **CSS changes use the existing token set**: Colours, radii, and spacing values already defined
  in the embedded `<style>` block are reused. New tokens are only introduced when no existing
  value is appropriate, and they MUST be named consistently with the existing scheme.

## Governance

This constitution supersedes all other project conventions. Amendments MUST follow this process:

1. Open a feature branch (`/speckit.git.feature`) and update this file.
2. Increment the version according to semantic rules:
   - **MAJOR**: Removal or redefinition of a non-negotiable principle.
   - **MINOR**: New principle or section added; new identifier standard scoped in.
   - **PATCH**: Clarifications, wording, formatting fixes.
3. Update the Sync Impact Report comment at the top of this file.
4. All PRs MUST verify that the proposed change does not violate any principle marked
   NON-NEGOTIABLE before merge.

**Version**: 1.0.0 | **Ratified**: 2026-05-21 | **Last Amended**: 2026-05-21
