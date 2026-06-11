---
name: figma_token_architecture
description: DTCG multi-mode convention, semantic/foundation layers, export/import pipeline
metadata:
  type: reference
---

## Token Architecture (W3C DTCG)

**Source of truth:** `tokens.json` (W3C Design Token Community Group format)

### Multi-Mode Convention

W3C DTCG doesn't standardize multi-mode (light/dark), so we use this pattern:

```json
{
  "color/primary": {
    "$value": "#3b82f6",           // default (light mode)
    "$extensions": {
      "modes": {
        "dark mode": "#1e40af"     // dark mode value
      }
    }
  }
}
```

- Default value in `$value`
- Mode-specific values in `$extensions.modes["<mode-name>"]`
- At runtime, correct value resolves based on current mode

### Three-Layer Token Hierarchy

**Foundation (raw colors):**
```
Color/Blue/50 → #eff6ff
Color/Blue/100 → #dbeafe
...
Color/Blue/950 → #0c2d6b
```

**Semantic (component-aware):**
```
color/primary → {$value: "Color/Blue/500", ...}
color/on-primary → {$value: "Color/White", ...}
button/bg-default → {$value: "color/primary", ...}
```

**Code names (CSS variables, SCSS):**
```
--color-primary: var(--color-blue-500)
--button-bg-default: var(--color-primary)
```

Figma variables cascade: code names → semantic colors → brand colors → raw colors → OKLCH literal

### Export/Import Pipeline

**Export (Figma → Code):**
```bash
figma_export_tokens --format dtcg --output tokens/tokens.json
```
Extracts all variables from Figma, writes DTCG JSON.

**Import (Code → Figma):**
```bash
figma_import_tokens --input tokens/tokens.json --strategy merge
```
Pushes token changes back to Figma (merge strategy = delta only, no destructive overwrites).

**Build CSS from tokens:**
```bash
npm run tokens:build
# Generates: app/globals.css from tokens.json
```

**Drift guard (CI):**
```bash
npm run tokens:check
# Verifies app/globals.css matches tokens.json; fails if out of sync
```

## Figma-Side Structure

**Collections (in Figma file):**
- Raw colors (primitives)
- Semantic colors (brand colors aliased to raw)
- Component tokens (button/card/input overrides)
- Code names (CSS variable dictionary)

**Single-mode vs multi-mode:**
- Foundation/raw: usually single-mode (shared across light/dark)
- Semantic: multi-mode (light primary vs dark primary)
- Component tokens: usually multi-mode

**Variable scopes:**
- Public: Published to libraries, referenced by components
- Hidden: Local to file, for internal organization
- Component properties: Also linked to variables for runtime reactivity

## Token Naming Convention

```
<category>/<variant>/<property>
```

**Examples:**
```
color/primary
color/primary/light           (variant for specific context)
spacing/vertical/sm
typescale/body/medium         (typography as tokens)
border/width/focus
button/filled/bg-primary/hover/text
```

**Rules:**
- Slashes = nesting (supports 4+ levels)
- Hyphens = compound words within a level
- No spaces; kebab-case preferred

## When to Add Tokens

- **New component state:** Button hover/focus/disabled → add tokens for each state
- **New color in palette:** Red/600 added → create token at semantic layer
- **Shared value used 3+ places:** Extract to token (avoid duplicates)
- **Brand change:** Update at semantic layer (one place, cascades to all components)

## Common Pitfalls

1. **Hardcoded values in Figma:** Always create a token first, then bind components
2. **Aliases not resolving:** Check if variable is hidden from publishing or in wrong collection
3. **Mode not syncing to code:** Ensure `$extensions.modes` is in tokens.json and build script reads it
4. **CSS variables not updating:** Check drift guard — tokens.json and globals.css must match
5. **Circular aliases:** Variable A aliases to B, B aliases to A — breaks resolution

## References

- [[figma_api_limitations_universal]] — Why paint binding is read-only (affects how tokens bind to fills)
- [[component_addition_sop]] — Adding a new component with tokens
