---
name: fidelity-sweep-sop
description: "per-component fidelity SOP — the repeatable steps to match an Angular Material component to Upland Figma (render→inspect→diff→fix→verify→commit→publish→log), generalized from the badge pass"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 1e4f022b-eee3-4738-b5a0-6f361fc2e76d
---

The repeatable per-component loop for the AM-vs-Upland fidelity sweep. Generalized from the badge pass; apply to every component. Order matters — inspect before touching, verify before committing.

**1. Render current state (Storybook).** Screenshot the component's stories in the running dev server (`http://localhost:6006/iframe.html?id=<story-id>&viewMode=story`, `--headless=new`). See the actual M3 default render before judging.

**2. Inspect the Upland source (Figma).** Pull the component set from the PILOT file (`jtQQW4eb9he8ZN0GHgrR0Y`) via `use_figma` (official MCP, fileKey-based — NOT the desktop bridge which can drop). Read: variant property names + options, per-variant geometry (size, radius, padding, layout mode + axis sizing), fills/strokes with their bound variable names, text color. Drill to the painted node — variant roots are often empty wrappers (geometry on an inner frame/instance).

**3. Diff measurable props.** Build a table: Upland value vs AM M3 default vs gap. Cover radius, height/size, padding, colors (resolve to hex), type, and *behavior* (e.g. does it grow with content? badge pill vs fixed circle). Catch structural mismatches too — variant count/names (e.g. 5 Types vs 3 M3 colors), boolean-vs-enum props, sizes M3 has that Upland doesn't (and vice-versa).

**4. Decide, then fix via tokens.** Surface real choices to the user (color reconcile, keep-M3-shape-or-not, structural rename). Add any new value to `design-tokens/upland.tokens.json` (source of truth, with Figma node in `$description`) → `npm run build:tokens` → wire in `src/_upland-theme.scss` via the right `mat.<comp>-overrides()` mixin or a CSS-class mixin (when M3 has no token for it, e.g. status Tags / badge Types). Never hardcode a hex in the theme partial — it goes through a `$upland-*` token.
   - Watch for paired tokens: changing one (container-size) often needs its sibling (line-height) or the result misaligns. M3 override mixins silently ignore unknown keys, so a no-op fix builds clean but does nothing — verify visually.

**5. Verify (screenshot, don't trust the build).** build-storybook EXIT=0 ≠ correct render. Re-screenshot the fixed stories. Confirm both the fix AND that nothing else regressed. For tokens not visible statically (tooltip bg, etc.), grep the live compiled CSS for the token value.

**6. Fix the story if it misrepresents the component.** Stories sometimes show non-Upland states (a disabled example with no Upland equivalent) or render states that read wrong (resting vs floated label). Fix the `*.stories.ts` to show the real Upland variants. A story-only fix is a separate, valid commit.

**7. If the Figma component itself needs changing** (rename a variant prop to match M3, add a missing variant): make the change in the PILOT file via `use_figma`, AND **log it to `.pilot-rebind/COMPONENT-RENAMES.md`** so it gets replayed on the real/canonical Upland DS later. The PILOT is where we validate; the rebind docs carry forward. Update any Code Connect file that references the changed prop.

**8. Commit per component** (or per logical fix). Conventional message, cite the Figma node id, note "verified in Storybook."

**9. Republish Code Connect if a `*.figma.ts` changed.** Dev Mode shows the PUBLISHED snippet, not local — stale CC = broken Dev Mode snippet. Publish per [[figma-access-token]]. Verify live with `get_code_connect_map`.

**Code Connect mechanism (context):** this repo publishes CC via the **CLI** (`npx figma connect publish`), NOT Figma's GitHub integration. So Dev Mode shows no GitHub connection and the snippet is read-only there — the repo `*.figma.ts` is the source of truth; edits happen in-repo + republish. Expected, not a bug.

Related: [[fidelity-sweep]] (progress + token-pipeline architecture), [[figma-access-token]], [[verify-at-source-not-doc-trace]] (verify at the real Figma file, not by tracing our own docs).
