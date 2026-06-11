---
name: design_system_collaboration
description: Ask-first culture for shared design system decisions; avoid silent changes with broad impact
metadata:
  type: feedback
---

## Collaboration Rules for Design Systems

**Core principle:** Always ask before deciding on anything with visible blast radius to other team members.

**Why:** Design system changes (Figma file, Code Connect, tokens, component structure) affect designers and developers simultaneously. Silent decisions create merge conflicts, inconsistent documentation, and diverging implementations.

### When to Ask (Stop & Ask)

- **Naming decisions** — component names, token naming conventions, file organization
- **File layout** — where to put components, tokens, Code Connect files
- **Library/tooling choices** — which token format, which design system as source of truth, Storybook vs other showcase
- **Approach/scope** — phased rollout (pilot 5 components) vs all-at-once
- **Ambiguous specs** — unclear requirements or gaps in instructions
- **Unexpected errors** — failing tests, broken Figma connections, API limitations
- **"While I'm here" cleanup** — refactoring unrelated to the current task
- **Anything touching Figma, Code Connect, or tokens** — these have visible blast radius

**How to ask:**
```
"I see two options here:
- Option A: Use Angular Material (M3) as the component source, Upland tokens for overrides
- Option B: Use Upland for both, align to AM later

A saves time initially but requires validation against AM code. B is more conservative but slower to start.

Which direction?"
```

Present tradeoffs, not just "what should I do?"

### Verify rules at the SOURCE, not by tracing your own docs

When asked to verify whether a design-system rule/pattern is valid, **do NOT verify by tracing it through your own synthesized evidence docs.** Those docs are where the rule came from — confirming "the doc says X" only confirms the doc's own claim, not reality. That's circular.

Verify at the **actual source**: open the real product/design files and look at usage directly. Count frequency, check the pattern isn't an outlier you don't want to reproduce.

**Why:** A "verified" rule (a floating-bottom-bar bulk pattern) was marked done purely because the evidence doc recommended it. On real file inspection the verdict INVERTED — the recommended pattern was a single-file outlier; the actual common pattern was different. Two traps that recur:
- An **adoption/component inventory is a census** (instance counts), NOT a behavior analysis. "tables: N" does not mean "has bulk editing."
- A file appearing in a sample list isn't automatically a source for the pattern you're checking — confirm the instance is actually that component's own usage.

**How to apply:** for any "is this rule valid" check — if the only evidence is your own synthesized docs, say so and inspect the source files before closing. Don't mark a finding verified on doc-trace alone.

### When to Proceed Without Asking (Exceptions)

- **Pure read operations** — `figma_search_components`, `git status`, reading code
- **Re-running approved commands** — user said "run this", you run it again with same args
- **Mechanical follow-through** — user approved step 1; step 2 is direct consequence (e.g., "create Code Connect mapping for the Button we just audited")

### Rules for Code Changes

**Don't add:**
- Unused abstractions or helpers
- Features not requested
- Extra error handling for impossible cases
- Refactors beyond the task scope
- Comments explaining WHAT code does (naming should be clear)

**Only add comments that explain WHY:**
- Hidden constraints ("localStorage max size is 5MB so we batch writes")
- Workarounds for specific bugs
- Non-obvious invariants ("mode resolution must cascade through alias chain")

**Bias toward simplicity:**
- Three similar lines > premature abstraction
- Don't design for hypothetical future use cases
- Delete truly unused code instead of marking it "for later"

### For This Project (angular-material-DS)

**Ask before:**
- Choosing between Angular Material vs Upland as component source (audit helps, but needs team buy-in)
- Deciding which 5-10 pilot components to start with
- Defining SCSS token conversion strategy (how to override AM's native variables)
- Structuring the monorepo layout
- Setting up Storybook integration approach
- Publishing Code Connect mappings (affects whole design + dev team)

**OK to proceed:**
- Navigating Figma files to inspect components
- Auditing token counts and naming
- Reading Angular Material 18+ docs to understand actual code structure
- Creating branch/draft work (before publication)

## References

- [[component_addition_sop]] — Checklist includes ask-first decisions
- [[figma_code_connect]] — Publishing affects team; ask before shipping
