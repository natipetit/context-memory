# Angular Material DS Memory Index

Project: git@github.com:your-org/angular-material-DS.git

> **Note (sharing copy):** This is a curated subset of the real project memory
> (10 of ~30 files), trimmed to show the *patterns* without the deep project
> minutiae. Some `[[wikilinks]]` below point to files left out of this copy —
> that is intentional. Per the memory spec, a `[[name]]` with no target file is
> valid; it marks something worth writing later. Left in to show the link graph.

## Memories

- [design-md-goal.md](design-md-goal.md) — NORTH STAR: design directly w/ Claude under DS constraints; design.md = missing patterns layer; hand-picked exemplar product files (read-only OK); brand chrome (LHN/TopBar/UserMenu) not CC'd yet = blind spot
- [verify-at-source-not-doc-trace.md](verify-at-source-not-doc-trace.md) — method: verify rules at actual Figma files, not by tracing our own docs (circular); adoption-inventory=census not usage analysis
- [repo-doc-map.md](repo-doc-map.md) — where docs live: .pilot-rebind=transferable; design-docs/=this project's DS decisions+references; DESIGN.md at repo root=entry doc
- [token-tier-darkmode-prereq.md](token-tier-darkmode-prereq.md) — token pipeline is flat-hex (no mode axis); MUST refactor to primitive+semantic-alias graph BEFORE dark mode / AM3 / Figma-mode round-trip; sequenced as first task of theme-modes milestone
- [per-component-standing-checks.md](per-component-standing-checks.md) — MANDATORY per-component gate (user-requested): a11y/aria-label in story+CC, interaction states actually render (use upland-state-layer mixin), treatment-vs-surface, disabled, CC republish; now SOP §5a
- [figma-access-token.md](figma-access-token.md) — Figma token in repo .env.local (gitignored); source it for CC publish/REST, don't ask user; Dev Mode shows PUBLISHED snippet (republish after CC edits)
- [fidelity-sweep-sop.md](fidelity-sweep-sop.md) — repeatable per-component SOP: render→inspect Figma→diff→fix via tokens→verify screenshot→commit→republish CC→log Figma changes to .pilot-rebind for the real DS
- [pilot-to-demo-reframe.md](pilot-to-demo-reframe.md) — ★ SCOPE SHIFT 2026-06-09: small pilot → full demo of north star (design-with-Claude + dev-consumption); work list = fidelity sweep, token pkg + GitHub CC, build missing comps, composed screen, dark mode + AM3 mode
- [code-connect-github-integration.md](code-connect-github-integration.md) — FUTURE: move CC from CLI publish to Figma GitHub integration (visible connection, publish-on-merge)

## Project Status

- **Stack:** Angular 21.2 standalone + Material/CDK 21.2.13, **M3** tonal theming via `mat.theme()`. Storybook 10.4.1 (`@storybook/angular`, inherits `src/styles.scss`). Stories for all pilot comps in `src/stories/`.
- **Theme = Path A+ (hybrid, Upland-source):** AM/M3 = code engine; M3 tokens restyled from Upland brand. `src/_theme-colors.scss` = M3 tonal palettes hand-built from Upland's ACTUAL primitive hexes (not `ng generate` auto-output). Brand at tone 40 → `--mat-sys-primary`=#2574db, secondary=#6b7786, error=#e60c51, surface=#f1f3f3. Font Open Sans 400/600/700. Keep Upland 3-file var architecture (don't consolidate). Full eval in repo `PARITY-ASSESSMENT.md` + `EVALUATION-PHASE2.md`.
- **PILOT COMPLETE 2026-06-03.** Phase 1 (Figma leak cleanup — all layers → 0 canonical/foreign/local bindings, fully remote+self-contained) + Phase 2 (Code Connect, merged PR #4). Full detail in [[phase1-leak-audit]], [[phase1-color-rebind]], [[foreign-instance-leak]], [[code-connect-phase2]], [[label-render-lock-root-cause]]. Latest Figma version `2360919273797564225`. Rebind key maps in repo `.pilot-rebind/`. Bridge gotchas → `~/.claude/docs/figma-desktop-bridge-notes.md`.
- **Scope guardrail (user):** work ONLY with unpublished copies user shares; do NOT touch living/published org libs.
- **OPEN doc follow-up (cosmetic):** (a) Color-page Primitive HEX text-labels static (Gray/30 reads "B2BAC5", renders #9fa9b7). (b) Alias doc-frame `2025:5834` swatch tiles gutted, scaffold kept — refill with Semantic `color-scheme/*`.
- **Open team decisions:** dark mode? tertiary hue (currently reuses blue ramp)? card rework (interaction vs elevation axis)?
- **Pilot scope:** Button, Input, Card, Checkbox + table, pagination, stepper (Upland-sourced). Slider deferred.
- **Current track (2026-06-08):** design.md review — see [[design-md-review]].

## Key Audit Findings

| Aspect | Upland DS | Angular Material (M3) | Selected |
|--------|-----------|-----------------------|----------|
| Components | 283 | 239 | Angular Material |
| Component Sets | 119 | 5 | Angular Material (canonical) |
| Token Structure | Material Design 3 semantic (195 tokens) | None | Upland semantic |
| Best For | Token source | Component source | Hybrid |

> NOTE: the AM column was originally labelled "Material 2" during the early audit. **The stack is M3** — AM 21.2 confirmed on the M3 tonal scheme (`mat.theme()`, `--mat-sys-*`, per-component `_m3-*.scss`). The M2 label was stale audit-era naming, corrected here.

**Tokens:** 91 color, 70 float, 34 string (Material Design 3, Light mode only)

## Figma Files

- M3 Design Kit (Community, the parity reference we actually use): https://www.figma.com/design/tSGPenJWqEdv9bXcViFCJA/Material-3-Design-Kit--Community-?node-id=51592-4768
- Upland Semantic: https://www.figma.com/design/fqfFZzAnb8hhExS5fiQJiu/
- Upland Main DS / PILOT: https://www.figma.com/design/jtQQW4eb9he8ZN0GHgrR0Y/
- ~~Material 2: https://www.figma.com/design/lURhv0j6ntdNDXwTLmIkVX/~~ (STALE — early-audit M2 community file, superseded by the M3 Design Kit above; not used)

## Global Memory Resources

See `~/.claude/memory/MEMORY.md` for universal learnings from ui-shadcn project:
- Figma API limitations
- Token architecture (DTCG multi-mode)
- Code Connect workflow
- Component addition SOP
- Design system collaboration rules
