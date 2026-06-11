---
name: memory-save-discipline
description: "save-to-memory / update-progress: detail to topic files, keep the MEMORY.md index compact (it has a hard size limit, loaded every session)"
metadata:
  type: feedback
---

When the user says "save to memory" / "update progress" at the end of a session or phase: write the detail to TOPIC files, and keep the main index `MEMORY.md` as compact as possible.

**Why:** MEMORY.md is the index loaded into context every session — it has a hard size limit (~24KB). Logging verbose changelogs inline blows past the limit and wastes context budget every session. (On the angular-material-DS project it overflowed once — a Phase 1 rebind changelog dumped into Project Status hit 25.9KB, trimmed to 6KB.)

**How to apply:**
- Detail (step logs, version IDs, methods, hex maps, root-cause writeups) → topic `.md` files (create or update existing).
- MEMORY.md = slim status lines + `[[topic-file]]` pointers only. One line per item, under ~200 chars.
- Before creating a topic file, check for an existing one covering it — update rather than duplicate.
- Applies to both the global index and each project's index.
