# freeside-score — agent instructions

This is a freeside-* attachment module: the keeper of score schemas. Four packages: `protocol/` (sealed schemas), `ports/` (TS interfaces), `mcp-tools/` (agent surface), `adapters/` (typed clients).

## When loaded

Load this CLAUDE.md when:
- Operator works on score schemas, NATS events, or the IScoreServiceClient port
- Operator authors a consumer of score data (worlds, ruggy, dashboard surfaces)
- Operator coordinates the loa-freeside extraction (read EXTRACTION-MAP.md + INTEGRATION-PATH.md first)

## Hard rules

- **Schemas live here, impls live elsewhere.** This repo carries sealed contracts. Hono routes / Postgres queries / V8 formula computation stay in `0xHoneyJar/score-api`. Per [[contracts-as-bridges]].
- **Schema governance imported from loa-constructs.** Enum-locked `schema_version`, additive-only minor bumps, major bumps require migration plan + new file + stable `$id` (per `packages/protocol/VERSIONING.md`).
- **Don't extract code from loa-freeside without coordination.** Today's schemas live in `loa-freeside/packages/{core/ports,adapters/chain,shared/nats-schemas}/`. Extraction is a coordination move with Jani; see `docs/INTEGRATION-PATH.md`.
- **Naming follows attachment-prefix doctrine.** This is `freeside-*` because it attaches to Freeside (and to score-api as the impl). Per [[loa-org-naming-conventions]].

## Composition

- `loa-freeside` — current owner of these schemas (will become consumer post-extraction)
- `0xHoneyJar/score-api` — impl that satisfies the IScoreServiceClient port
- `0xHoneyJar/freeside-ruggy` — consumes MCP tool specs + NATS event schemas
- `0xHoneyJar/freeside-world` — sibling module; world manifests reference compose_with: freeside-score
- Worlds (mibera-world, purupuru-world, etc.) — runtime consumers via score-api over network

## What this repo does NOT own

- The Hono API impl (`score-api` repo)
- World-specific scoring formulas (per-world handlers in score-api)
- Per-world trait/archetype enums (live in each world's own `packages/{world}-protocol/`)

## References

- Doctrine: `vault/wiki/concepts/freeside-modules-as-installables.md`
- Doctrine: `vault/wiki/concepts/score-vault.md` (precedent — this repo's lineage)
- Parent: `vault/wiki/concepts/contracts-as-bridges.md`
- Sibling: `vault/wiki/concepts/world-system-pattern.md`
