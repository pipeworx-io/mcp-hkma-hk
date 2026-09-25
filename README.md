# mcp-hkma-hk

Hong Kong Monetary Authority (HKMA) public open API MCP. Keyless.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1679+ live data sources.

## Tools

| Tool | Description |
|------|-------------|
| `hkma_dataset` | Fetch ANY HKMA public dataset by its endpoint path (the part after https://api.hkma.gov.hk/public/). Returns the uniform envelope: data rows are under result.records, total under result.datasize. Use this for any HKMA series not covered by a convenience tool. Verified example paths: "market-data-and-statistics/daily-monetary-statistics/daily-figures-interbank-liquidity" (daily HIBOR/aggregate balance/TWI), "market-data-and-statistics/daily-monetary-statistics/daily-figures-monetary-base" (monetary base components), "market-data-and-statistics/monthly-statistical-bulletin/er-ir/hk-interbank-ir-daily" (HIBOR fixings by tenor), "market-data-and-statistics/monthly-statistical-bulletin/er-ir/er-eeri-daily" (HKD exchange rates vs major currencies), "market-data-and-statistics/monthly-statistical-bulletin/er-ir/hkd-fer-daily" (HKD forward points), "market-data-and-statistics/monthly-statistical-bulletin/er-ir/composite-ir" (composite interest rate, monthly), "market-data-and-statistics/monthly-statistical-bulletin/banking/customer-deposits-by-currency", "coin-cart-schedule" (requires lang). Browse more at apidocs.hkma.gov.hk. Dates YYYY-MM-DD; paginate with offset/pagesize. |
| `hkma_interbank_liquidity` | Daily interbank liquidity figures: overnight & 1-month HIBOR, aggregate balance (opening/closing), discount window base rate, the convertibility undertaking strong/weak-side rates (7.75/7.85), and the trade-weighted index (TWI). One record per business day, most recent first. Use from/to to window the dates. |
| `hkma_monetary_base` | Daily figures for the components of the Hong Kong Monetary Base: Certificates of Indebtedness, government notes & coins in circulation, aggregate balance (before/after discount window), and outstanding Exchange Fund Bills & Notes. One record per business day. Use from/to to window the dates. |
| `hkma_interbank_interest_rates` | HKD interbank offered rates (HIBOR) by tenor: overnight, 1-week, 1/3/6/9/12-month, one record per day (end_of_day). Source is the Monthly Statistical Bulletin er-ir series. Use from/to to window the dates. |
| `hkma_exchange_rates` | HKD market exchange rates vs major currencies (USD, GBP, JPY, CNY, EUR, AUD, CAD, SGD, etc.), one record per day (end_of_day). Source is the Monthly Statistical Bulletin er-ir er-eeri-daily series. Use from/to to window the dates. |

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "hkma-hk": {
      "url": "https://gateway.pipeworx.io/hkma-hk/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/hkma-hk/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1679+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/hkma_dataset \
  -H 'Content-Type: application/json' \
  -d '{"path":"market-data-and-statistics/daily-monetary-statistics/daily-figures-interbank-liquidity"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/hkma_dataset`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "hkma-hk": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-hkma-hk"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-hkma-hk
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Hkma Hk data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
