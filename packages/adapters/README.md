# packages/adapters — typed clients

Concrete implementations that bind the `ports/` interfaces to specific transports (HTTP, NATS, gRPC). The port is the contract; the adapter is one fulfillment.

## Status: scaffolded; content extraction pending

Source: `loa-freeside/packages/adapters/chain/score-service-client.ts:75-205`.

## Planned contents (post-extraction)

| file | purpose |
|---|---|
| `score-service-client.ts` | HTTP-over-fetch typed client. Implements `IScoreServiceClient`. Used by every TS consumer that calls into score-api over the network. |
| `nats-publisher.ts` (NEW) | NATS publisher for `activity-event` schemas. Net-new — score-api today emits these directly; this package gives a reusable typed publisher for any service that wants to emit conforming events. |
| `webhook-verifier.ts` (NEW) | HMAC verification helper for inbound score-api webhooks. Net-new per [[score-vault]] proposal. |

## Why adapters live here (not in `score-api`)

The adapter wraps the wire-format. It's the same code every TS consumer needs (mibera-world, purupuru-world, ruggy, dashboard). Living here means:
- One canonical typed client; no consumer rolls their own fetch
- Bumps follow `protocol/` + `ports/` versions (not `score-api` deploy version)
- Future Rust score-service can ship its own adapter (`packages/adapters-rust/`) without affecting TS consumers

## Pattern

Each adapter:
1. Imports the port interface from `freeside-score/ports`
2. Imports schemas from `freeside-score/protocol`
3. Implements the port; validates inputs/outputs against schemas at runtime (Zod)
4. Exposes only the port; transport details are internal

## Consumers

- World repos (mibera-world, purupuru-world, sprawl-world, apdao-world)
- `freeside-ruggy` for outbound queries (in addition to MCP)
- `loa-freeside/themes/sietch/src/jobs/score-distribution.ts` post-extraction
