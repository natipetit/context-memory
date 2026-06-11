---
name: verify-at-source-not-doc-trace
description: method lesson — verifying a rule by tracing it through our own docs is circular; check the actual Figma/source files
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 7ee51af7-4992-4868-879c-136f304e3852
---

When asked to verify whether a DESIGN.md rule is valid, **do NOT verify by tracing it through the evidence docs (DESKTOP-PATTERNS, adoption inventory, etc.).** Those docs are where the rule came from — confirming "the doc says X" only confirms the doc's own claim, not reality. That's circular.

**Verify at the actual source:** open the real product Figma files (Desktop Bridge) and look at usage directly. Count frequency, check it's not an outlier.

**Why:** I "verified" the §4 floating-bottom-bar bulk rule by confirming DESKTOP-PATTERNS recommended it — and marked it done. User pushed back: the source pattern could be an outlier we don't want to reproduce (like the modal-overuse we're correcting). On real file inspection the verdict INVERTED — inline header toolbar was actually most common, floating bar was the single-file outlier. See [[bulk-action-canonical]].

**Two specific traps:**
- The **PRODUCT-DS-ADOPTION-INVENTORY is a component CENSUS** (instance counts: "tables: 36, dialog: 5"), NOT a usage/behavior analysis. "tables: N" does NOT mean "has bulk editing" — UCM files had table counts but no bulk-edit (modal-edit only).
- A file appearing in the sample list isn't automatically a source for the pattern you're checking — e.g. Assist Basics shows a "table" but it's UA backdrop behind the Assist side panel, not Assist's own table.

**How to apply:** for any "is this rule/pattern valid" verification — if the only evidence is our own synthesized docs, say so and check the source files (bridge) before closing. Don't mark a finding verified/done on doc-trace alone. When in doubt, the user often inspects files directly — surface the candidate list + exclusions and let real usage decide.
