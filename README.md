# Simple Product Feeds — agent skills

Operate your Shopify product feeds from Claude Code, Codex, Cursor, or any
MCP-capable agent: diagnose feed health, trace any row through the
transformation pipeline, fix issues with preview-first writes, publish, and
verify — across Google Merchant Center, Commission Junction, and more.

This repo packages the **feed-ops skill** (domain knowledge your agent
loads on demand) and the **remote MCP server config** for
[Simple Product Feeds](https://www.simpleproductfeeds.com) (14 tools, full
read + preview-first write surface, per-shop audit trail).

## Install

**Claude Code (plugin — recommended):**

```
/plugin marketplace add simpleproductfeeds/skills
/plugin install simple-product-feeds@simpleproductfeeds
```

Then set your API key (create one in the app under **Settings → AI Agents**):

```bash
export SPF_API_KEY=spf_live_sk_...
```

**Any MCP client (server only):**

```bash
claude mcp add --transport http simple-product-feeds \
  https://app.simpleproductfeeds.com/v1/mcp \
  --header "Authorization: Bearer $SPF_API_KEY"
```

Codex, Cursor, and `.mcp.json` snippets:
[docs → Connect an AI Agent](https://www.simpleproductfeeds.com/docs/api/agents).

## What's inside

- `skills/feed-ops/SKILL.md` — the operating loop (read → preview → confirm
  → publish → verify), connection setup, and the source-vs-output column
  vocabulary that trips up generic knowledge.
- `skills/feed-ops/rules/` — loaded on demand: pipeline precedence, Google
  Merchant Center field requirements, channel delivery quirks, proven
  workflow recipes.
- `.mcp.json` — remote server config (`${SPF_API_KEY}` reads from your
  environment; no secret is stored).

## Design: thin skill, live server

The MCP server is remote — every tool, description, and guardrail updates
the moment Simple Product Feeds deploys, with nothing to upgrade locally.
This skill stays deliberately thin and points at live sources of truth
(the server's own instructions, [llms.txt](https://www.simpleproductfeeds.com/llms.txt),
[docs as markdown](https://www.simpleproductfeeds.com/docs/api/agents.md),
[OpenAPI](https://app.simpleproductfeeds.com/v1/openapi.json)), so an older
install keeps working. Update the skill itself with
`/plugin marketplace update simpleproductfeeds`.

## Safety model

Reads are free; writes are guarded: rules require a preview token bound to
the exact rule text (the agent must show you the diff first), overrides and
cell edits are undoable, vague requests get proposals rather than changes,
and every write lands in your shop's audit trail.

## License

MIT
