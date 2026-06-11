# design.md — Designing with the Upland Design System

**Purpose.** This is the constraints + composition layer for designing Upland product UI *with Claude*. Code Connect already gives accurate per-component code (`<mat-*>` ↔ Upland Figma). This file gives the layer above that: **which component/surface to use for which task, and how our products compose them.** When asked to "design a screen / feature / flow," follow this.

**The stack (what we ship).** Angular Material 21.2 (M3 tonal), themed with Upland brand tokens via `--mat-sys-*`. Typeface **Open Sans**. Brand primary **#2574db**. Components = Angular Material primitives, restyled with Upland tokens, mapped to the `[PILOT] Upland DS 2.1` Figma via Code Connect.

**How this doc was built (evidence, not opinion).** Synthesized from: the **authored** UplandOne 2.0 DS docs (`design-docs/SKETCH-DS-DOC.md`), **7 product deep-reads** (`design-docs/DESKTOP-PATTERNS.md` D1–D7, `design-docs/MVP-MOBILE-PATTERNS.md`, `design-docs/AI-ASSISTANT-PATTERNS.md`), and **Material reconciliation** (`design-docs/MD-RECONCILE.md`, M3 + Angular Material). Rules below cite their source. Where Upland, production usage, and Material 3 agree, the rule is **triple-sourced** and firm; where they diverge, the chosen rule and the reason are called out.

---

## 0. The three rules that decide most screens

1. **Don't reach for a modal.** Floating overlay dialogs are the old pattern. Default to keeping the user's context visible. (Triple-sourced: Upland Forms doc · QV/UA/PSS usage · M3 "dialogs are interruptive, use sparingly".)
2. **Keep-context work goes in a side panel; whole-entity work goes full-screen.** Config / filters / detail-of-selected-row / secondary content → **right side panel** (data stays visible). Heavy create/edit of one entity → **full-screen focused page**. (§3)
3. **Match the surface to the device.** Desktop side panel ⇄ mobile full-screen page / bottom sheet. Panels become flyover (scrim) on small breakpoints. (Upland responsive rule · M3 side-sheet⇄bottom-sheet.)

If you remember nothing else: **side panel over modal, full-screen for create, never a floating dialog except a tiny confirm.**

---

## 1. How to use this doc when designing

1. **Identify the task shape** — navigate, create an entity, edit/update, filter, view detail, confirm, give feedback, ask AI. Each maps to a surface (§3) + components (§2).
2. **Pick the surface first** (page / side panel / full-screen / dialog / inline) via the taxonomy in §3.
3. **Pick components** via §2; get the code from Code Connect (`src/figma/*.figma.ts`).
4. **Compose** using the patterns in §4 (table, wizard, entity-detail, bulk actions, AI panel).
5. **Apply chrome** (§5), **AI rules** (§6), **a11y + tokens** (§7).
6. If the task needs something we don't have, see **§8 gaps** — flag it, don't improvise a one-off.

---

## 2. Component selection — task → component

Use Angular Material components (all Upland-themed). Connected components have a `.figma.ts` (see `CODE-CONNECT-COVERAGE.md`).

| Task | Component | Notes |
|---|---|---|
| Primary/secondary action | `mat-button` (filled/outlined/text) | Button.figma.ts |
| Icon-only action | `mat-icon-button` (+ toggle variant) | **accessible name required** — see icon-button rule §7 |
| Floating primary action | `mat-fab` | |
| One-of-many toggle | `mat-button-toggle-group` | |
| Single-line text input | `matInput` | label floats, always visible (§7) |
| Multi-line | `textarea matInput` | |
| Search | `matInput` + search affordance | SearchField |
| Single select | `mat-select` | Upland "Dropdown Field" |
| Multi/boolean | `mat-checkbox` · `mat-radio` · `mat-slide-toggle` | |
| Date / time | `mat-datepicker` · `mat-timepicker` | |
| Action/choice menu | `mat-menu` (+ `matMenuTriggerFor` submenus) | row ⋮, context menus |
| Tabular data | `mat-table` + `mat-paginator` + `mat-sort` | the enterprise table (§4) |
| Multi-step task | `mat-stepper` (horizontal or vertical) | the wizard (§4); orientation by step weight + viewport |
| Container/grouping | `mat-card` | prefer flat over elevated (§4) |
| Sectioned content (one block) | **`mat-accordion`** | NOT separate cards (Cimpl fix, §4) |
| Status label | `mat-chip` (Upland "Tag") | PUBLIC/SHARED/PRIVATE etc. |
| Transient feedback | `MatSnackBar` | bottom, auto-dismiss (§ notifications) |
| Persistent message + action | **Banner** (Upland Inline/Section) | top, user-dismiss (§ notifications) |
| Progress | `mat-progress-bar` · `mat-progress-spinner` | |
| Tooltip | `matTooltip` | |
| Section/panel/card header | **Title Bar** (1 component, 3 configs) | §5 |
| Contextual config/detail/filters | **Side Panel** (right) | the build-it component, §3/§8 |
| Left navigation | **LHN** (`mat-sidenav` side) | §5 |
| AI / help assistant | **Assist panel** (right side panel) | §6 |

**When in doubt between two components, the deciding axis is usually persistence or context** — e.g. transient vs persistent message = Snackbar vs Banner; keep-data-visible vs full-focus = side panel vs full-screen page.

---

## 3. Surface taxonomy — the modal decision (the core framework)

Triple-sourced (Upland authored doc · QV/UA/PSS deep-reads · M3 guidelines). Maps directly to Angular Material `mat-sidenav` modes.

### Tier 1 — ❌ Overlay dialog (floating box on dimmed scrim) = AVOID
The old pattern. M3: "use dialogs sparingly because they are interruptive… forces users to stop their current task." Replace with tier 2 or 3.
**Only justified exception:** a tiny, momentary **confirm** (e.g. delete) that must not lose the current selection/context, with no form input. `MatDialog`, minimal.

### Tier 2 — ✅ Side panel (right-anchored) = the default for keep-context work
**Use for:** contextual configuration, filters, detail of a selected row, related content you don't want to leave the page for, bulk-edit of a table selection.
- Upland calls this a **Push panel**; M3 calls it a **standard side sheet** — *"keep secondary content visible when the main region is scrolled or panned."* Same thing.
- **Content stays visible and reflows; no scrim.** This is the whole point — the user keeps the table/list in view while acting.
- **Code:** `mat-sidenav mode="side" position="end"` (no focus-trap, content reflows). Use `mode="over"` (scrim + focus-trap + Esc) only when you deliberately want to block the background (= Upland "Flyover", M3 "modal side sheet").
- **Anatomy** (Upland authored): Container · **Title Bar header (with close button)** · scrollable content (own scroll area, fixed full-height) · optional bottom action bar. Close button at the start; provide >1 way to close; don't cover the Top Bar / Title Bar. Width: **min 320px, max ≤ 50% of screen**. Does not nest inside cards — top-level UI.
- **One panel per side** at a time.

> **Left vs right.** Same component, two anchors. **Left** = navigation (LHN) and steppers. **Right** = everything contextual (filters / detail / config / AI). The authored doc over-emphasizes the left/LHN use; production (QV/UA/PSS) and M3 both confirm the **right-side contextual panel is a first-class, primary use.** Treat it as such.

### Tier 3 — ✅ Full-screen focused page = heavy create/edit of one entity
**Use for:** creating or editing a whole entity where the user needs full focus and is NOT referencing the list while working (QV Create/Edit Report; MVP mobile action flow).
- M3 "full-screen dialog": use *"when the task includes form fields, when changes aren't saved instantly, or when components open additional dialogs."* = our heavy-form case.
- Full-viewport takeover, own top bar + explicit **Close**, optional internal `mat-tab-group`, Cancel/primary footer.

### The form rule (settles tier 2 vs 3)
Upland authored: *"Filling in a form requires the full attention of the user, which is why we either use a full screen (creation) or a dialog (updates and filters)."* **Reconciled:** create → **full-screen** (tier 3); **updates/filters → side panel** (tier 2), NOT a dialog — this satisfies both M3's "keep context" and what products actually do. Reserve dialogs for tiny confirms.

### Decision flow
```
Is it navigation?            → LHN (left side panel)              §5
Tiny confirm, no input?      → small MatDialog (tier 1 exception)
Create/edit a whole entity?  → full-screen focused page (tier 3)
Else (config/filter/detail/  → right side panel (tier 2)          DEFAULT
  related/bulk-edit/AI)?
```

---

## 4. Composition patterns (from the deep-reads)

### Enterprise data table (D4 — the fidelity reference: PSS User Mgmt, 8103 instances)
`mat-table` + the full kit: per-column **sort** + **filter** (funnel turns brand-blue when active) with a filter popover (Clear/Save), column-scoped search, **row ⋮ menu**, **select-all master checkbox**, selected-row highlight, **`mat-paginator`** footer.
- **Bulk actions on a selection → inline header toolbar.** When rows are selected, action controls appear in/above the table header — commonly an **"Actions" button enabled on multi-select**. *This is the canonical bulk idiom* (most common across products: QV Reports, QV Bulk-Add, Cimpl Inventory Mgmt; PSS uses the enabled-on-selection Actions button). A **right-click context menu** may *supplement* it for extra or per-row actions (QV), but is optional, not the primary. The floating bottom action bar (PSS, D4b) appears in only one file — **not canonical; don't use for new work.** **No icons inside action menus** (Cimpl Inventory Mgmt icon-menu = outlier).
  - **What the action opens — by weight, modal as last resort** (modals are an a11y liability — see §7 — so push users to a non-overlay surface wherever the work admits one):
    1. **Quick action** (delete / export / tag, resolves immediately) → stays inline; confirm via snackbar-undo or a tiny confirm dialog. No heavy surface.
    2. **Heavy multi-field bulk-edit form** on the selection → **right side panel** (§3 tier 2, keep-context) — preferred over a modal.
    3. **Multi-step / focus-requiring / long-running process** (the action itself is a guided process, not a single edit) → **modal OR full-screen by scale**: small multi-step → modal (QV, PSS — the justified overlay that keeps selection context behind it); whole-entity / very heavy → full-screen focused page (§3 tier 3). **This is the one justified bulk-modal case** — and only after side panel / full-screen have been ruled out.

### Multi-step task wizard (D2 — cross-product: UA Report Builder, PSS Dashboard Creation)
**`mat-stepper`** with persistent Cancel/primary footer; optional steps; repeatable-row sub-forms via form-array. The canonical "guided creation" flow. Long forms break into stepper sections (Upland Forms: "broken into sections/pages", progressive disclosure).

**Orientation:**
- **Horizontal** — desktop, few steps, light per-step content (D2 — UA Report Builder). Steps as a top row.
- **Vertical** — when **(a) steps are heavy** (each step is a full config panel/form; desktop is fine — *PSS Dashboard Creation*: 4-step left rail ~320px + content right, at 1920px, steps = Name·Columns·Charts·Sorting incl. a sub-modal) **OR (b) narrow / mobile viewport** (responsive collapse — see breakpoints, §responsive). Layout = left vertical step rail (~320px) + content panel right.
- **Mobile alternative:** a multi-step task may drop the stepper entirely for **sequential full-screen pages** (MVP). Choose vertical-stepper vs full-screen-sequence by whether step progress/context must stay visible.

### Entity detail = top tab-nav (D6a — Cimpl, GOOD)
Dense entity detail → **top `mat-tab-group`** sections (e.g. Service Info / Financial / Contract / …) + entity header (back arrow + title + **status badges**) + persistent **Save/Reset** top-right.

### Sectioned content → single-block accordion, NOT separate cards (D6b — Cimpl ANTI-PATTERN fix)
❌ Old: each section in its own detached grey card. ✅ Fix: collapsible **`mat-accordion`** panels inside **one content block** (`multi="false"` = exclusive / `multi="true"` = independent). Ties to the global guidance "prefer flat over elevated", "remove the wrapping elevated card, use full width". M3 also frames inline expansion as the keep-context alternative to dialogs.

### Cards (D-global)
Prefer **flat over elevated**. Don't wrap a whole page region in an elevated card; use full width. (Open team decision: Upland's interaction-axis card — Plain/Selectable/Draggable — vs AM's elevation axis — is unresolved; see `design-docs/EVALUATION-PHASE2.md`.)

### Justified-modal note
A modal for a bulk action is justified **only when the action is a multi-step / focus-requiring process** (not the bulk-ness itself, and not a single-field edit) — see the bulk-action weight ladder above. The deciding factor is *needs full focus*, the same logic as full-screen-for-create (§3 tier 3). Even then, **prefer a right side panel or full-screen edit first** — modals are an a11y liability (§7); reach for one only when the work genuinely needs the overlay-over-context and the other surfaces don't fit.

---

## 5. Brand chrome (Top Bar / Title Bar / LHN) — now spec'd

These make a product recognizably Upland. Built on AM primitives but **brand-locked** (Upland delta from M3, deliberate). Source: `SKETCH-DS-DOC.md`.

### Top Bar (M3 Top App Bar / `mat-toolbar`)
Highest-level nav, on all products except embedded (e.g. Salesforce). **Locked:** Upland blue background, white icons/text, element positions. **Customizable:** logo initials, user-area icons, title submenus. Anatomy: LHN-menu icon · **section title** · menu (opt) · blue bar · product logo · user area · user actions. **Show the section title, not the product name.** First in keyboard/screen-reader order.

### Title Bar — one primitive, three configs
Forms a breadcrumb with the Top Bar: **Section Title › Page Title.** Three configurations:
1. **Main page** (under Top Bar): back-arrow (opt) · page title · bottom border · 1–2 buttons (more → dropdown).
2. **Side-panel header**: back-arrow (drilldown) · panel title · line · **close button**. ← this is the side-panel header from §3.
3. **Card/accordion header**: back-arrow · title · line (opt) · overflow ⋮.

Back-arrow alt text: "Go back to [parent page name]."

### LHN (left navigation = left-anchored push panel)
`mat-sidenav` left. **Drilldown (push, content reflows right) = recommended; Flyover (scrim overlay) = fallback** where push can't be implemented. Breakpoint: **≥1600px → expanded L1 default; <1600px → collapsed** (icon rail + hover/focus tooltip + bottom-right drill indicator). L2 = non-clickable group headers + clickable child links; scroll when taller than screen; **search bar appears when >15 items** (section-scoped, not global). Same Push/Flyover taxonomy as §3 side panels — one mental model for all sliding surfaces.

---

## 6. AI features (the Assist pattern — D7)

When designing an AI feature, follow the Assist panel contract (`AI-ASSISTANT-PATTERNS.md`). **AI is additive, not replacive** — it sits on top of existing search/results/actions, never removes the human/keyword path.

**The AI-answer contract (mandatory affordances):**
- **Label the AI.** GenAI tag at the input (✨), `BETA` badge + sparkle icon on the answer. Never present generated content as indistinguishable from authored content.
- **Cite sources.** List the docs the answer was built from, as links ("built using these sources").
- **Collect answer feedback.** 👍 / 👎; on 👎, open an inline "How can we improve?" capture; 👍 doesn't interrupt.
- **Keep the human path.** Keyword search results stay *below* the AI answer; "Contact Support" escalation stays.
- **Legal disclaimer** present when GenAI is on.
- **Truncate** long answers with "Show more."

**Surface:** the assistant lives in a **right side panel** (§3 tier 2), opened from a Top Bar button — same component family as every other contextual panel. Three view depths with back-arrow nav: home (search) → answer list → article detail (in-panel `mat-tab-group` + NOTE callouts). Errors surface as a **non-destructive in-panel banner** (don't blow away the view or the query). Minimized state supported.

---

## 7. Accessibility + tokens

**Forms a11y (Upland authored, M3-aligned):**
- **Labels always visible**, even after input starts (float to top on focus). Tie label to field via `for`/nesting; **hint and error text are part of the label.**
- **Errors:** explain them; show on focus-lost vs after-submit appropriately; real-time where it helps (e.g. password criteria). **Use color AND icon** — never color alone.
- **Never hide** forward/back buttons — keep them in a **focusable disabled** state until requirements are met.
- Single-column, left-aligned by default (reading order, completion time).

**Surface a11y:** side panels show a clear contextual relationship to what opened them; close button at the start; >1 way to close; don't cover Top/Title bar. `mat-sidenav` `over`/`push` trap focus + close on Esc/backdrop; `side` does not trap (correct — content stays usable). Top Bar must not create a keyboard trap; "skip to" links target the main Title Bar first.

**Modals are an a11y liability** — focus management, screen-reader context loss, keyboard traps, and on small screens they overwhelm. This is *why* the doc pushes work to side panels / full-screen pages (§0, §3, §4): prefer a non-overlay surface wherever the work admits one. Reserve modals for the justified cases (tiny confirm; multi-step focus-requiring bulk process that genuinely needs overlay-over-context). ⚠️ **Open investigation:** is a `mode="over"` side panel (which *also* traps focus + uses a scrim) just as a11y-bad as a modal? If so, the "side panel over modal" preference only holds for `mode="side"` (no trap, content stays usable) — confirm before treating side panel as the universal safer choice.

**Icon-only buttons (`mat-icon-button`):**
- **Accessible name is mandatory, always.** Every icon-only button carries an `aria-label` (the Upland Button Icon component has a Tooltip slot — fill its label; don't leave it on the boolean default with no text). This covers screen-reader users on every device.
- **An unlabeled icon-only button = incomplete → flag it, don't ship a bare icon.** The one exception: a short allowlist of **universally-understood glyphs** (close ✕, back ‹, search 🔍, overflow ⋮) may go without a *visible* label — but they still carry `aria-label`.
- **Touch (mobile has no hover, so hover-tooltips don't fire):**
  - **Default = prefer a visible text label** over icon-only for any non-obvious action. Reduce ambiguity where you can't hover to learn the icon.
  - **Exception = long-press tooltip** (`matTooltipTouchGestures="on"`) for actions **common enough to be learned** — recurring across multiple pages — so dense screens stay dense without losing a11y. Maintain an **extended allowlist** of these learnable long-press icons. *Example: the "assignment actions" icon (recurs across Assignments screens, MVP).* The tradeoff is deliberate: density is allowed, but never at the cost of an accessible name (which is always present per the first rule).

**Contrast/chrome:** white-on-Upland-blue meets min contrast; custom chrome elements (e.g. notification dot) keep min contrast (blue border around the light-blue dot).

**Tokens:** theme through **`--mat-sys-*`** only (the Upland brand seam). Don't hardcode hex. Primary #2574db, Open Sans 400/600/700. Status: `--upland-success/warning/info`. Light mode only today (dark mode = open team decision).

---

## 8. Gaps — what to build, what to flag (don't improvise)

Cross-referenced AM (35) / Code Connect (30) / authored doc / deep-reads (`MD-RECONCILE.md` §verdict). If a design needs one of these, flag it rather than inventing a one-off.

**Build + Code-Connect (priority):**
1. **Side Panel (right)** — the #1 missing component, quadruple-confirmed. `mat-sidenav side end` + Title-Bar header + optional bottom action bar. No Upland set yet → build it.
2. **Accordion** — AM has `mat-accordion`; Upland Figma has no set. Needed for the Cimpl single-block fix. → add the set + connect.
3. **Banner** (Inline + Section) — Upland *has* the sets, not connected. Persistent/top/actionable messages (pairs with the connected Snackbar). → wire CC.
4. **Title Bar** — one component, three configs (§5). Build + connect.
5. **Breadcrumb** — Upland-defined (Section › Page); M3 has none, so this is an Upland addition. Reconsider connecting.

**Notifications rule (resolved):** transient operation feedback → **Snackbar** (bottom, auto-dismiss, one at a time, never the only path to a core action). Persistent message needing action → **Banner** (top, below Top Bar, user-dismiss, one at a time). Two components, split by persistence.

**Out of scope / deferred:** Slider (deferred), Grid List, Bottom Sheet (desktop uses side sheet; mobile only), Bottom Nav. **Revisit — evidence exists:** **Tree** (`mat-tree`) — file-tree in production (Panviva "b. Invest", QV); not flatly out-of-scope, build when prioritized (see FIGMA-SAMPLES). **Revisit if AI/typeahead is prioritized:** Autocomplete (Assist search, QV autosuggest). **Nice-to-have:** `mat-list` for LHN L2 / settings rows.

**Open team decisions** (don't assume): dark mode (light-only today) · tertiary hue (reuses blue) · card axis (interaction vs elevation) · adopting #2574db over Azure.

---

## Evidence index
- `design-docs/SKETCH-DS-DOC.md` — authored UplandOne 2.0 DS docs (Push Panel, Forms, chrome specs).
- `design-docs/DESKTOP-PATTERNS.md` — D1–D7 desktop deep-reads + modal taxonomy.
- `design-docs/MVP-MOBILE-PATTERNS.md` — mobile patterns (full-screen modal replacement).
- `design-docs/AI-ASSISTANT-PATTERNS.md` — Assist / AI-answer contract.
- `design-docs/MD-RECONCILE.md` — Material 3 + Angular Material reconciliation + missing-component verdict.
- `design-docs/CODE-CONNECT-COVERAGE.md` — component ↔ Figma mapping (the code layer).
- `design-docs/EVALUATION-PHASE2.md`, `design-docs/PARITY-ASSESSMENT.md` — token/parity analysis + open decisions.
