# @pipeworx/mospi-in

India's Ministry of Statistics and Programme Implementation (MoSPI): the monthly Index of Industrial
Production (IIP) and Wholesale Price Index (WPI), with the latest month that has data, the series
names, and the index values. India only. Live call per request, nothing stored.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2015+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `mospi_in_options(product, base_year?)` — for `iip` or `wpi`: the base years (newest first), the
  series you can ask for, and the latest month with data.
- `mospi_in_search_series(query, product?, base_year?, limit?)` — find a WPI group, sub-group or item
  ("wheat", "mineral oils") or an IIP category ("capital goods") by words in its name.
- `mospi_in_get_data(product, series?, year?, month?, base_year?)` — index values (IIP also the
  year-on-year growth rate) for a month, a year, or the latest month. Omit year and month for the
  latest month that has data.

## Auth

Keyless.

## Data sources

- Ministry of Statistics and Programme Implementation (MoSPI), eSankhyiki data portal: Index of
  Industrial Production and Wholesale Price Index.

## Notes and traps

- Each product exists under several base years that cover different spans. Leaving the base year out
  of the upstream request selects an old one (WPI then answers from 2011-12, which stops in April
  2026, while 2022-23 runs to September 2026). This pack uses the newest base unless told otherwise
  and reports `base_year` on every response. Index levels from different bases are not comparable.
- A period with no data is an empty list with a 200 upstream. The pack reports it with the latest
  month that does have data.
- The Consumer Price Index is not served. Its 2012-base series ends in December 2025 and the 2024-base
  series is not returned by the route this pack uses, so serving CPI would return a stale month as if
  it were current.
- Results are paged by the upstream; the pack follows up to six pages and says when more rows match.
- The older WPI base names its columns differently from the newer one; rows are normalised.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "mospi-in": {
      "url": "https://gateway.pipeworx.io/mospi-in/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/mospi-in/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 2015+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/mospi_in_options \
  -H 'Content-Type: application/json' \
  -d '{"product":"wpi"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/mospi_in_options`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-mospi-in` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/mospi-in/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "mospi-in": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-mospi-in"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Mospi In data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
