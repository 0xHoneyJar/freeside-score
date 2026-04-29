# packages/mcp-tools — MCP tool specs

Agent-callable surface. MCP tool manifests + per-tool JSON schemas that wrap the `freeside-score` port for consumption by agent runtimes (ruggy bot, future Freeside dashboard MCP, third-party agent integrations).

## Status: scaffolded; content NEW (not extraction)

Per [[score-vault]] §"Three additions to IScoreServiceClient": the MCP wrapper is net-new — score-api today doesn't expose an MCP surface. Authored from scratch as part of this repo's v0.

## Planned contents

| file | purpose |
|---|---|
| `manifest.json` | MCP server manifest (name, description, tools array) |
| `tools/get-ranked-holders.json` | Tool spec for ranked holder query (paginated, filterable by chain) |
| `tools/get-address-rank.json` | Tool spec for single-address rank lookup |
| `tools/get-recent-activity.json` | Tool spec for time-bounded activity stream (per RecentActivity port) |
| `tools/get-activity-summary.json` | Tool spec for digest queries ("what happened this week, grouped by factor") |
| `tools/check-action-history.json` | Tool spec for has-X-acted-Y-times queries |

## Pattern

Each tool spec follows MCP convention: name + description + inputSchema (JSON Schema) + outputSchema (references `../protocol/` for shape).

## Consumers

- `0xHoneyJar/freeside-ruggy` — primary MCP client; persona-bot uses these tools to answer queries about scoring activity
- Future Freeside dashboard MCP wrapper (per [[freeside-as-subway]] §"MCP wrapper")
- Third-party agents that want to query THJ scoring (operator-authorized via API key)

## Composition

- Wraps `freeside-score/packages/ports/` over MCP transport
- Validates I/O against `freeside-score/packages/protocol/` schemas
- Registered in ruggy's MCP server config as a remote tool source
