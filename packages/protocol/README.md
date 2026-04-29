# packages/protocol — sealed score schemas

Wire-format contracts that bridge `freeside-score` consumers (worlds, ruggy, score-api impl, dashboard) to a single coherent vocabulary about scoring activity, summaries, rank changes, and webhook payloads.

## Status: scaffolded; content extraction pending

This package is the destination for schemas currently living in `loa-freeside/packages/shared/nats-schemas/` and adjacent paths. See [`../../docs/EXTRACTION-MAP.md`](../../docs/EXTRACTION-MAP.md) for the per-file source map and [`../../docs/INTEGRATION-PATH.md`](../../docs/INTEGRATION-PATH.md) for the staged cutover.

## Planned contents (post-extraction)

| file | purpose |
|---|---|
| `activity-event.schema.json` + `.ts` (Zod) | Per-action score event — emitted by score-api on every score-mutating action. NATS subject: `score.activity.{world}.{action}`. |
| `activity-summary.schema.json` + `.ts` | Periodic digest — top actors, factor breakdown, notable events per period. |
| `rank-change.schema.json` + `.ts` | Rank-delta event — emitted when a holder's rank crosses a threshold. |
| `webhook-payload.schema.json` + `.ts` | HMAC-signed webhook payload format. Net-new per [[score-vault]] proposal. |
| `types.ts` | Branded TS types — factor IDs, dimension axes, archetype tags. Used by all schemas + the port. |
| `VERSIONING.md` | Schema governance (imported from loa-constructs). Enum-locked, additive-only minors. |

## Governance

Same as `freeside-world/packages/protocol/VERSIONING.md` — imported verbatim from `loa-constructs/.claude/schemas/VERSIONING.md`. Enum-locked `schema_version`. Minor bumps additive only. Major bumps require new file + migration plan + stable `$id`.

## Consumers (post-extraction)

- `freeside-score/packages/ports/` — type the port off these schemas
- `freeside-score/packages/adapters/score-service-client.ts` — validates over wire
- `freeside-score/packages/mcp-tools/` — agent-callable surface
- `0xHoneyJar/score-api` — impl validates inputs/outputs against these
- `0xHoneyJar/freeside-ruggy` — consumes activity-event + rank-change for fan-out
- `0xHoneyJar/freeside-world` — world manifests reference compose_with: freeside-score; dashboard renders schema-typed events
- (worlds) `mibera-world`, `purupuru-world`, etc. — typed consumers via `freeside-score/ports`
