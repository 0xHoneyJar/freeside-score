# Integration Path — staged cutover with loa-freeside

`loa-freeside` today owns the score schemas (per [`EXTRACTION-MAP.md`](EXTRACTION-MAP.md) source paths). This doc describes the staged cutover so consumers don't break.

Per [`freeside-modules-as-installables`](https://github.com/0xHoneyJar/loa-hivemind/blob/main/wiki/concepts/freeside-modules-as-installables.md): two stages, soft then hard.

## Stage 1 — Soft cutover (this scaffold)

**Goal**: this repo exists with the right shape; ready to receive extracted schemas.

**State**:
- ✅ Repo scaffolded (you're reading this)
- ✅ Package layout matches the eventual extraction targets
- ✅ EXTRACTION-MAP names every source path
- ✅ Doctrine landed at [`freeside-modules-as-installables`](https://github.com/0xHoneyJar/loa-hivemind/blob/main/wiki/concepts/freeside-modules-as-installables.md)
- ⏳ Schemas remain in `loa-freeside`; consumers continue to import from there

**Why stage 1 first**: scaffolding the destination repo BEFORE extraction lets the operator + Jani align on shape. No code moves; no consumers break. The repo sits ready.

## Stage 2 — Hard cutover (per-package, when coordinated)

**Goal**: schemas physically extract; loa-freeside consumes via package import (or git URL fetch — pattern TBD per doctrine).

### Per-package sequence

For each row in `EXTRACTION-MAP.md`:

1. **Coordinate window**: Slack/issue Jani naming the package to extract. Confirm no in-flight work touches it.
2. **Move + tests**: copy the source file into `freeside-score/packages/<target>/`. Bring tests. Run them in this repo until green.
3. **Cross-repo import in loa-freeside**: rewire loa-freeside consumer to import from `freeside-score/packages/<target>/` (via npm package OR git path OR file fetch — depends on install pattern decided by then).
4. **Delete loa-freeside copy**: only after consumers verified. Keep a comment-stub pointing at `freeside-score` for one cycle.
5. **Verify**: `loa-freeside` still builds + deploys; `score-api` still satisfies the port; ruggy MCP still resolves tools.

### Order of extraction (lowest risk → highest)

| order | package | risk | gate |
|---|---|---|---|
| 1 | `packages/protocol/` (NATS event schemas + branded types) | low — schema-only, no impl coupling | jani window |
| 2 | `packages/ports/` (IScoreServiceClient interface) | low — interface only | after package 1 stable |
| 3 | `packages/adapters/score-service-client.ts` | medium — typed client; consumers rewire imports | after ports extracted |
| 4 | `packages/mcp-tools/` (manifest + tool specs) | low — net-new content | parallel; ruggy consumes |

Webhook schemas + RecentActivity port additions land as net-new in this repo; not extraction work.

## What loa-freeside looks like after the cutover

```
loa-freeside/
├── packages/
│   ├── core/
│   │   └── ports/        ← (no longer holds score-service.ts; imports from freeside-score)
│   ├── shared/
│   │   └── nats-schemas/ ← (no longer holds activity-event.ts etc.; imports)
│   └── adapters/
│       └── chain/        ← (no longer holds score-service-client.ts; imports)
├── apps/
│   ├── dashboard/        ← consumes freeside-score schemas for world-tile rendering
│   └── ...
└── themes/sietch/src/
    └── jobs/
        └── score-distribution.ts  ← imports IScoreServiceClient from freeside-score
```

## What changes for downstream consumers

| consumer | before | after |
|---|---|---|
| `loa-freeside/themes/sietch/src/jobs/score-distribution.ts` | `import { IScoreServiceClient } from '../../../packages/core/ports/score-service'` | `import { IScoreServiceClient } from 'freeside-score/ports'` (or equivalent install path) |
| `0xHoneyJar/score-api` (impl) | satisfies port defined inline | satisfies port from `freeside-score/ports`; depends on it explicitly |
| `0xHoneyJar/freeside-ruggy` | (in flight; never had access to the schemas) | imports MCP tool specs + NATS schemas from `freeside-score` from day 1 |
| `mibera-world` / world consumers | call score-api over network using ad-hoc types | type via `freeside-score/ports`, validate against `freeside-score/protocol` |

## Open coordination items

- [ ] Confirm extraction window with Jani (slack or issue)
- [ ] Decide install mechanism (npm vs git-url vs hybrid) per [`freeside-modules-as-installables`](https://github.com/0xHoneyJar/loa-hivemind/blob/main/wiki/concepts/freeside-modules-as-installables.md) — affects how downstream `import` paths look
- [ ] Confirm `score-api` rename plan (stays as `score-api` impl, or renames to `freeside-score-api`?) — per [[loa-org-naming-conventions]] attachment-prefix discipline, the impl could be renamed to track the schema repo's name
- [ ] First consumer to migrate post-extraction (suggested: ruggy — it's net-new, no legacy import path to break)
