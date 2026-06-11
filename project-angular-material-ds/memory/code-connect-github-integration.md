---
name: code-connect-github-integration
description: "FUTURE STEP — switch Code Connect from CLI publish to Figma's GitHub integration so the repo↔Figma connection is visible/managed in Dev Mode and publishes on merge"
metadata: 
  node_type: memory
  type: project
  originSessionId: 1e4f022b-eee3-4738-b5a0-6f361fc2e76d
---

**Want (future step, not done):** connect the GitHub repo to Figma's **Code Connect GitHub integration**, so the design↔code link is a first-class, visible connection in Dev Mode — instead of the current CLI-publish model.

**Current state:** CC is published via the **CLI** (`npx figma connect publish`, token from [[figma-access-token]]). Consequence: Figma shows **no GitHub connection**, the snippet is **read-only in Dev Mode** (repo `*.figma.ts` is source of truth), and every edit needs a manual republish. Fine for a small pilot; not ideal for a demo where stakeholders inspect the connection in Figma.

**What the GitHub integration adds:**
- Visible "connected to GitHub" link in Figma Dev Mode / org settings.
- Publish-on-merge (Figma reads `*.figma.ts` from the repo automatically) — no manual CLI step.
- Stakeholders can see the source repo from Figma.

**Setup (when we do it):** Figma org/team settings → Dev Mode → Code Connect → connect the GitHub repo (`git@github.com:your-org/angular-material-DS.git`); point it at the `src/figma/**/*.figma.ts` glob (matches `figma.config.json`). Does NOT change editability model — source still lives in-repo; the integration just automates publish + surfaces the link.

Part of the [[pilot-to-demo-reframe]] push toward the north star.
