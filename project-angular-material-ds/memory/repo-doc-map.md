---
name: repo-doc-map
description: Where reference docs live in the repo + the transferable-vs-AM-project split. Read before touching .pilot-rebind or angular-material docs.
metadata: 
  node_type: memory
  type: project
  originSessionId: aaf05f60-a5ec-4170-8bdf-94e8881c5b49
---

Doc organization (set 2026-06-03), split by lifecycle:

**`.pilot-rebind/` = TRANSFERABLE** (canonical cleanup / any framework / Figma quirks):
- `README.md` — **execution-status table = SOURCE OF TRUTH** for what's done in the PILOT cleanup. The `*-PLAN.md` files are frozen DECISION records; their per-file `## Status` lines were stale ("Not executed") but ALL work is done — back-annotated 2026-06-03. Trust README, not a plan file's status prose.
- `FOREIGN-INSTANCE-SWAP-RUNBOOK.md` — the 4-layer leak audit (var/style bindings, instance mainComponents, INSTANCE_SWAP preferredValues/defaults, enabled-library subscriptions) + wrong-token + partial/over-rounded radius. STEPS 1–6/5a–5f. Reusable on canonical.
- `STYLE-DRIFT-PRINCIPLE.md` — generic 3-class drift framework (token-value / structure-variant / genuine-conflict). Transferable; fix-paths are per-project.
- `*.json` — name→key maps (semantic-name-to-key, foundations-name-to-key, color-*).
- `*-PLAN.md`, `MAPPING-REVIEW.md` — decision records (all EXECUTED).

**`/angular-material/` (repo root) = THIS PROJECT** (Code Connect → Angular Material, NOT transferable to canonical→other-framework):
- `CODE-CONNECT-AM.md` — `mat-*` mapping playbook: CLI-HTML path, setup, publish flow, HTML-parser gotchas, full Upland→mat-* table (7 comps + 10 table cells) w/ node ids + omitted axes.
- `DECISIONS-AND-STEPS.md` — sequencing rule (connect-more-first; structural reworks before re-connecting that comp), AM drift fix-paths, deferred decisions (dark mode / tertiary / card axis) each with execute-steps, decision template. M3-driven component reworks logged here (project-scoped, not transferable).

**Portable (cross-project, NOT in repo):** `~/.claude/docs/figma-desktop-bridge-notes.md` — Desktop Bridge gotchas, typography-binding ownership, CC token-leak rule (NEVER echo the PAT — `${VAR:-...}` leaks it), HTML-parser constraints + snippet polish.

**Token:** Figma PAT in repo `.env.local` (gitignored). The originally-leaked one was revoked; current one valid. See [[code-connect-phase2]].
