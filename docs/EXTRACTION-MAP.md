# Extraction Map — what to pull from loa-freeside

The schemas this repo will keep currently live inside `loa-freeside`. This doc names the source paths so the staged cutover (per [`INTEGRATION-PATH.md`](INTEGRATION-PATH.md)) is mechanical.

**Coordination with Jani required before physical extraction.** This map is reference; the cutover is a separate PR cycle.

## packages/ports/

| from (loa-freeside) | to (freeside-score) | notes |
|---|---|---|
| `packages/core/ports/score-service.ts:191-254` | `packages/ports/score-service-client.ts` | The 4 snapshot-shaped methods (`getRankedHolders`, `getAddressRank`, `checkActionHistory`, `getCrossChainScore`). |
| `packages/core/ports/score-service.ts:191-254` (extension) | `packages/ports/recent-activity.ts` (NEW) | The 3 additions per [`score-vault`](https://github.com/0xHoneyJar/loa-hivemind/blob/main/wiki/concepts/score-vault.md): `RecentActivity`, `ActivitySummary`, event-stream shape. |

## packages/protocol/ — schemas

| from (loa-freeside) | to (freeside-score) | notes |
|---|---|---|
| `packages/shared/nats-schemas/activity-event.ts` (or wherever exists) | `packages/protocol/activity-event.schema.json` + `.ts` (Zod) | NATS event shape — every score-mutating action. |
| `packages/shared/nats-schemas/activity-summary.ts` | `packages/protocol/activity-summary.schema.json` | Digest shape per period. |
| `packages/shared/nats-schemas/rank-change.ts` | `packages/protocol/rank-change.schema.json` | Rank-delta event. |
| `packages/shared/nats-schemas/SCHEMA-GOVERNANCE.md` | `packages/protocol/VERSIONING.md` | Already imported (enum-locked semver from loa-constructs). |
| (proposed, doesn't exist yet) | `packages/protocol/webhook-payload.schema.json` | HMAC-signed webhook payloads (per [`score-vault`](https://github.com/0xHoneyJar/loa-hivemind/blob/main/wiki/concepts/score-vault.md)). |

## packages/mcp-tools/

| from (loa-freeside) | to (freeside-score) | notes |
|---|---|---|
| (doesn't exist yet) | `packages/mcp-tools/manifest.json` + `tools/*.json` | MCP tool specs for agent-callable score queries (per `score-vault` proposal). Authored fresh; ruggy consumes. |

## packages/adapters/

| from (loa-freeside) | to (freeside-score) | notes |
|---|---|---|
| `packages/adapters/chain/score-service-client.ts:75-205` | `packages/adapters/score-service-client.ts` | Typed HTTP client for the IScoreServiceClient port. |
| `themes/sietch/src/jobs/score-distribution.ts:59-103` | (CONSUMER, stays in loa-freeside) | This is a consumer of the port; it stays in loa-freeside but rewires to import from `freeside-score/packages/ports`. |

## Branded types / enums

| from (loa-freeside) | to (freeside-score) | notes |
|---|---|---|
| (search `loa-freeside/packages/` for: factor IDs, dimension axes, archetype tags) | `packages/protocol/types.ts` | Branded TS types — used by both schemas and the port. |

## What does NOT extract

- Hono routes (stay in `score-api`)
- V8 mibera scoring formula (stays in `score-api/src/scoring/`)
- World-specific consumers (stay in their world repos; rewire imports post-extraction)

---

## How to use this map

1. Coordinate with Jani — confirm extraction window
2. For each row above, follow the staged process in `INTEGRATION-PATH.md`
3. Update `IDEMPOTENCY-REPORT` (TBD) per package as cutover lands
