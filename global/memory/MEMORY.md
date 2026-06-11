# Global Memory Index

Universal learnings across all projects. Project-specific context lives in `.claude/projects/<project>/memory/`.

## Preferences

- [Terse answers](terse-answers.md) — cut affirm-and-restate padding (not length); keep explanations + action narration (all projects, via ~/.claude/CLAUDE.md)
- [User Figma proficiency](user-figma-proficiency.md) — user is fluent in Figma; one-line action only, no UI walkthroughs; wait for selection confirmation before reading it
- [Memory save discipline](memory-save-discipline.md) — detail → topic files; keep the MEMORY.md index compact (hard size limit, loaded every session)

## Figma & Design Systems

- [Figma API limitations](figma_api_limitations_universal.md) — Paint binding read-only; instance children untargetable; Desktop Bridge most stable
- [Figma batch screenshot rule](figma_batch_screenshot_rule.md) — always screenshot-verify batch writes; the data API confirms writes the render ignores
- [Figma token architecture (DTCG)](figma_token_architecture.md) — Multi-mode convention, semantic/foundation layers, export/import workflow
- [Code Connect best practices](figma_code_connect.md) — Component-set node IDs, mapping file structure, publication workflow
- [Component addition SOP](component_addition_sop.md) — Audit → map → Code Connect → showcase (canonical workflow)
- [Design-system collaboration rules](design_system_collaboration.md) — Ask-first culture, no silent decisions; verify rules at source not via own docs

## Projects Using These Learnings

- **ui-shadcn** (source): First project; established patterns and hit API limitations
- **angular-material-DS** (applying): Angular Material **M3** + Upland semantic tokens, SCSS override + DTCG token pipeline
