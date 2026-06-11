# Project Progress — Angular Material Design System

_Last updated: 2026-06-10_

## Current track (2026-06-09) — Storybook coverage + fidelity sweep; pilot → demo reframe

Branch `storybook-coverage-fidelity` (off main).

- **Storybook coverage — DONE.** Every Code-Connected component (29/29) now has
  a story in `src/stories/` (was 4). Standalone Meta + Playground + variants
  showcase per component.
- **Component fidelity sweep — IN PROGRESS.** Matching each AM component to its
  Upland Figma source via the token pipeline. Process in
  `design-docs/FIDELITY-SWEEP-SOP.md` (now incl. §5a mandatory per-component
  standing checks: a11y, interaction states render, treatment-vs-surface,
  disabled, CC republish). Done: button (4px radius), input (40px),
  type scale (15-role remap), tag (status colors), card (radius+outline),
  checkbox/radio/switch (#0c5cc5), chip (16px) + badge (5 types, Size enum,
  centered numerals, multi-digit pill), tabs + stepper (active/inactive colors),
  progress (#4a8fea), tooltip (navy), FAB (solid brand #2574db, standard-only),
  button-icon (transparent resting, 4px, 5 icon-color treatments + the
  state-layer/focus-outline system), **button-icon-toggle (2026-06-10:
  secondary→primary promotion on aria-pressed, two-slot glyph swap, table-row
  surface tokens)**.
  - **RESUME (next session): button-toggle-group IN PROGRESS** (committed WIP,
    not done). Token override written (4px shape, 40px, blue-wash selected, grey
    `#bfc6ce` borders via new `outline-subtle` token) + compiles. OPEN: (1) user
    wants **Figma to match the code** — selected segment's blue `#0c5cc5` border
    can't be reproduced (AM has one divider-color token); identify the exact
    Figma drift + edit + log to `.pilot-rebind`. (2) visually verify in Storybook.
  - **NOT yet swept:** datepicker, divider, dropdown-button, menu, pagination,
    search-field, select, table, textarea, timepicker.
  - **RE-SWEEP needed (only got a quick sweep, issues seen):** card — radius +
    outline done, but padding / elevation / header type / divider not deep-checked
    (CC node `14408-7621`).
  - **Custom classes namespaced `upland-`** (upland-tag/badge/icon-btn-*) +
    documented (`design-docs/UPLAND-CLASSES.md` + Storybook Autodocs).
  - **★ Interaction-state system (2026-06-10).** Upland state layers bake alpha
    INTO the token (render @ opacity 1), unlike M3's colour×per-state-opacity.
    Reusable `upland-state-layer($treatment, $token-prefix, $ripple)` mixin in
    `_upland-theme.scss` — drives `--mat-*-state-layer-color` + opacity=1, paints
    distinct pressed/active on `:active::before`, draws the keyboard-focus 2px
    ring. New token groups `state-layer/*` + `focus-outline/*`. Reuse on
    button/FAB/chip as the sweep continues. Full model + the table-row surface
    tokens in `.pilot-rebind/STATE-LAYER-MODEL.md`.
  - **Code Connect treatment mapping (2026-06-10).** Icon-button family CC
    (ButtonIcon / ButtonIconToggle) now maps Style × on-Background → the exact
    `upland-icon-btn-*` class via TWO variant-restricted `figma.connect` blocks
    (CC placeholders can't combine two axes; static parser needs inline literal
    maps). All carry aria-label + the Icon glyph slot. FAB = single treatment,
    no class. Republished.
  - **Figma changes** (logged in `.pilot-rebind/COMPONENT-RENAMES.md` for the
    real-DS pass): Badge `Is Small`→`Size` enum; FAB page unflagged + junk
    State8 deleted; Button Icon C1 tooltip (nested instance) = open investigation.
  - **OPEN Figma quirk (cosmetic, not code):** Button Icon Primary/Hover previews
    GREY in Dev Mode's behaviour panel though the variant `content` fill is
    verifiably primary-variant light blue; same in the OLD DS → pre-existing
    Figma render/mode quirk. Logged in `.pilot-rebind/STATE-LAYER-MODEL.md`.
  - **State-layer bug RESOLVED.** The on-inverse/on-primary state layers now
    render — the fix was rendering at opacity 1 with alpha baked into the token
    (the prior code left M3's .08 opacity, so the opaque fill was invisible).
- **Token pipeline — NEW.** `design-tokens/upland.tokens.json` (DTCG source of
  truth) → Style Dictionary (`npm run build:tokens`) → `src/_upland-tokens.scss`
  → `src/_upland-theme.scss` (importable `theme` + `tags` + `badges` mixins) →
  thin `src/styles.scss`. Devs consume via `@use 'upland-theme'`. See
  `design-tokens/README.md`.
- **Figma writes logged for the real DS.** Variant-prop renames made on the
  PILOT (e.g. Badge `Is Small` → `Size` enum) are logged in
  `.pilot-rebind/COMPONENT-RENAMES.md` to replay on the canonical Upland DS.
- **★ Reframe: pilot → full demo.** Working toward the north star end-to-end
  (design-with-Claude AND dev-consumption). Work list in
  `design-docs/DEMO-ROADMAP.md`: finish fidelity sweep · package tokens + move
  Code Connect to GitHub integration · build missing components (Side Panel,
  Accordion, Banner, Title Bar) · a real composed screen · theme modes
  (dark mode + an "AM3" stock-Material-3 mode), round-tripping to Figma variable
  modes.

---


## Goal

Build a design system on **Angular Material**, then connect it to Figma via **Code Connect** and document it in **Storybook**. Angular Material had to be running first so there are real components to connect and document.

**North star (2026-06-04):** be able to **design directly with Claude under the design-system's constraints** — Figma components + Angular Material + the composition patterns our products actually use (captured in a `design.md`). Code Connect + tokens are the vocabulary; `design.md` is the grammar.

---

## ✅ Done so far

### 1. Angular Material up and running
Scaffolded per the official `material.angular.dev/guide/getting-started` flow.

- **Angular 21.2** standalone app at repo root (SCSS, routing, no SSR)
- **Angular Material + CDK 21.2.13**, added via `ng add @angular/material`
- **M3 theming** via `mat.theme()` in `src/styles.scss`, prebuilt **Azure/Blue** palette
  - Emits `--mat-sys-*` CSS variables (the M3 system-variable layer)
- Roboto + Material Icons fonts linked in `src/index.html`
- **Pilot components rendered** in `src/app/app.html`: Button, Input, Card, Checkbox
- Verified: `ng build` clean, `ng serve` serves 200, screenshot confirms M3 styling
- Committed: branch `setup/angular-material`, commit `f2b9b2c` (**not pushed**)

### 2. Figma parity check — prebuilt theme vs M3 Design Kit (Community)
Compared resolved Light-mode values: Figma file `tSGPenJWqEdv9bXcViFCJA` vs the `--mat-sys-*` the theme compiles.

- **Structure = identical match:** corner radius (0/4/8/12/16/28/full), type scale (body/label/title/headline), Roboto, and the full `--mat-sys-*` token taxonomy all line up 1:1 with the Figma M3 tokens.
- **Only difference = brand hue:** Design Kit default is M3 baseline purple `#6750A4`; our theme is Azure/Blue `#005CBB`. Neutrals, error, and outlines are near-identical.
- **DECISION: keep Azure/Blue prebuilt theme, ship as-is. No custom token pipeline for the pilot.**
  - If brand needs purple later → one-liner: `primary: mat.$violet-palette`
  - If real Upland brand needed → wire in Upland M3 tokens; parity proves the structure drops in cleanly.

### 3. Storybook up and running (2026-05-29)
Initialized via `npx storybook@latest init` — **Storybook 10.4.1**, `@storybook/angular` framework.

- Builder uses `browserTarget: angular-material-DS:build`, so it **inherits `src/styles.scss`** → M3 Azure/Blue theme + `--mat-sys-*` vars load in the canvas automatically (no preview import needed).
- Storybook pulled in the webpack builder (`@angular-devkit/build-angular`) for its own use, alongside the app's `@angular/build`. Both coexist fine.
- Removed the boilerplate demo stories (`src/stories/*` button/header/page).
- **Wrote real stories for all 4 pilot components** in `src/stories/`: `button`, `input`, `card`, `checkbox`. Each has a controls-driven **Playground** + an **all-variants/states** story mirroring `app.html`.
- **Verified:** `build-storybook` completes clean; dev server serves on :6006; screenshotted all 4 — M3 Azure/Blue theme renders correctly (filled button = `#005CBB`, checkbox states, card raised vs outlined, fill/outline inputs).
- Addons installed: a11y, docs (Autodocs + Compodoc), onboarding.
- **Known minor issue:** Compodoc logs a per-file parse error during doc generation but "continues" — non-blocking, docs JSON still generates and Autodocs work. Not chased.
- `documentation.json` (Compodoc output) added to `.gitignore` (regenerated each run).

### 4. Parity assessment — AM vs M3 Figma vs Upland DS 2.1 (2026-05-29)
Two-phase evaluation, full docs in repo: `design-docs/PARITY-ASSESSMENT.md` (Phase 1 inventory) and `design-docs/EVALUATION-PHASE2.md` (Phase 2 deep dive).

- **Phase 1 (inventory + gap):** Angular Material 35 components vs M3 Figma copy (`tSGPenJWqEdv9bXcViFCJA`) vs Upland DS 2.1 copy (`jtQQW4eb9he8ZN0GHgrR0Y`). Both Figma DSs cover ~80% of AM but are **complementary** — M3 has slider/bottom-sheet Upland lacks; **Upland has table/stepper/pagination M3 lacks.**
- **Phase 2 (property + color, all 34 greenlit components):**
  - **AM 21.2 confirmed on M3 tonal color scheme** (not M2): full 0–100 tone ramps, per-component `_m3-*.scss`, `--mat-sys-*` vars. Restyle = re-seed the M3 palette; no migration.
  - M3 matches AM's variant model literally; Upland matches intent but richer (validation states, read-only, extra axes).
  - **Decision-critical:** only Upland has table, pagination, sort, stepper, real select. Team requires table + pagination.
  - Upland brand: primary blue `#2574db`, secondary gray `#6b7786`, full status ramps. Maps cleanly onto M3 by seeding the palette generator. Gaps: no dark mode, no tertiary hue.
- **Team decision: Path A+ (hybrid).** AM/M3 as code engine; restyle M3 tokens with Upland brand; reduced Upland set as Figma component source; M3 fallback for slider (expansion to be built by team).

---

## Building the A+ pilot (branch `figma-to-code-setup`, off main)

### Code side ✅ DONE (commit f7d58c0)
1. **Color restyle** — `src/_theme-colors.scss` holds M3 tonal palettes built from Upland's **actual** primitive hexes (not auto-regenerated). `styles.scss` uses them. Verified resolved: `--mat-sys-primary` #2574db, secondary #6b7786, error #e60c51, surface #f1f3f3. Status colors (`--upland-success/warning/info` + container/on-container) from green/yellow/teal ramps. Azure/Blue dropped.
2. **Font** — Roboto → **Open Sans** (400/600/700) in `mat.theme()` typography + `index.html` + `.storybook/preview-head.html`.
3. **Verified:** ng build + build-storybook clean; app + Storybook screenshots show Upland blue + Open Sans.

### Figma side
4. **Reduced Upland set** ✅ DONE — flagged 10 drop-candidate pages with `- ` prefix (flag, not delete): Alert banner, Avatar, Breadcrumb, Button Floating, Link, Title Bar, Top Bar, User Menu, illustrations, Logos. (Tags KEPT → maps to mat-chip.)
5. **Slider** ⏳ MANUAL STEP (user) — no clean programmatic cross-file import (M3 copy unpublished → import-by-key fails; recreate would break M3 color bindings). User to copy-paste the 3 M3 slider sets (Standard/Centered/Range) into the Upland copy in Figma desktop, then rebind to Upland blue.
   - _Not doing:_ expansion panel (team builds), tree, grid-list (not needed).

### Phase 1 — publish + self-contained rebind (Figma, Desktop Bridge)
User published the 3 Upland copies (Foundations → Semantic → Components). Copies' bindings still pointed at **canonical org libs** → leak. Two-phase remap making the published copies a self-contained PILOT chain (Components → PILOT Semantic → PILOT Foundations, zero canonical refs). Full detail: `.pilot-rebind/README.md` (status table = source of truth) + memory `phase1-leak-audit`, `foreign-lib-mapping`, `phase1-color-rebind`.

- ✅ **Canonical leak audit + rebind** (2026-06-02): Semantic→Foundations 169; Components→Semantic ~116k bindings. Canonical leak = 0 for color+typography.
- ✅ **Foreign-lib FLOAT cleanup** (2026-06-02): radius (~7,500) + spacing (~38,500) + border-width + Search font → PILOT. New tokens: corner-radius/{none,medium,full}, spacing/{h,v}/{none,xl}.
- ✅ **Foreign-lib COLOR cleanup** (2026-06-03, snapshots `2360858070483610807` + `2360880688328479874`): semantic (3./03. Tokens — canonical `3. Tokens` eliminated) + interactive (state-layer model) + component (Search Field/Button/Stepper) + local Components layer (32 bound vars). ~1,900 paints → PILOT `color-scheme/*`, 0 errors. Transparent unbound. Buttons render-verified Upland blue.
  - Method: `setBoundVariableForPaint` for fills/strokes (NOT `setBoundVariable` — that's floats). Maps in `.pilot-rebind/color-*.json`.
- ⏳ **Remaining (deferred, low priority):** local `2. Alias`(41)+`4. Primitives`(59) doc-only swatches → Foundations color/* (needs Foundations key harvest); legacy tail (Qvidian/Semantic Tokens v1); 7 Modal letterSpacing + node-level stuck bindings; final full re-audit.

### Phase 2 — Code Connect (DONE + extended)
- ✅ **Pilot 7 connected** (2026-06-03, merged PR #4 `code-connect-pilot`): Button, Input, Card, Checkbox + table (10 cells), pagination, stepper. HTML-parser path, published + verified Dev Mode. Snippet whitespace polished (`32c36ea`).
- ✅ **Full AM coverage** (2026-06-04, branch `code-connect-am-coverage`): connected every Upland set with a real AM equivalent — 22 more + Divider. **30 AM components / 41 connect blocks total.** Added: Badge, FAB, Button Toggle Group, Icon Button (+toggle), Chip, Datepicker, Menu+trigger, Select, Radio, Search, Snackbar, Slide Toggle, Tabs, Tag, Textarea, Timepicker, Tooltip, Progress Bar/Spinner, Divider. All dry-run valid + published --force + spot-verified `hasTemplate:true`.
- **Reverse gap (AM with no Upland set, NOT connectable):** Slider (deferred), Expansion Panel, Bottom Sheet, Tree, List, Grid List, Autocomplete (user left), Bottom Nav; Toolbar/Sidenav exist only as product chrome (Top Bar/LHN). Full report: `design-docs/CODE-CONNECT-COVERAGE.md`.

### Then
6. ~~Code Connect: map AM code ↔ reduced-Upland components~~ DONE — see Phase 2 above. Slider still deferred (M3-only set, unpublished).

### Phase 3 — design.md (DRAFTED 2026-06-05, branch `design-md-evidence`)
**North star:** design directly with Claude under DS constraints (Figma + AM + composition patterns). The missing layer = a `design.md` capturing HOW our products compose components. Code Connect gives accurate per-component code; `design.md` gives selection + composition rules. (Full context: memory `design-md-goal`, `design-md-sources`.)

- ✅ **Adoption scan (28 product files):** `design-docs/PRODUCT-DS-ADOPTION-INVENTORY.md`. Per-file tier/archetype/flags. 26/28 readable (MVP+Bulk-Add = cover-only shells via REST, but MVP actually has 23 pages — REST under-reports, bridge needed). **Findings:** bespoke side-panel = #1 missing component (14 files); modal overuse ↔ weak adoption; chrome (top-bar/LHN/title-bar) universal + UNCONNECTED = the CC blind spot; flat-vs-elevated card split.
- ✅ **MVP mobile deep-read (FileBound):** `design-docs/MVP-MOBILE-PATTERNS.md`. 6 token-level mobile patterns incl. **full-screen-modal action flow = the mobile modal-replacement** (serves "reduce modals" goal). File mixes old SVG screens + DS-rebuilt; read DS-rebuilt only.
- ✅ **Desktop deep-read (QV + UA + PSS):** `design-docs/DESKTOP-PATTERNS.md`, D1–D5 + modal taxonomy. D1 QV bulk context-menu+justified-modal · D2 UA+PSS **horizontal mat-stepper wizard** (cross-product canonical; +repeatable-row form-array) · D3 UA drilldown **side-panel** · D4 PSS User-Mgmt **enterprise data-table** (8103 LIVE instances = fidelity ref: per-col sort+filter, filter popover, col-scoped search, row ⋮, select-all-master, **floating bottom bulk bar**, paginator) · D5 QV Reports (best QV file, 11k instances): inline-toolbar bulk + Tag status pills + **full-screen-focused-page** create/edit.
- ★ **MODAL TAXONOMY (design.md centerpiece):** (1) ❌ overlay dialog = avoid; (2) ✅ side panel = keep data visible; (3) ✅ full-screen focused page = heavy create/edit. Rule: list-in-view→panel, full-focus→page, never floating overlay.
- 🎯 **CONFIRMED: side panel = #1 missing library component** (QV + UA + 14 inventory files) — config while keeping data visible. No AM 1:1 → must be built + Code-Connected.
- ⚠️ **Bulk-action fragmentation:** 3 idioms for one task (QV context-menu / PSS floating-bar / QV-Reports inline-toolbar). design.md picks ONE (rec: PSS floating bar).
- 📦 **Product map done:** `design-docs/FIGMA-SAMPLES.md` regrouped by 10 Upland products (Adestra/Assist/Cimpl/UCM/FB/PSS/QV/RA/SS/UA); memory `product-file-map`. UA = chart-heavy (viz OUT OF SCOPE, separate lib) — read UA for nav + multi-step only; its files are SVG-flattened.
- ✅ **Cimpl Inventory_Mgmt (D6):** entity-detail = **top tab-nav** (mat-tab-group across sections + status-badge header + Save/Reset) = GOOD. ❌ collapsible-panel-CARDS (old) → fix to single-block accordion. ❌ overlay "Add Travel Pack" dialog (modal tier 1) → fix to side-panel/inline. 6 products now deep-read (FB/QV/UA/PSS/Cimpl + scan).
- ✅ **Assist deep-read (D7, 2026-06-05):** `design-docs/AI-ASSISTANT-PATTERNS.md` + DESKTOP-PATTERNS D7. Assist = AI/help **right-docked drawer (360px)** over UA, opened from top-bar btn — **side-panel taxonomy applied to AI/help** (= #1-missing-component instance). 3 bridged files: non-AI panel + Basics "with AI Answers" + Content-Feedback answer-feedback loop. **Canonical AI-answer pattern:** GenAI-labeled input · `BETA` AI-answer block + Show-more · **source citations** ("built using these sources") · per-answer **👍/👎** (neg→inline improve-capture textarea) · keyword results KEPT below AI answer (hybrid) · **legal disclaimer footer**. AI = **additive not replacive** (before/after delta table in doc). 3 view depths (home→answer→article-detail w/ in-panel mat-tabs + NOTE callouts); states incl minimized + non-destructive **error banner** (404). **⚠️ LIVE modal-conflict:** SAME feedback feature = side panel in UA ✅ vs centered overlay dialog in Mobile Commons ❌ = taxonomy failure mode caught in production (worked example for design.md). Source copy bug noted (helper says "10" but scale 0–5). **Served BOTH the AI-features track AND cross-file-micro track.** Layers = named groups (not flat SVG), usable.
- ✅ **SKETCH DS DOC MINED (2026-06-05) — the authored patterns layer arrived:** user provided `UplandOne_V2_Documentation.sketch` (gitignored). No Sketch MCP exists → read locally: `.sketch`=ZIP, unzip + parse `pages/*.json` (python walker on `_class`/`name`/`frame`/`attributedString.string`). Evidence doc `design-docs/SKETCH-DS-DOC.md` + memory [[sketch-ds-doc]]. **This is the DS team's OWN written rules** (overview/usage/anatomy/behavior/a11y/specs prose), not instances. **Big finds:** (1) **Push Panel = authored spec for the #1-missing side-panel** — Push✅(no scrim) vs Flyover(scrim); "Left or right"; anatomy Container/Title/Close/bottom-action-bar; 320min/≤50% width; own scroll; →flyover on mobile. **⚠️ doc headline over-indexes LEFT (LHN/steppers); RIGHT push panel (filters/related/row-detail/config) under-emphasized → design.md must make it first-class** (user-flagged; matches QV/UA/PSS). (2) **Forms authored modal rule:** *"full attention → full screen (create) or dialog (updates/filters)"* → full-screen=tier3✅; "dialog for updates/filters" = the case prod moved to right push panel → design.md updates doc. + single-column/labels-always-visible/never-hide-nav-btns rules. (3) **CHROME now SPEC'd** (was CC blind spot): Top Bar (locked blue+positions, section-title-not-product-name), Title Bar (1 primitive 3 configs: page/panel-header/card-header; breadcrumb), LHN (=left push panel, drilldown-vs-flyover, ≥1600 expanded, search>15 items). Push/Flyover = ONE unified taxonomy across LHN+panels.
- ✅ **MATERIAL RECONCILE (2026-06-05) — `design-docs/MD-RECONCILE.md`:** DS is Material-based → reconciled Upland authored specs + deep-reads vs **M3 (m3.material.io, intent) + Angular Material (code)**; M3 leads on why/when, AM on how (user decision). **HEADLINE: Upland patterns are MD-aligned — M3 backs them in its own words.** (1) **Side Sheet = Push Panel EXACT match:** M3 standard side-sheet ("keep secondary content visible… main region scrolled/panned") = Upland push (right-anchored use M3 CONFIRMS); M3 modal side-sheet = flyover. Code: `mat-sidenav mode="side" position="end"` (no focus-trap) vs `mode="over"` (scrim+trap+Esc). (2) **M3 VALIDATES modal taxonomy from Google:** "dialogs sparingly… interruptive"; "alternatives = menus/inline expansion, maintain context"; full-screen dialog "when form fields / changes not instant" = Upland Forms rule verbatim. **Taxonomy now TRIPLE-sourced** (Upland doc + QV/UA/PSS + M3). Delta resolved: Upland's "dialog for updates/filters" → **design.md picks side sheet** (M3 keep-context + prod reality). (3) **Gap components filled by MD:** Accordion (`mat-accordion multi=false`) = directly the Cimpl-D6b fix (no Upland set→ADD); **Banner vs Snackbar** settles notifications micro-comp = TWO comps by persistence (M3: banner=persistent/top/action, snackbar=transient/bottom; Upland HAS Inline+Section Banner sets→WIRE CC). (4) Fetch note: m3.material.io JS-rendered→use WebSearch not WebFetch.
- 🎯 **MISSING-COMPONENT VERDICT (cross-ref AM35/CC30/Sketch/deep-reads):** ACT-ON priority: (1) **Side/Push Panel right** — build+CC, quadruple-confirmed; (2) **Accordion** — Cimpl fix needs it, AM has/Upland lacks → add+CC; (3) **Banner** (Inline+Section) — Upland HAS, not connected → wire CC; (4) **Title Bar** 1-comp/3-config — build+CC; (5) **Breadcrumb** — Upland-defined, skipped → reconsider (Upland addition, M3 has none). OUT/deferred: Slider, Tree, GridList, BottomSheet, BottomNav. Revisit-if-AI: Autocomplete (Assist/QV). Nice-to-have: List.
- ✅ **`design.md` DRAFTED (2026-06-05) — repo root `/design.md`:** the north-star artifact (design w/ Claude under DS constraints; selection+composition layer above Code Connect). 8 sections: (0) 3 rules that decide most screens [side-panel>modal, full-screen for create, match device]; (1) how-to-use flow; (2) task→component table; (3) **surface/modal taxonomy** (tier1 overlay=avoid, tier2 right side-panel=DEFAULT keep-context, tier3 full-screen=create; + decision flow + AM mat-sidenav code mapping + left/right push); (4) composition patterns (D4 table+floating-bulk-bar canonical, D2 stepper, D6a tab-nav, D6b accordion-not-cards, cards flat>elevated); (5) chrome (Top Bar/Title Bar 1-comp-3-config/LHN); (6) AI Assist contract; (7) a11y+tokens; (8) gaps+build-list+notifications rule+open decisions. Every rule CITED to evidence docs. Evidence index at bottom. Location = root (user-chosen, Claude entrypoint).
- ✅ **MERGED to main (2026-06-05, PR #6 `design-md-evidence`):** `design.md` + all evidence docs now on main. Branch deleted.
### Phase 3.5 — design.md REVIEW (IN PROGRESS 2026-06-08, branch `design-md-review`)
Reviewing the merged `DESIGN.md` for gaps. Findings tracked in **`design-docs/DESIGN-MD-GAPS.md`** (source-tagged claude/ux-agent/skill/user · severity 🔴🟠🟡🔵 · status). Branch `design-md-review`.
- **THE SIGNAL:** doc nails surface-selection (§0/§3 modal taxonomy = strongest, ux-agent agreed). Holes cluster in (a) **states** — empty/loading/error/permission (most blockers), (b) **screen archetypes** — settings/dashboard/list-detail/search-results.
- **Reviewers ALL DONE:** claude ✅ · ux-agent ✅ (agent `a127a9bec213130b4`) · **skill:impeccable ✅** (product-register lens; B5/S11/S12/S13/N9/N10 — doc aligns well, gaps = component-state-matrix/focus-ring/chrome-surface/state-color-vocab/touch-targets/icon-consistency) · **user ✅** (S14–S18, N11).
- **Open blockers (unwritten):** B1 empty/zero-data · B2 loading/skeleton (MVP-MOBILE P1 had it) · B3 page/form/bulk-partial error (evidence: UA "Errors and warnings" Figma `rFXzsX20bQi3r0XGQBIjw6`) · B4 §3 decision-flow mis-routes entity detail→side panel · B5 no interactive-component state matrix.
- ✅ **ALL 4 review questions CLOSED (2026-06-08), all written into DESIGN.md, source-verified via live bridge:**
  - **Q1/S14/N6 — bulk-action canonical:** verdict INVERTED at source. Canonical = **inline header toolbar** (Actions-button-on-selection); floating bottom bar = single-file outlier, dropped; right-click menu = supplement. Weight ladder: quick→inline · heavy edit→side panel · multi-step/focus→modal-or-full-screen by scale. Modal = a11y last resort. DESKTOP-PATTERNS D4b/D5a corrected. See [[bulk-action-canonical]].
  - **Q2 — icon-button a11y (§7):** aria-label mandatory always; unlabeled=incomplete→flag (except universal glyphs); touch=prefer visible label, long-press-tooltip allowlist for learnable recurring actions. (MVP file: Button Icon tooltip is boolean w/ no label → C1.)
  - **Q3 — vertical stepper (§4/§2):** horizontal=desktop/light (UA D2); vertical=heavy-step (desktop OK, PSS Dashboard Creation `68:8501` screenshot-verified) OR narrow/mobile; mobile alt=sequential full-screen (MVP).
  - **Q4:** CC path `src/figma/*.figma.ts` verified.
- 🧩 **Component/CC tasks logged (for build phase):** C1 Button Icon needs tooltip-LABEL text property + CC · C2 Tree (`mat-tree`) — file-tree in Panviva "b. Invest"+QV, §8 reclassified to "revisit, evidence exists."
- ⏳ **Open investigations:** S19 a11y (is `mode=over` side panel as bad as a modal? — flagged §7, confirm before "side panel over modal" is universal) · N11 card rules (gated on card-axis).
- **Evidence-source corrections:** Assist Basics = AI panel over UA backdrop (not table source) · UCM = modal-overuse cluster (no bulk) · UA report-builder = read-only data-preview tables (→ multi-step examples, not bulk) · adoption-inventory = component census, not usage analysis. Method lesson [[verify-at-source-not-doc-trace]].
- ⏳ **NEXT (tomorrow) = THE STATES LAYER (high-value core):** B1 empty/zero-data · B2 loading/skeleton (MVP-MOBILE P1 evidence) · B3 page/form/bulk error (evidence: UA "Errors and warnings" `rFXzsX20bQi3r0XGQBIjw6`) · B5 component state matrix. Then B4 + should-fix + nice-to-have → push branch → PR.

- ⏳ **Later:** optionally execute §8 build-list (Side Panel, Accordion, Banner CC, Title Bar).
- ⏳ **Remaining cross-file micro-comps:** notifications RESOLVED via MD reconcile (Banner vs Snackbar). Still trace **tooltips, menus** through product files; reconcile each to one canonical usage.
- **Sources:** Confluence ✅ mineable · Upland 2.0 DS Figma = images-only ⚠️ · Zeplin = DEAD ❌ · **Sketch DS doc = GOT IT ✅ (mined, gitignored)**.
- **Branches (2026-06-08 cleanup):** all feature branches merged + deleted (local + remote). Only `main` remains. `card-rework-spec` work salvaged → `design-docs/CARD-REWORK-SPEC.md` on main (parked; tied to open card-rework decision).
- **Doc reorg (2026-06-08, branch `design-md-review`):** `angular-material/` docs folder → **`design-docs/`** (held design decisions + references, name was misleading — app lives at repo root). Moved into it: `FIGMA-SAMPLES.md`, `EVALUATION-PHASE2.md`, `PARITY-ASSESSMENT.md`, `DESIGN-MD-GAPS.md`. Renamed `design.md` → **`DESIGN.md`** (consistent naming). `DESIGN.md` stays at repo root (entry doc + impeccable auto-finds it). All internal path refs rewritten.

### Open team decisions (from EVALUATION-PHASE2.md Part 5)
- [ ] Dark mode? (Upland is light-only)
- [ ] Tertiary color? (no Upland source hue)
- [ ] Card rework? (Upland interaction axis vs AM elevation axis)
- [x] Confirm adopting `#2574db`, dropping Azure `#005CBB` — DONE (code restyle `f7d58c0`, resolved `--mat-sys-primary` #2574db)

---

## Pilot scope
5–10 core components. Started with: **Button, Input, Card, Checkbox**. Table, pagination, stepper now in scope (Upland-sourced).

## Key references
- Angular Material guide: https://material.angular.dev/guide/getting-started
- Parity/property/color analysis: `design-docs/PARITY-ASSESSMENT.md`, `design-docs/EVALUATION-PHASE2.md`
- Figma copies (unpublished, read via Desktop Bridge): M3 `tSGPenJWqEdv9bXcViFCJA`, Upland 2.1 `jtQQW4eb9he8ZN0GHgrR0Y`, Semantic `fqfFZzAnb8hhExS5fiQJiu`, Foundations `LeZVTuGFzEURhRU6Tm8QXM`
- Theme seam for token/brand swap: `--mat-sys-*` variables in `src/styles.scss`
- **Scope guardrail:** work only with the unpublished Figma copies the user shares — not the living/published org libraries.
