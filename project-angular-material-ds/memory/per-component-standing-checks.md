---
name: per-component-standing-checks
description: "every component in the fidelity sweep must pass standing checks — a11y/aria-label, interaction states render, treatment-vs-surface, disabled, CC republish"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: b2b465a6-9e0e-4608-84eb-c8efd752b696
---

For EVERY component from now on (fidelity sweep + beyond), run the standing checks
inline — not as a later pass. User asked for these explicitly (2026-06-10) after the
icon-button toggle, where aria-label and interaction-state rendering were easy to miss.

The checklist (now in `design-docs/FIDELITY-SWEEP-SOP.md` §5a):
- **a11y / accessible name** — icon-only / non-text controls MUST carry `aria-label`
  in BOTH the Storybook story AND the Code Connect `*.figma.ts` example (DESIGN.md §7).
  Toggles also need `aria-pressed`.
- **Interaction states actually render** — hover / focus / pressed verified in
  Storybook, not just a clean build. Use the `upland-state-layer()` mixin; don't
  hand-set `--mat-*-state-layer-color` and leave M3's .08/.12 opacity (that was the
  icon-button bug — opaque fill invisible). Focus ring is keyboard-only
  (`.cdk-keyboard-focused`), so TAB to test, don't click. See [[fidelity-sweep-sop]]
  and `.pilot-rebind/STATE-LAYER-MODEL.md`.
- **Treatment vs surface** — a Figma `on-Background`/surface variant usually documents
  WHERE the component sits (table row, dark bar), not a new button style; capture the
  surface token, don't build a new class.
- **Disabled** look matches.
- **Republish Code Connect** if any `*.figma.ts` changed — Dev Mode shows the published
  snippet, not the local file. Source the token from repo `.env.local` (see
  [[figma-access-token]]), `npx figma connect publish --dir src/figma`.

**Why:** a11y + interaction fidelity are the easiest things to skip (build passes, looks
fine at rest) and the most expensive to retrofit across N already-"done" components.
**How to apply:** treat §5a as a gate before the per-component commit. Related:
[[fidelity-sweep]], [[fidelity-sweep-sop]].
