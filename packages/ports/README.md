# packages/ports — typed interfaces

Hexagonal architecture ports. These are the interfaces consumers depend on; impls (in `score-api` or future Rust score-service) bind to them. Per [[contracts-as-bridges]]: the port is the bridge that survives impl rotation.

## Status: scaffolded; content extraction pending

Source: `loa-freeside/packages/core/ports/score-service.ts:191-254`.

## Planned contents (post-extraction)

| file | purpose |
|---|---|
| `score-service-client.ts` | The 4 snapshot-shaped methods (`getRankedHolders`, `getAddressRank`, `checkActionHistory`, `getCrossChainScore`). Today's port. |
| `recent-activity.ts` | Net-new per [[score-vault]]: `RecentActivity`, `ActivitySummary` event-stream port. Adds the digest surface persona-bots need (per [[two-layer-bot-model]]). |
| `index.ts` | Public exports |

## Why split from the protocol

- `protocol/` is wire format (data shapes — JSON Schema, Zod, NATS subjects)
- `ports/` is method signatures (the API consumers call)

A port references protocol types but adds method semantics (parameters, return shapes, error modes). Splitting lets a future Rust impl satisfy the port without inheriting TypeScript-specific bindings.

## Consumers

- `0xHoneyJar/score-api` (impl)
- `freeside-score/packages/adapters/score-service-client.ts` (typed client adapter — wraps HTTP)
- World consumers (mibera-world, purupuru-world) via the adapter
- `freeside-ruggy` for digest queries
