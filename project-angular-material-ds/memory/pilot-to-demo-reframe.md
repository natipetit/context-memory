---
name: pilot-to-demo-reframe
description: SCOPE SHIFT (2026-06-09) — reframe from small pilot to a full demo proving the north star end-to-end (design-with-Claude AND dev-consumption); the demo-grade work list
metadata: 
  node_type: memory
  type: project
  originSessionId: 1e4f022b-eee3-4738-b5a0-6f361fc2e76d
---

**Reframe (user, 2026-06-09):** stop treating this as a small pilot — build it toward a **fully fledged pilot / demo** that proves the [[design-md-goal]] north star end-to-end. We're working toward the north star now, not just validating pieces.

**Demo goal = BOTH halves of the full loop:**
1. **Design-with-Claude** — Figma components + Angular Material code + `design.md` patterns producing a real screen under DS constraints (audience: DS/design leadership).
2. **Dev-consumption / handoff** — devs consume it on their own Angular installs: shared token package, Code Connect via GitHub, Storybook as the reference (audience: eng teams).
One connected story: **Figma ↔ tokens ↔ AM ↔ Storybook ↔ Code Connect**, all demo-grade.

**Demo-grade work list (the pilot→demo gaps, user-selected):**
1. **Component coverage + fidelity** — finish the fidelity sweep across all 29 CC'd components so Storybook + Figma match convincingly. IN PROGRESS — see [[fidelity-sweep]] / [[fidelity-sweep-sop]]. (done: button, input, tag, type, card, checkbox/radio/switch, chip, badge, tabs, stepper, progress, tooltip, snackbar, fab/icon-btn.)
2. **Shared token package + GitHub Code Connect** — publishable token package devs install (token pipeline `design-tokens/` exists; may need packaging/publish); switch CC from CLI to GitHub integration → [[code-connect-github-integration]].
3. **Build the missing components** — execute `design.md` §8 build-list: **Side Panel, Accordion, Banner, Title Bar** (the patterns products need that AM/Upland lack as connected components). See [[component-gaps-buildlist]].
4. **A real composed screen** — end-to-end example screen built with Claude using the DS (chrome + components + a `design.md` pattern), to prove the north star — not just isolated components.

**NEW — theme MODES (multi-mode theming):**
5. **Dark mode** — Upland is light-only today; add a dark scheme (M3 can generate one). Was an open team decision; now in scope for the demo.
6. **"AM3" mode** — a mode that leans into stock **Material 3 styling** as a contrast to the Upland-branded mode: rounded/pill buttons, larger paddings, squircle icon buttons, tertiary color, the M3 defaults we currently override away. This is the flip side of the [[fidelity-sweep]] — instead of overriding M3→Upland, expose M3-as-designed as a selectable mode. Connects to the parked code→Figma push idea: these modes become Figma variable MODES too (Upland / AM3 / dark) for A/B in both code and design.

**Architecture implication:** the token system ([[fidelity-sweep]] `design-tokens/upland.tokens.json` → Style Dictionary → SCSS) should grow to emit **multiple modes** (Upland-light, Upland-dark, AM3), and those modes should round-trip to Figma variable modes. Style Dictionary's multi-file/mode output + a Figma transform is the path.

This memory is the umbrella; the linked memories hold the detail per workstream.
