---
name: token-tier-darkmode-prereq
description: token pipeline is flat-hex (single tier); MUST refactor to primitive+semantic-alias BEFORE dark mode / AM3 mode / Figma-mode round-trip
metadata: 
  node_type: memory
  type: project
  originSessionId: b2b465a6-9e0e-4608-84eb-c8efd752b696
---

The token pipeline is currently **single-tier flat hex**: Figma semantic vars are
resolved to a leaf hex at capture time, hardcoded in `design-tokens/upland.tokens.json`,
emitted to `src/_upland-tokens.scss`, fed into `mat.theme()` as literal hex (e.g.
`interactive/primary` → `#0c5cc5` → `$upland-color-interactive-primary` → `mat.theme`).
No live link to Figma; provenance lives only in token `$description` + `.pilot-rebind/`.

This was correct for the pilot. It will **not** survive the demo roadmap.

**Why:** dark mode, the "AM3" stock-Material mode, and the Figma-variable-mode
round-trip (all on [[pilot-to-demo-reframe]]'s work list) ALL require a **mode axis**.
Figma does light/dark as variable *modes* on the same semantic var. Flat hex has no
mode axis → dark mode would mean hand-maintaining a parallel hex set + a second
`mat.theme()` block, kept in sync forever. Same flatness also makes re-toning a manual
mass-edit (one primitive hex like `blue/60` is copied into many leaf tokens with nothing
linking them) and lets the code drift from Figma silently.

**Decision (2026-06-10):** refactor to a **3-tier graph** — primitive (`color/blue/60`)
→ semantic alias (`interactive-primary = {blue/60}`) → component — **as the FIRST task
of the dark-mode milestone**, NOT retrofitted after. The semantic-alias tier is the
*prerequisite* for dark mode, not a parallel nice-to-have. Style Dictionary supports
multi-mode + alias resolution; primitive/semantic get the mode axis, consumed via
`light-dark()` or a `[data-theme]` switch.

**Not yet:** finish the fidelity sweep first — light-mode hexes are correct, the sweep
doesn't need the graph. The flat→tiered migration is ~half a day, mechanical (~40 flat
tokens re-expressed as primitive+alias).

**Why:** locking the sequencing now prevents discovering mid-dark-mode that the whole
token base needs reworking under time pressure.
**How to apply:** when the dark-mode milestone starts, do the tier refactor before any
dark-value work. Mirror Figma's variable graph (names = Figma var names) so it stays
diffable + round-trippable. Related: [[fidelity-sweep]], [[pilot-to-demo-reframe]].
