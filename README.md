# Texas Code — Texas Statutes by citation

Keyless. `PE` + `19.02` returns the murder statute.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1669+ live data sources.

## Why this took three attempts to find

`statutes.capitol.texas.gov` returns HTTP 200 with an identical 250,874-byte
Angular shell for **every** path, including nonsense ones — so no probe of that
host can ever produce a document or a miss. The statutes are not served from it.

The real location sits in an environment constant inside `chunk-7GRZWKYH.js`, a
chunk `index.html` never references (so a sweep of *preloaded* chunks misses it):

```
FileServerPath: https://tcss.legis.texas.gov/resources
```

The file server holds the same per-chapter HTML the old site used, at the same
path minus the `Docs/` prefix — and it 404s honestly.

## Files are per chapter; citations are per section

`19.02` lives inside `PE.19.htm` next to 19.01-19.06, so the chapter is fetched
and the section cut out of it, bounded by the next `Sec. N.NN.` heading.
Returning the chapter would hand back six statutes for one ask.

A missing chapter and a missing section in a real chapter are reported
differently.

## Data source

Texas Legislative Council. Texas statutes are public record.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "texas-code": {
      "url": "https://gateway.pipeworx.io/texas-code/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/texas-code/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1669+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/tx_statute \
  -H 'Content-Type: application/json' \
  -d '{"code":"PE","section":"19.02"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/tx_statute`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "texas-code": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-texas-code"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-texas-code
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Texas Code data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
