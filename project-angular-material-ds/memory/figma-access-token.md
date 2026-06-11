---
name: figma-access-token
description: "Figma access token lives in repo .env.local (gitignored) — source it for any Code Connect publish or REST call; don't ask the user for a token"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 1e4f022b-eee3-4738-b5a0-6f361fc2e76d
---

A Figma personal access token is stored in the repo at **`.env.local`** (gitignored), under key **`FIGMA_ACCESS_TOKEN`**. Use it directly — do NOT ask the user to supply a token or run publish themselves.

**Publish Code Connect (or any CLI/REST needing the token):**
```bash
set -a; . ./.env.local; set +a; npx figma connect publish
```
`set -a` exports every var the file defines so the Figma CLI picks up `FIGMA_ACCESS_TOKEN` automatically. The token never echoes.

**Why this matters:** Figma Dev Mode shows the **published** Code Connect snippet, not the local `src/figma/*.figma.ts`. After editing any CC file (or renaming a Figma variant prop a CC references), republish or Dev Mode shows a stale/broken snippet (e.g. a dangling renamed prop). Verify the live result with `get_code_connect_map`.

Scope guardrail still applies: publish only targets the unpublished PILOT copies the user shares, never living org libs.
