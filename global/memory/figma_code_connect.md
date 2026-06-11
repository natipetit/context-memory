---
name: figma_code_connect
description: Component mapping workflow, node ID requirements, publication process
metadata:
  type: reference
---

## Code Connect: Figma-to-Code Mapping

**Purpose:** Link Figma components to codebase components so designers see live code in Figma, and developers see design specifications.

### Critical: Node ID Types

| ID Type | Used For | Example | ✅ or ❌ |
|---------|----------|---------|----------|
| **Component-Set Node ID** | Code Connect mapping (CORRECT) | `19:6979` | ✅ Use this |
| **Component Instance ID** | Figma UI selection | `1:234` | ❌ Don't use |
| **Page ID** | Navigation | `1:1` | ❌ Don't use |

**How to get the right ID:** Use `figma_search_components` — it returns component-set node IDs automatically. Don't pull from Figma Inspector panel; it often shows instance IDs.

### Mapping File Structure

Create one file per component: `components/ui/<name>.figma.tsx`

```typescript
// components/ui/button.figma.tsx
import React from 'react'
import { Button } from './button'

const figmaConfig = {
  url: 'https://www.figma.com/design/FILE_KEY/...',
  nodeId: '19:6979',  // Component-set node ID from figma_search_components
}

export default function ButtonStory(props: React.ComponentProps<typeof Button>) {
  return <Button {...props} />
}

// Example variants (only map axes that correspond to real code props)
ButtonStory.variants = {
  variant: ['default', 'secondary', 'outline'],
  size: ['sm', 'md', 'lg'],
  disabled: [true, false],
}
```

**What to include:**
- Figma file URL (design/FILE_KEY)
- Component-set node ID (NOT page ID or instance ID)
- Code component import
- Minimal example showing real props
- Only map Figma variant axes that exist in code

**What NOT to do:**
- Don't fabricate code props to fit Figma axes
- Don't use instance IDs or page IDs
- Don't over-document (Code Connect is for mapping, not docs)

### Publication Workflow

1. **Dry run (validate):**
   ```bash
   npx figma connect publish --dry-run
   ```
   Reports which components will be published, any errors.

2. **Publish:**
   ```bash
   npx figma connect publish
   ```
   Requires `FIGMA_ACCESS_TOKEN` in `.env.local`.

3. **In Figma:**
   - Open the design file
   - Component should now show "Code" tab with live component preview
   - Designers can inspect code right in Figma

### Finding Component-Set Node IDs

```bash
figma_search_components --query "Button"
```

Returns:
```
Button (Primary) — component-set: 19:6979
Button (Secondary) — component-set: 19:6980
```

Use the component-set ID (the one after the component name), not the individual variant IDs.

### Common Mistakes

1. **Using instance ID instead of component-set ID:**
   - Wrong: `1:234` (selected an instance in Figma)
   - Right: `19:6979` (component-set that contains all variants)

2. **Fabricating code props to match Figma axes:**
   - Figma has Size: S/M/L and Style: Filled/Outline
   - Code only has `size` prop
   - Map ONLY size, don't invent a `style` prop in code

3. **Publishing without dry-run:**
   - Always validate first
   - Errors caught by `--dry-run` are easier to fix than debugging broken mappings in Figma

4. **Token in `.env` vs `.env.local`:**
   - Use `.env.local` (git-ignored)
   - Add to `.gitignore`: `.env.local`

### Integration with Design System

**In showcase page (Storybook, etc.):**
```tsx
export { default as ButtonCodeConnect } from '@/components/ui/button.figma'
```

This makes the Code Connect mapping available to build tools.

**In Figma:**
- Each published component shows in the "Code" tab
- Designers click "Open in Figma" to see the component in context
- Developers click "Open in codebase" to jump to implementation

## References

- [[component_addition_sop]] — Full SOP including Code Connect step
- [[figma_token_architecture]] — How tokens bind to components (relevant for variant properties)
- [[design_system_collaboration]] — Ask-first rule for Code Connect changes (affects whole team)
