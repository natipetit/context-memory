# Figma Desktop Bridge — working notes & gotchas (portable)

Framework-agnostic learnings for any project that syncs code ↔ Figma via the figma-console MCP / Desktop Bridge plugin and uses Figma Code Connect. **Nothing here is shadcn/Obra/React/Angular-specific** — those live in each project's own CLAUDE.md / DECISIONS.md. This file is shared across projects (lives in `~/.claude/docs/`).

Last updated 2026-06-01.

---

## Cross-file paste has TWO distinct remote leaks — audit them separately

When you paste a component/page from one Figma file into another (e.g. a clean source library → your working file), the pasted nodes can keep referencing the SOURCE file in two independent ways. Fixing one does NOT fix the other. If the source file will be deleted later, BOTH must be cleaned to zero or instances break.

1. **Remote variable bindings.** Fills/strokes/effects/radius bound to variables keep pointing at the source's *remote library variables* (`variable.remote === true`) — often SAME NAME, wrong cascade. Detect by walking `node.fills/strokes/effects[].boundVariables` and `node.boundVariables[radiusFields]`, resolving each var and checking `.remote`. Fix: rebind by NAME to the local equivalent var (`setBoundVariableForPaint` / `setBoundVariableForEffect` / `node.setBoundVariable`). Prefer the canonical local collection when a name exists in several.

2. **Remote main components.** INSTANCE nodes keep their *main component* pointing at the source's remote library component (`(await inst.getMainComponentAsync()).remote === true`). **This is invisible to the variable sweep** and invisible to `figma.root.findAllWithCriteria({types:['COMPONENT']})` (that lists only the document's OWN components, so it can report "0 remote" while instances still reference remote mains). Detect per-instance via `getMainComponentAsync()` key/`.remote` compare against your local component of the same name. Fix: `instance.swapComponent(localMain)`.

**Wholesale-page-paste is the usual root cause of #2** — pasting an entire documentation page drags in hundreds of instances (icons, buttons, headers, example cards) all bound to remote mains. If you only need the component sets, consider not pasting the whole doc page; or accept that you'll do a full remote-main swap pass afterward.

### Swap gotchas
- **Effect overrides survive `swapComponent`.** If an instance had an instance-level drop-shadow override, swapping the main does NOT clear it — reset `instance.effects = []` (or filter out the shadow) separately after the swap.
- **Swaps are LOSSY when local structure differs from the source.** Swapping to a local component with different internal layout/slots collapses or reflows the instance (e.g. a content-filled wrapper → an empty-slot card). Don't swap blindly across structurally different components — either rebuild, or expect to repair auto-layout by hand.
- **Verify after:** re-walk all pasted roots and assert zero `remote` bindings AND zero remote mains. Don't trust a sample — walk every node (variant sets can be large).

## Native slots > component-as-slot-swap

Older Figma files fake "slots" by nesting a placeholder component (`.Slot` → `.Slot Inner`) that you swap to insert content. Figma now has **native slots**. For any container component whose code equivalent takes arbitrary `children` (cards, popovers/panels, input groups, list items), prefer native slots — it matches the code mental model and avoids the swap hack. Converting legacy slot-hacks to native slots WILL break auto-layout; plan to repair.

## Use a Figma BRANCH for high-blast-radius changes

For changes touching published/shared components (structural swaps, slot migration), work on a **Figma branch** (native branching, created in the desktop app — not via plugin API). Benefits: diff branch-vs-main for before/after, and a human can manually repair broken auto-layout/slot reflow before merging. Often a human is FASTER at fixing reflow by hand than an agent is at preventing it — so let swaps land correct *references* even when layout breaks, and repair layout manually on the branch.

## Active-pointer DRIFT when multiple files are connected

If two+ files have the Desktop Bridge plugin open, the MCP "active file" pointer **drifts between them unpredictably** — observed flipping on roughly every other call during a long session. Consequences & defenses:
- **Assert `figma.fileKey === '<target>'` at the TOP of every `figma_execute` that writes.** Make the script no-op + return an error if it's wrong, so a drifted write hits the wrong file harmlessly.
- `figma_navigate(url)` switches the active pointer back; `figma_get_status`/`figma_list_open_files` show which file is active.
- **Best fix: close the other file (or its Bridge plugin)** so only the target is connected. Eliminates drift entirely — do this for any multi-step write job.

## `figma_execute` operational limits
- **`node.findAll(() => true)` / root-wide `findAll` can TIME OUT** (default 5s; bump `timeout` up to 30000). Scope searches to specific node IDs, not the whole root, where possible.
- **Large return values overflow** the result token limit and get spilled to a file — return compact summaries (counts, IDs, names), not full node dumps. Build name→id maps server-side and return only what you need.
- **`getMainComponentAsync()` is slow and occasionally times out / drops the connection** — it may resolve across files. Batch sparingly; retry on drop.
- **Bridge connection drops on idle / under load.** `figma_get_status({probe:true})` to health-check; `figma_reconnect` to force reconnect. A "successful reconnect" can still drop on the next call if the bridge is genuinely flaky — pause briefly rather than hammering.
- **Text edits:** load `node.fontName` and set ONE text node per `figma_execute` call (batching font loads times out the bridge window).
- **Always `saveVersionHistoryAsync('<label>')` before a destructive pass** — gives a named restore point. Re-audit (zero-remote / squared / etc.) after.

## Variable / text-style rebinding at scale (repointing remote→local bindings)

Repointing thousands of bound-variable references (e.g. cleaning a copy that leaks to a canonical published library) has several hard-won gotchas:

- **Rebind by VARIABLE KEY, not the team-library list.** `figma.teamLibrary.getAvailableLibraryVariableCollectionsAsync()` often does NOT surface a published *copy* library if it shares a name with the canonical one (Figma dedupes by name) — it lists only the canonical. But the copy's variables are still reachable: `importVariableByKeyAsync(key)` resolves them. Capture name→key maps from the source file directly; don't depend on the lib list.
- **Newly-added/published library variables won't `import` until the consuming file accepts the library update AND often a full Figma DESKTOP RESTART.** Observed: library *catalogs* the new var (shows in `getVariablesInLibraryCollectionAsync`) yet `importVariableByKeyAsync` fails — until app restart. Reconnect/accept-update alone was insufficient.
- **Per-binding `getVariableByIdAsync` + `importVariableByKeyAsync` is the speed killer.** 100k+ bindings × a bridge round-trip each = guaranteed 30s timeout (even ONE component set can blow it). Fix: there are only ~N *distinct* source variables even across 100k uses — cache `distinctSourceVarId → importedTargetVar` (resolve each once), then the per-node swap is synchronous `setBoundVariableForPaint`/`node.setBoundVariable` with no await. This drops a whole-file pass under the cap.
- **Persist the name→key map in `figma.root.setPluginData(k, json)`** so batched calls (one per page / per N-nodes, with a cursor) read it back instead of re-embedding a 15KB literal each call (and avoid `__PLACEHOLDER__` substitution — it isn't a thing; inline real literals).
- **`figma_execute` placeholders / template tokens are NOT substituted** — `__MAP__` etc. throw `unexpected token`. Inline the actual value.
- **A timed-out `figma_execute` may write NOTHING (no partial flush) OR partially — verify, don't assume.** Observed both. Always re-audit the touched scope after a timeout/drop before resuming.

### TEXT typography bindings are owned in layers — `setBoundVariable` SILENTLY NO-OPS
The single biggest trap. A text node's `fontSize/lineHeight/letterSpacing/fontFamily/fontStyle` binding can be owned by something above the node, and `node.setBoundVariable(field, v)` returns WITHOUT error but the binding does NOT change. Ownership layers, each needing a different fix:
1. **Local text style** → rebind the *style*: `textStyle.setBoundVariable(field, v)`. One fix cascades to every node using it.
2. **Remote text style** (the node references a published/canonical style) → can't edit it; **swap the node to the local equivalent**: `await node.setTextStyleIdAsync(localStyleId)`. This touches style-id ONLY — does NOT alter fills, so **no color loss** (unlike manually re-applying a style in the UI, which can bake a static fill and wipe a variable-bound color — the `(1)` same-name remote style in the picker is the usual culprit).
3. **Component TEXT property** (the text is exposed as a component property) → the property holds the external style ref; node/style swaps look clean but the picker still shows the remote style with a `(1)`. Must unlink the text property → detach typography → relink to the correct local style (manual; no clean plugin path found).
4. **Direct node override that still won't move** (e.g. `letterSpacing` on doc/annotation text) → `setBoundVariable` no-ops even with font loaded; ownership is style/property-locked. Treat as manual or accept.
- **Detection that SEES past the no-op:** after node+style level reads "0 canonical", anything that *re-surfaces* in a full re-audit is owned higher up (property/remote-style). A node-level scan alone will report clean while the property binding still leaks. Sweep `getStyledTextSegments(['fills'])` for per-range (`textRangeFills`) bindings too — those need `setRangeFills` after `loadFontAsync(getRangeAllFontNames(...))`, not the node `fills` array.
- **Remote text styles are a leak class of their own** (style-level analog of the variable leak). A clean copy needs BOTH zero canonical variable bindings AND zero remote text styles in use. The `(1)` suffix in the style picker = a cached remote style sharing a name with a local one.
- **Don't `.slice()` a style/variable id before `getStyleByIdAsync`/`getVariableByIdAsync`** — the full `S:hash,nodeid` / `VariableID:...` string is required; truncating returns "unresolvable".
- **`componentPropertyReferences` can't be JSON-serialized directly** (`Cannot unwrap symbol`) — copy its keys into a plain object first.

## Code Connect: two components on ONE node clobber silently

If two different code components must map to the SAME Figma node (e.g. a shared "Select & Combobox" set), two `figma.connect(..., sameNodeUrl, ...)` calls with NO `variant:` restriction **silently overwrite each other on publish** (last write wins, no error). Code Connect keys connections by (node-id + variant-restriction). Fix: add a distinguishing **variant axis** to the Figma set and restrict each `figma.connect` with `variant: { Axis: 'Value' }`. Lightweight version: don't duplicate the whole variant matrix — add the axis, stamp existing variants with one value, clone ONE representative for the other value. (Different nodes → no clobber, no restriction needed.)

## Code Connect CLI auth: NEVER echo the token; MCP auth ≠ shell env

- The figma MCP servers (figma + figma-console) authenticate via their OWN config (OAuth/internal token). The `@figma/code-connect` **CLI** needs a SEPARATE `FIGMA_ACCESS_TOKEN` shell env var — it is NOT shared with the MCP. So a session can have working MCP tools yet an empty `FIGMA_ACCESS_TOKEN`. Don't assume one implies the other.
- **TOKEN LEAK TRAP (burned a token once):** `echo "${FIGMA_ACCESS_TOKEN:+yes (len ${#FIGMA_ACCESS_TOKEN})}${FIGMA_ACCESS_TOKEN:-NO}"` — the intent was a presence-only check, but the trailing `${FIGMA_ACCESS_TOKEN:-NO}` (default-VALUE expansion) PRINTED THE WHOLE TOKEN into the transcript when the var was set. The `:-` / fallback form expands to the value, not "NO", when set. Any command that can expand the secret into stdout will leak it to chat.
  - **Rule:** to check presence, use a form that CANNOT emit the value: `[ -n "$FIGMA_ACCESS_TOKEN" ] && echo PRESENT || echo MISSING`. Never `${VAR:-...}` / `${VAR:+... $VAR ...}` on a secret. Never `cat .env.local`, never `env | grep`.
  - To USE the token, pass it straight into the CLI: `figma connect publish --token "$FIGMA_ACCESS_TOKEN"` — the flag value is not echoed. Source the file with `set -a; . ./.env.local; set +a` (no print).
  - If a token does leak: tell the user to REVOKE + rotate immediately; treat as compromised.
- **gitignore `.env.local` BEFORE creating it** (so there's no commit window). Verify with `git check-ignore .env.local`.
- **Credential scanning is auto-blocked:** recursively grepping home-dir configs for a token trips the auto-mode classifier (Credential Exploration). Don't sweep for tokens — ask the user where it is or to create one.

## Code Connect for Angular = no parser; two viable paths, they're DIFFERENT formats

- Figma Code Connect has NO Angular parser. `figma.connect()` parser-based publish supports React / Web Components / Vue / Svelte / SwiftUI / Compose / etc — not Angular.
- **Path 1 — MCP template (`send_code_connect_mappings`):** writes `.figma.ts` using the *MCP skill* shape (`figma.selectedInstance`, `instance.getEnum(...)`, `export default { example: figma.code\`...\` }`). Publishes server-side. GOTCHA: a plain `template` without `templateDataJson:{isParserless:true}` stores as a **simple component_browser mapping** (node→component name+import, `hasTemplate:false`, NO code snippet). Adding the template AFTER a simple mapping exists fails with **"Component is already mapped to code"** — and there is NO MCP unpublish tool to clear it.
- **Path 2 — CLI HTML (`figma connect publish`):** uses `@figma/code-connect/html` → `import figma, { html } from '@figma/code-connect/html'` then `figma.connect(url, { props: { x: figma.enum/boolean/string(...) }, example: (props) => html\`<mat-checkbox ...>\`, imports: [...] })`. Publishes under the **Web Components** label. CLI owns publish/unpublish cleanly (`figma connect unpublish --node <url> --label <label>`). This is the right path for Angular Material (emit `<mat-*>` markup in the html template).
- The two formats are NOT interchangeable — the CLI parser will not read an MCP-skill `.figma.ts`, and vice-versa. Pick the path first, write the file in that path's shape.
- `figma.config.json` for the CLI: `{ "codeConnect": { "parser": "html", "include": ["src/figma/**/*.figma.ts"], "importPaths": {...} } }`. Exclude `src/figma/**` from the Angular app build (`tsconfig.app.json` exclude) so `ng build` doesn't try to compile the CC templates. Add `@figma/code-connect/figma-types` to a tsconfig `types` for editor typing (keep it off the app build entry).

### HTML Code Connect: two more parser constraints + snippet polish
- **No inline ternaries in `html`** template — `${cond ? a : b}` throws "Expected a call expression as a placeholder, got ConditionalExpression". Resolve conditionals in `props` via `figma.enum/boolean` value-maps, interpolate the resolved prop.
- **`figma.connect(url, ...)` URL must be a string LITERAL** — a template-var URL (`` `${FILE}?node-id=` ``) is rejected by the static parser. Inline the full URL in every call.
- **Snippet polish (attr-or-empty):** `attr=${prop}` ALWAYS emits `attr="value"` — even when the value is `''`/false, leaving junk (`matButton=""`, `disabled=`, blank lines in multi-line templates). Fix: map the WHOLE attribute text or `''` in props — `disabled: figma.enum('State', { Disable: 'disabled', Default: '' })`; value-bearing too: `appearance: { Filled: 'matButton="filled"', Text: 'matButton' }`. Then interpolate as a standalone token: `<button ${props.appearance} ${props.disabled}>`. Collapse to single line where the structure allows. LIMIT: minor inter-token spaces remain where an empty token sat — the static parser can't collapse runtime whitespace; harmless, no fix.
- Publish output shows component name "undefined" + empty `source` for HTML-parser files — cosmetic (no inferred name like a React fn); the snippet is still correct/stored.

## Batch-reading Figma via subagents: use Sonnet, NOT Haiku (Haiku fake-blocks)

Fanning out many file reads across subagents (e.g. scan N product files via `get_metadata`/`get_design_context`) is the right shape — each big payload stays in the agent's context, only a compact summary returns. But the model tier matters:

- **Haiku agents HALLUCINATE a block instead of calling the tool.** Burned on a 28-file scan: ~11 agents returned `ok:false` with confident-sounding excuses ("MCP tool not invokable in this context", "requires authenticated access", "tool loaded but not callable from bash"). The tool worked fine — they never actually called it. Re-running the same files inline (main thread) or on Sonnet succeeded immediately. The "failures" were fabricated, not real barriers.
- **Rules for Figma batch-reads via agents:**
  - Use **Sonnet** for any agent that must call an MCP tool. Reserve Haiku for pure mechanical text work (grep/python on already-saved files), not tool-invocation.
  - **Deferred MCP tools must be loaded first:** an agent can't call `mcp__figma__get_metadata` until it runs `ToolSearch` (`select:mcp__figma__get_metadata`). If the agent skips that, the call fails for real — but Haiku then mislabels it as "tool not invokable" rather than loading it. Put the ToolSearch step explicitly first in the prompt.
  - **Don't trust `ok:false` from a batch** — verify a sample inline. A real empty file (e.g. cover-only) looks identical in the summary to a hallucinated failure; only an inline read distinguishes them.
- **Extraction (any tier):** `get_metadata`/`get_design_context` payloads overflow the token cap (80k–3.2M chars) and auto-save to a `.txt`. NEVER Read the raw file into context. Parse with python: `data-name="..."` → structure/component names, `var(--...)` → token families, `name="..."` (metadata XML) → node-name frequency via Counter. Returns ~2k summary from a 100k+ payload.
- **REST `get_metadata` under-reports pages** — the no-nodeId page list returned 1 page (Cover) for a file that actually had 23. For full page enumeration use the Desktop Bridge (`figma.root.children` after `loadAllPagesAsync()`), not REST.
