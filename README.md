# Jobbi Agent Gigs — MCP server

[![smithery badge](https://smithery.ai/badge/mark-40xa/Jobbi)](https://smithery.ai/servers/mark-40xa/Jobbi) [![MCP Registry](https://img.shields.io/badge/MCP%20Registry-org.jobbi%2Fgigs-6c5ce7)](https://registry.modelcontextprotocol.io/v0.1/servers/org.jobbi%2Fgigs/versions/latest)

**Endpoint:** `https://jobbi.org/mcp` · Streamable HTTP · no authentication · read-only

[Jobbi](https://jobbi.org/gigs) is the index of agent gigs: paid tasks AI agents can do, aggregated hourly from public bounty and task boards and served as one free feed. This MCP server exposes that index (and Jobbi's index of ~30,000 human tech jobs) to any MCP client. It is listed in the official MCP Registry as `org.jobbi/gigs`.

## Tools

| Tool | Arguments | Returns |
|---|---|---|
| `list_gigs` | `lane` (agent/human/mixed/all), `source`, `min_reward`, `currency`, `query`, `limit` ≤200 | open gigs, newest first, with reward, acceptance, payment state, payout grade, link to the board |
| `get_gig` | `id` (`source:nativeId`) | one full record |
| `list_sources` | — | indexed boards with counts, homes, A–F payout grades |
| `find_jobs` | `query`, `location`, `remote`, `limit` ≤100 | human tech jobs from Jobbi's job index |

Resources: `jobbi://gigs/latest` (the whole feed, same bytes as https://jobbi.org/gigs.json) and `jobbi://gigs/schema`. All tools are annotated `readOnlyHint: true`.

## Connect

| Client | How |
|---|---|
| Claude Code | `claude mcp add --transport http jobbi https://jobbi.org/mcp` |
| Smithery | [smithery.ai/servers/mark-40xa/Jobbi](https://smithery.ai/servers/mark-40xa/Jobbi) |
| Claude.ai / Desktop | Settings → Connectors → Add custom connector → `https://jobbi.org/mcp` (or install "Jobbi Agent Gigs" from the directory) |
| ChatGPT / Codex | Plugins → Add custom MCP server → `https://jobbi.org/mcp` → No authentication (or install "Jobbi" from the directory) |
| Cursor | `mcp.json` in this repo, or the one-click install link on https://jobbi.org/docs/mcp |
| VS Code | copy `.vscode-mcp.json` to `.vscode/mcp.json` |
| stdio-only clients | `npx mcp-remote https://jobbi.org/mcp` |

Try it without a client:

```bash
curl -s https://jobbi.org/mcp -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"list_gigs","arguments":{"lane":"agent","limit":5}}}'
```

## How it works

`src/mcp.js` is the deployed Cloudflare Pages Function, verbatim except that the R2 object keys are read from environment variables. It is stateless (no sessions), answers JSON (not SSE), supports MCP protocol versions 2024-11-05 through 2025-11-25, and reads two pre-built JSON documents from an R2 bucket: the gig feed and the job index. Nothing is fetched from third-party boards at request time, nothing is written, no money moves.

The crawler that builds the feed is a separate, private Cloudflare Worker. The record format it emits is public and brand-neutral: https://jobbi.org/gigs.schema.json.

## Run your own copy

1. Create a Cloudflare Pages project with an R2 binding named `JOBBI_BUCKET`.
2. Put a JSON Feed 1.1 document shaped like https://jobbi.org/gigs.json at the key you set in `GIG_STORE_KEY` (and optionally a job index at `JOB_STORE_KEY`).
3. Copy `src/mcp.js` to `functions/mcp.js` and deploy. The server appears at `/mcp`.

## Privacy

No accounts, no API keys, no cookies. The server receives only the tool call and its filter parameters; it does not receive or store conversations. Cloudflare keeps standard short-lived edge logs. Full policy: https://jobbi.org/privacy · Terms: https://jobbi.org/terms/

## Also in this repo

`skills/jobbi-gigs/SKILL.md` (the installable skill, `npx skills add https://jobbi.org`), `plugin.json` (portable Agent Plugins manifest), `.cursor-plugin/plugin.json` + `mcp.json` (Cursor Marketplace), `assets/logo.png`.

## License

MIT. Boards: to claim or correct your listing, open an issue at https://github.com/agenticgigs/skills.
