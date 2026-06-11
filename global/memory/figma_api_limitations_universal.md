---
name: figma_api_limitations_universal
description: Paint fill variable binding blocked across all access methods; instance children untargetable; tooling stability trade-offs
metadata:
  type: reference
---

## Paint Fill Variable Binding (Blocked)

**Status:** Read-only across all Figma APIs.

- **Plugin API:** `node.setBoundVariable('fills', 0, variable)` throws "fills and strokes variable bindings must be set on paints directly"
- **Paint object:** `paint.boundVariables` property is read-only; no writable method exists
- **figma-cli:** `set fill "var:color"` command works but targets wrong scope (instance instead of child rectangle)
- **REST API:** Enterprise-only; public docs don't confirm paint binding support

**Workaround:** Manual rebinding in Figma UI or contact Figma support for Enterprise capability.

**Why this matters:** Automating swatch/fill rebinding or bulk variable reassignments is currently impossible.

## Instance Child Elements (Untargetable)

**Problem:** Child elements of component instances don't have independent node IDs.

```
.SomeComponent [INSTANCE, ID: 7119:644]
├── Rectangle [no separate ID — part of component definition]
│   └── fill: bound to variable
└── Text [no separate ID]
```

**Impact:** Can't target child elements via node ID for variable binding or direct manipulation.

**Workaround:** Traverse via Plugin API (`node.children.find(...)`) then mutate, but bounded by write limitations above.

## Tooling Comparison

| Tool | Stability | Capability | Best For |
|------|-----------|-----------|----------|
| **Desktop Bridge** | ✅ Most stable (WebSocket) | Read, inspect, limited mutations | Visual inspection, understanding structure |
| **figma-cli** | ⚠️ Flaky (disconnects when switching plugins) | `set fill` works, but limited scoping | Simple property updates (when connected) |
| **Plugin API** | ✅ Stable (direct Figma context) | Async traversal works; mutations blocked | Reading state, understanding node trees |
| **REST API** | ✅ Stable (HTTP) | Enterprise-only; paint binding unknown | Unknown; needs Figma support validation |

**Key:** Desktop Bridge and figma-cli can't run simultaneously — they compete for plugin connection.

## Recommendations

1. **For inspection/reading:** Use Desktop Bridge (figma-console MCP)
2. **For simple updates:** Use figma-cli `set` commands when stable
3. **For complex mutations:** Use Plugin API via figma_execute, accept read-only paint limitations
4. **For paint binding automation:** Escalate to Figma support or wait for Enterprise REST API clarification

## References

- [[figma_token_architecture]] — How to work around paint binding limits via semantic layers
- [[code_connect_best_practices]] — Mapping components without relying on automated variable binding
