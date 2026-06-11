---
name: component_addition_sop
description: Standard operating procedure for adding/mapping new components across design and code
metadata:
  type: reference
---

## Component Addition SOP

Use this workflow when adding a new component to a design system with Code Connect.

### Phase 1: Audit & Planning

**Decision:** Choose canonical source (Material 2 for Angular Material DS; shadcn for web apps)

1. Verify the component exists in the canonical source
2. Check for existing tokens/styles needed
3. Identify variant axes (size, state, variant, etc.)
4. Document real code props that map to Figma variants

**Example:** Button component
- Canonical: Material 2 Figma file
- Variants in Figma: Size (sm/md/lg), Type (Filled/Outlined/Text)
- Code props: `variant`, `size`, `disabled`, `children`
- **Map only:** variant (matches Figma Type) and size (matches Figma Size)
- **Don't fabricate:** A "type" prop in code just to match Figma — use actual prop names

### Phase 2: Token Setup (if needed)

If the component uses color tokens that don't exist:

1. Add to `tokens/tokens.json`:
   ```json
   {
     "button/bg-primary": {
       "$value": "{color/primary}",
       "$extensions": {
         "modes": {
           "dark mode": "{color/primary/dark}"
         }
       }
     }
   }
   ```

2. Run `npm run tokens:build` to generate CSS

3. Verify in `app/globals.css`:
   ```css
   --button-bg-primary: var(--color-primary);
   ```

### Phase 3: Code Component

1. Add/update the component in `components/ui/<component-name>.tsx`
2. Ensure it accepts real props (variant, size, disabled, etc.)
3. Wire props to CSS classes or inline styles using tokens

```typescript
export interface ButtonProps {
  variant?: 'default' | 'secondary' | 'outline'
  size?: 'sm' | 'md' | 'lg'
  disabled?: boolean
  children: React.ReactNode
}

export function Button({ variant = 'default', size = 'md', disabled, children }: ButtonProps) {
  return (
    <button 
      className={cn('btn', `btn-${variant}`, `btn-${size}`, disabled && 'btn-disabled')}
      disabled={disabled}
    >
      {children}
    </button>
  )
}
```

### Phase 4: Figma Mapping

1. **Find the component-set node ID:**
   ```bash
   figma_search_components --query "Button"
   ```
   Returns: `Button — component-set: 19:6979`

2. **Create mapping file** `components/ui/button.figma.tsx`:
   ```typescript
   import React from 'react'
   import { Button } from './button'

   const figmaConfig = {
     url: 'https://www.figma.com/design/FILE_KEY/...',
     nodeId: '19:6979',
   }

   export default function ButtonStory(props: React.ComponentProps<typeof Button>) {
     return <Button {...props} />
   }

   ButtonStory.variants = {
     variant: ['default', 'secondary', 'outline'],
     size: ['sm', 'md', 'lg'],
     disabled: [true, false],
   }
   ```

3. **Validate:**
   ```bash
   npx figma connect publish --dry-run
   ```

4. **Publish:**
   ```bash
   npx figma connect publish
   ```

### Phase 5: Showcase & Documentation

1. Add to design system showcase page (Storybook, design-system page, etc.):
   ```tsx
   import { Button } from "@/components/ui/button"
   import ButtonCodeConnect from "@/components/ui/button.figma"

   export default function ComponentShowcase() {
     return (
       <section>
         <h2>Button</h2>
         <p>Links: <a href="https://material.angular.dev/components/button">AM Docs</a></p>
         <ButtonCodeConnect />
       </section>
     )
   }
   ```

2. Optional: Add component-level documentation
   - Figma description
   - Code comments on non-obvious props
   - Links to official Material Design docs

### Phase 6: Validation Checklist

- ✅ Component exists in canonical source (Figma)
- ✅ All needed tokens created and built to CSS
- ✅ Code component accepts real props (no fabricated variants)
- ✅ Code Connect mapping uses component-set node ID (not instance ID)
- ✅ `npx figma connect publish --dry-run` passes without errors
- ✅ `npx figma connect publish` succeeds
- ✅ Component visible in Figma "Code" tab
- ✅ Added to showcase/documentation
- ✅ Tests pass (if test suite exists)

### Common Pitfalls

1. **Wrong node ID:** Using instance ID from Figma UI instead of component-set ID from `figma_search_components`
2. **Fabricated props:** Creating a "style" prop in code just to match Figma's "Style" variant axis
3. **Missing tokens:** Component uses a color that doesn't have a token yet
4. **No dry-run:** Publishing without validating, catching errors later in Figma
5. **Incomplete mapping:** Only mapping some variants, leaving others unmapped in Code Connect

## References

- [[figma_code_connect]] — Detailed Code Connect workflow
- [[figma_token_architecture]] — Token creation and binding
- [[design_system_collaboration]] — Ask-first rule before adding components
