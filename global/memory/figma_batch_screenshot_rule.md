---
name: figma_batch_screenshot_rule
description: Always screenshot-verify Figma batch writes — the plugin/data API confirms writes the render ignores; data-audit is render-blind
metadata:
  type: feedback
---

When doing Figma batch writes (`setBoundVariable` / `setBoundVariableForPaint` / `setRangeFills` / fill rebinds, via Desktop Bridge or `use_figma`), VERIFY each batch with a screenshot, not just by re-reading the data via API.

**Why:** The plugin/data API confirms writes that the render ignores — the data lens is provably blind to render truth. Real case: hours burned debugging a text-field label color bug where every data-audit (`resolveForConsumer`, `boundVariables`, segment reads) reported the binding CORRECT, but the canvas rendered gray (a render-locked node). A whole-file mismatch scan also produced thousands of false positives (raw segment color matching the resolved var on healthy nodes).

**How to apply:** After each batch, screenshot the affected node/set and eyeball hue/state. If render ≠ intended, STOP writing — don't iterate blindly (repeated rewrites never moved a render-locked node). A manual fix in Figma is often faster than chasing a render-locked node via the API. Slower per-step, large net time saver.

User confirmed: "batch updates will be done with testing using screenshots. SLOWER but huge time saver."
