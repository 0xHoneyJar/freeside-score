# freeside-score

> The keeper of score schemas. Sealed wire-format contracts + typed ports + MCP tool specs + adapters for the THJ scoring substrate. Installable as a module; consumed by worlds, persona-bots, and dashboards alike.

This repo houses the score-related sealed schemas + clients. Today, those schemas live inside `loa-freeside/packages/{core,adapters,shared}/`. Extracting them here gives every consumer (worlds, ruggy, score-api, future Freeside dashboard, third-party agents) a single clean import surface — and lets `score-api` impl rotate without breaking consumers.

Doctrine: [`freeside-modules-as-installables`](https://github.com/0xHoneyJar/loa-hivemind/blob/main/wiki/concepts/freeside-modules-as-installables.md) (instance-2 of the freeside-* attachment-prefix family; instance-1 is `freeside-world`).

## The four packages

```
freeside-score/
├── packages/
│   ├── protocol/      📐 sealed schemas — NATS events, webhook payloads, branded types
│   ├── ports/         🔌 TS interfaces — IScoreServiceClient + RecentActivity port
│   ├── mcp-tools/     🤖 MCP tool specs — agent-callable surface (consumed by ruggy)
│   └── adapters/      🔁 typed clients — score-service-client.ts (HTTP / NATS bindings)
└── docs/
    ├── EXTRACTION-MAP.md    where each piece lives in loa-freeside today
    └── INTEGRATION-PATH.md  how loa-freeside transitions to consuming this repo
```

| package | role | analogous to |
|---|---|---|
| `protocol/` | wire-format contracts (Draft 2020-12 JSON Schema + Zod) | `freeside-world/packages/protocol/`, `loa-constructs/.claude/schemas/runtime/` |
| `ports/` | TS interfaces consumers depend on (replaceable impls bind to these) | hexagonal architecture port pattern, per [[contracts-as-bridges]] |
| `mcp-tools/` | MCP tool specs for agent-callable surface | `score-vault/packages/mcp-tools/` (as proposed) |
| `adapters/` | typed clients that bind to the protocol over wire | `loa-freeside/packages/adapters/chain/score-service-client.ts` (current home) |

## What lives here

- NATS event schemas (`activity-event`, `activity-summary`, `rank-change`)
- Webhook payload schemas (HMAC-signed)
- Branded types: factor IDs, dimension axis enums, archetype tags
- IScoreServiceClient port (the typed API consumers depend on)
- score-service-client.ts (typed client over HTTP — adapter)
- MCP tool specs for agent-callable score queries

## What does NOT live here

- The Hono API impl — stays in `0xHoneyJar/score-api` (the runtime that satisfies the port)
- World-specific scoring formulas (V8 mibera formula, etc.) — stay in `score-api` per-world handlers
- Per-world domain types (mibera traits, dimension factors specific to one collection) — those live in each world's `packages/{world}-protocol/` per [[mibera-world-consolidation]]

## Status

- 2026-04-28 — scaffolded as part of `freeside-modules-as-installables` doctrine landing
- ⏳ schemas not yet extracted from `loa-freeside` — coordination with Jani required
- 📜 see `docs/EXTRACTION-MAP.md` for source paths
- 📋 see `docs/INTEGRATION-PATH.md` for the staged cutover plan
- 🟡 codified consumer guidance below per ecosystem-health cycle 001 phase 4 (TEND)

## Consumer guidance (extraction-pending · codified cycle 001 phase 4)

**The split**: this repo owns the schema/port surface that consumers depend on; `score-api` owns the runtime that consumers should NOT touch directly. The split is canon. The transition state is what's codified here.

**Today (extraction-pending)** — schemas described above LIVE in `loa-freeside/packages/{core,adapters,shared}/`. This repo exists to reserve the canonical name and document the target shape. Until extraction lands, consumer behavior depends on which half they touch:

| Surface | Today (extraction-pending) | After extraction |
|---|---|---|
| **Sealed schemas / typed ports / MCP specs / adapters** (canonical for consumers) | depend on `@loa-freeside/...` packages | swap to `@freeside-score/...` packages |
| **Hono API runtime** (consumers should NOT touch directly) | `0xHoneyJar/score-api` (separate repo · stable contract surface) | unchanged — `score-api` keeps owning impl |

**Why the split matters**: consumers (worlds, persona-bots, dashboards, third-party agents) should depend on the schema/port surface, not the runtime. The runtime can rotate (Hono → faster framework, NATS topology change) WITHOUT breaking consumers — IF consumers depend only on the typed contract. Today the contract lives in loa-freeside; tomorrow it lives here. Either way, the runtime stays in `score-api` and consumers don't import from it directly.

**Cutover ownership**: Jani (loa-freeside maintainer) is the gating coordinator. The cycle that lands extraction follows `docs/INTEGRATION-PATH.md` — staged cutover plan with version-pinning + back-compat aliases during the window.

## Family

| sibling | role |
|---|---|
| [`freeside-world`](https://github.com/0xHoneyJar/freeside-world) | world manifests + scaffold + creator + registry |
| [`freeside-ruggy`](https://github.com/0xHoneyJar/freeside-ruggy) | persona-bot signals + agent surface (consumes freeside-score schemas) |
| [`freeside-metadata`](https://github.com/0xHoneyJar/freeside-metadata) | NFT metadata serving + storage layout (sister stub) |

## License

MIT.

---

🌱 instance-2 of the freeside-* module pattern. The schema is the bridge.
