---
name: user-figma-proficiency
description: "User is a proficient Figma user — don't explain how to use Figma; wait for selection confirmation before reading it"
metadata:
  type: feedback
---

User is fluent in Figma and wants TERSE communication. For any Figma action they perform manually, state only the ACTION (one line), e.g. "the Components library needs to accept the updates from Semantic." No UI walkthroughs, no panel names, no step-by-step, no explaining where to click. If they need the how, they'll ask.

**Why:** Asked directly (2026-06-02), then again after a panel-level "in the Libraries panel, accept the pending update" instruction still slipped through.
**How to apply:** Manual Figma step = one terse action line + any non-obvious gotcha only.

**WAIT for selection:** When asking the user to select a node/object in Figma, WAIT for explicit confirmation ("ok / selected / done") before reading `figma.currentPage.selection`. Reading on my own timing — even in auto mode — caught a stale/wrong selection and led to reasoning from the wrong node. Never push through on an assumed selection. Same for any "go do X in the UI then I'll check": wait for the go-ahead.
