---
name: design-md-goal
description: "The pilot's north star — design directly with Claude under DS constraints; design.md plan + evidence approach + card-rework context"
metadata: 
  node_type: memory
  type: project
  originSessionId: 03cb5501-a741-4323-ada4-b5396fd9747d
---

**North-star goal of the whole pilot (user, 2026-06-04):** be able to **design directly with Claude using the design-system constraints** — what's in Figma + Angular Material + the composition patterns our products use. Those patterns must be captured in a **`design.md`** file.

**Three knowledge layers for "design with Claude":**
1. Tokens — DONE (self-contained PILOT chain, `--mat-sys-*` / `color-scheme/*`).
2. Components — Code Connect maps 30 components Figma↔code (strong for node→code; weak for intent→component).
3. Patterns — **MISSING = the `design.md` gap.** Which component when, how composed, variant conventions, anti-patterns.

**Key insight (agreed):** tighter component coupling (richer CC, e.g. card rework) improves per-component **fidelity** (correct emitted code) but does NOT fix **selection/composition** accuracy — that's `design.md`. CC + `design.md` reinforce: CC = accurate vocabulary, `design.md` = grammar/style. Each `design.md` pattern should reference CC'd components by name. `design.md` is the bigger accuracy lever; CC fidelity is the smaller-but-real one.

**Evidence approach (REVISED 2026-06-04):** user will **hand-pick representative product files** per project (good-usage exemplars) instead of scanning everything. So adoption-scoring step DROPPED. Treat each file user provides as **canonical good practice** → patterns seen become rules, not candidates to score.

**Reading product files:** can't crawl a project/folder URL — both Desktop Bridge and REST/MCP are per-file (fileKey). Options: (1) user opens files in Figma Desktop → Bridge sees all, iterate via figma_navigate; (2) user gives file URL/key list → read by key. Need actual file keys, not folder links. Guardrail update: user EXPLICITLY OK'd **read-only** pattern extraction from canonical/published-DS product files (prior guardrail was unpublished-copies-only; this is read-only evidence, no rebind/link/write).

**Brand chrome blind spot:** LHN, Top Bar, User Menu are the frame of every product page but are NOT Code-Connected (no AM 1:1 — sidenav/toolbar loose). Live in PILOT file: Top Bar `2037:237`, LHN web `2050:9035` / mobile `2172:6811`, User Menu `2034:7794`. Must become connectable for product-page reads to have a baseline shell. CC target (AM sidenav/toolbar shell vs custom `<upland-*>` brand component refs) = DECIDE AFTER reading how chrome is actually used. See [[code-connect-phase2]].

**design.md structure (planned):** component-selection rules (decision trees) · composition patterns (forms/lists/dialogs/page shells) · variant conventions · token usage rules · anti-patterns. Each pattern names CC'd components.

**Status (2026-06-05): `design.md` DRAFTED — north star delivered.** Repo root `/design.md` (user-chosen location; Claude entrypoint). The patterns/grammar layer (#3 above) now exists. 8 sections: 3-rules-that-decide-most-screens · how-to-use · task→component table · **surface/modal taxonomy** (tier1 overlay avoid / tier2 right side-panel DEFAULT / tier3 full-screen create; decision flow + mat-sidenav code) · composition patterns (table+floating-bulk-bar, stepper, tab-nav, accordion-not-cards) · chrome (Top/Title/LHN) · AI Assist contract · a11y+tokens · gaps+build-list. Every rule cited to evidence docs (see [[design-md-sources]], [[sketch-ds-doc]]). Built from authored Sketch doc + 7 product deep-reads + Material reconcile = triple-sourced. **Chrome CC target now decidable** (Title Bar=build 1-comp/3-config; Top Bar/LHN=mat-toolbar/sidenav brand-locked). NEXT: user review/iterate; optionally execute §8 build-list (Side Panel, Accordion, Banner CC, Title Bar); then OPEN PR (hold lifts — samples done). Card rework parked on `card-rework-spec` (complementary CC-fidelity track).