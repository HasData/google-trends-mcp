# Google Trends MCP Server

<!-- mcp-name: com.hasdata/google-trends -->

A hosted Model Context Protocol (MCP) server that gives Claude, Cursor, Windsurf and any other MCP client one Google Trends tool. Pull interest over time, interest by region, and the rising and top related queries and topics for any term, all as structured JSON, with no scraping library to keep alive and no Google account.

```
https://mcp.hasdata.com/api/mcp?apis=google_trends
```

[![Glama score](https://glama.ai/mcp/servers/HasData/google-trends-mcp/badges/score.svg)](https://glama.ai/mcp/servers/HasData/google-trends-mcp)
[![tool contract](https://github.com/HasData/google-trends-mcp/actions/workflows/contract.yml/badge.svg)](https://github.com/HasData/google-trends-mcp/actions/workflows/contract.yml)
[![MCP](https://img.shields.io/badge/MCP-remote%20%7C%20streamable%20HTTP-6366f1?style=flat-square)](https://modelcontextprotocol.io)
[![Tools](https://img.shields.io/badge/tools-1-10b981?style=flat-square)](#tools)
[![npm](https://img.shields.io/npm/v/@hasdata/google-trends-mcp?style=flat-square&logo=npm&label=npm&color=cb3837)](https://www.npmjs.com/package/@hasdata/google-trends-mcp)
[![PyPI](https://img.shields.io/pypi/v/hasdata-google-trends-mcp?style=flat-square&logo=pypi&logoColor=white&label=PyPI&color=3775a9)](https://pypi.org/project/hasdata-google-trends-mcp/)
[![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)

## Contents

- [What you need](#what-you-need)
- [Quick start](#quick-start)
- [Example prompts](#example-prompts)
- [Tools](#tools)
- [Errors and failure paths](#errors-and-failure-paths)
- [Pricing, free tier and limits](#pricing-free-tier-and-limits)
- [Tool selection](#tool-selection)
- [How it compares](#how-it-compares)
- [FAQ](#faq)
- [HasData links](#hasdata-links)
- [Development](#development)
- [Contributing](#contributing)
- [License](#license)

## What you need

An MCP client and a HasData API key from the [dashboard](https://app.hasdata.com/sign-up?utm_source=github&utm_medium=syndication&utm_campaign=google-trends-mcp), free to create with no card, and the trial covers about 200 calls at the 5-credit rate. This is a remote server, so the simplest path is a URL and an `x-api-key` header, with no container to run and no Google account anywhere in the flow. A client that only speaks stdio reaches it through a thin launcher, published as `@hasdata/google-trends-mcp` on npm and `hasdata-google-trends-mcp` on PyPI, shown below.

## Quick start

The server URL is the same for every client. We run it hands-on in Claude Code and Claude Desktop. The other blocks follow each client's own documented format for a remote server.

| Field | Value |
| :--- | :--- |
| URL | `https://mcp.hasdata.com/api/mcp?apis=google_trends` |
| Transport | HTTP, streamable |
| Auth header | `x-api-key: HASDATA_API_KEY` |

Clients with OAuth support can add the same URL as a connector and sign in without putting a key in a config file.

<details>
<summary><b>Claude Code</b></summary>

```bash
claude mcp add --transport http google-trends "https://mcp.hasdata.com/api/mcp?apis=google_trends" \
  --header "x-api-key: HASDATA_API_KEY"
```

</details>

<details>
<summary><b>Claude Desktop</b></summary>

Settings, then Connectors, then Add custom connector, then paste `https://mcp.hasdata.com/api/mcp?apis=google_trends` and sign in.

For the config-file route, Claude Desktop loads only local (stdio) servers, so it reaches a remote server through a stdio launcher. The `@hasdata/google-trends-mcp` package is that launcher, and it reads the key from the environment. Add this to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "google-trends": {
      "command": "npx",
      "args": ["-y", "@hasdata/google-trends-mcp"],
      "env": { "HASDATA_API_KEY": "YOUR_KEY" }
    }
  }
}
```

For Python instead of Node, swap the launcher for the PyPI package, which `uvx` runs without a manual install:

```json
{
  "mcpServers": {
    "google-trends": {
      "command": "uvx",
      "args": ["hasdata-google-trends-mcp"],
      "env": { "HASDATA_API_KEY": "YOUR_KEY" }
    }
  }
}
```

</details>

<details>
<summary><b>Cursor</b></summary>

`~/.cursor/mcp.json` for every project, or `.cursor/mcp.json` for one:

```json
{
  "mcpServers": {
    "google-trends": {
      "url": "https://mcp.hasdata.com/api/mcp?apis=google_trends",
      "headers": { "x-api-key": "HASDATA_API_KEY" }
    }
  }
}
```

</details>

<details>
<summary><b>Windsurf</b></summary>

`~/.codeium/windsurf/mcp_config.json`. Windsurf calls the field `serverUrl`, not `url`:

```json
{
  "mcpServers": {
    "google-trends": {
      "serverUrl": "https://mcp.hasdata.com/api/mcp?apis=google_trends",
      "headers": { "x-api-key": "HASDATA_API_KEY" }
    }
  }
}
```

</details>

<details>
<summary><b>VS Code</b></summary>

`.vscode/mcp.json` in the workspace:

```json
{
  "servers": {
    "google-trends": {
      "type": "http",
      "url": "https://mcp.hasdata.com/api/mcp?apis=google_trends",
      "headers": { "x-api-key": "HASDATA_API_KEY" }
    }
  }
}
```

</details>

## Example prompts

Prompts, not code. Paste one in and the agent picks the tool itself. Each is annotated with the calls it takes, because every successful call costs 5 credits.

> Chart interest in "cold brew coffee" in the US over the past 12 months and tell me which weeks it peaked.

*One call, 5 credits. The weekly series comes back in a single request.*

> For "cold brew coffee" in the US, give me the rising related queries and flag the ones marked Breakout.

*One call, 5 credits.*

> Compare interest in "cold brew" against "iced coffee" worldwide over five years and say which one is growing.

*One call, 5 credits. The tool takes several terms in one timeseries request.*

> Show me interest in "sunscreen" by US state over the past 90 days so I can see where demand is highest.

*One call, 5 credits. This is the interest-by-region view at state granularity.*

A comparison across terms rides in one `timeseries` call. Region breakdowns, related queries and related topics are each their own `dataType`, so a prompt that wants a chart plus its rising queries is two calls.

## Tools

One tool, read-only. The sample below is trimmed from a real call, and the numbers move as the trend moves. Read it as a shape. The tool name links to its endpoint reference, which carries the full parameter list.

The sample is the payload, not the whole response. A `tools/call` result carries one text block, and that text is itself JSON holding `url`, `status`, `text` and `json`, with the scraped data under `json`. From a raw JSON-RPC response the path is `result.content[0].text`, parsed, then `.json`. A chat client unwraps that for you and code talking to the endpoint directly does not.

### Get Google Trends data

[`hasdata_google_trends_search_getTrendsData`](https://docs.hasdata.com/apis/google-trends/search?utm_source=github&utm_medium=syndication&utm_campaign=google-trends-mcp)

Interest over time, by region, or the related queries and topics for a term.

| Parameter | Type | Required | Notes |
| :--- | :--- | :--- | :--- |
| `q` | string | yes | The search term. `timeseries` and `geoMap` take up to 5 comma-separated terms to compare, and a sixth is rejected with a 400 |
| `dataType` | string | | `timeseries` by default, plus `geoMap`, `relatedTopics` and `relatedQueries`. The two related types take a single term only |
| `date` | string | | A window such as `now 7-d`, `today 12-m`, `today 5-y` or `all`, or a custom `yyyy-mm-dd yyyy-mm-dd` range |
| `geo` | string | | A location code such as `US` or `US-CA`. Worldwide when empty |
| `region` | string | | Granularity for `geoMap` only: `country`, `region` (subregion), `dma` (metro) or `city`. The default depends on `geo`, `country` worldwide and finer once a `geo` is set |
| `cat` | string | | Category id to narrow the term. `0` is all categories |
| `gprop` | string | | The Google property: `images`, `news`, `froogle` (Shopping) or `youtube`. Web search when empty |
| `tz` | number | | Time-zone offset in minutes, default `420` (PDT). Shifts how hourly ranges are bucketed |

The response key depends on `dataType`. `timeseries` returns `interestOverTime.timelineData`, `geoMap` returns interest by region, and the related types return `relatedQueries` or `relatedTopics`, each split into `rising` and `top`. Read the key that matches the type you asked for.

`timeseries` (the default) returns a value from 0 to 100 for each point, both as a string and pre-parsed in `extractedValue`. The most recent point often carries `isPartial: true`, meaning the week is still filling in. Drop it before you compute a trend, or the last bar reads as a dip that is not real.

```json
{
  "interestOverTime": {
    "timelineData": [
      { "date": "Apr 12 – 18, 2026", "timestamp": "1775952000", "isPartial": false,
        "values": [{ "query": "cold brew coffee", "value": "100", "extractedValue": 100, "hasData": true }] },
      { "date": "Aug 23 – 29, 2026", "timestamp": "1787443200", "isPartial": true,
        "values": [{ "query": "cold brew coffee", "value": "44", "extractedValue": 44, "hasData": true }] }
    ]
  }
}
```

`relatedQueries` splits into `rising` and `top`. A rising entry reads as a percentage like `+300%`, or `Breakout` for a jump too large to score, and `extractedValue` gives the number behind it. A `Breakout` comes back with a sentinel `extractedValue` well above any real percentage, so sort on the string label, not on the raw number.

```json
{
  "relatedQueries": {
    "rising": [
      { "query": "organic cold brew coffee", "value": "+300%", "extractedValue": 300, "link": "https://trends.google.com/trends/explore?q=organic+cold+brew+coffee&date=today+12-m&geo=US" }
    ],
    "top": [
      { "query": "how to cold brew coffee", "value": "100", "extractedValue": 100, "link": "https://trends.google.com/trends/explore?q=how+to+cold+brew+coffee&date=today+12-m&geo=US" }
    ]
  }
}
```

The [endpoint reference](https://docs.hasdata.com/apis/google-trends/search?utm_source=github&utm_medium=syndication&utm_campaign=google-trends-mcp) lists every `geo`, `cat` and date format the tool accepts.

## Errors and failure paths

Your client almost never sees an HTTP error code from a tool call. The MCP layer answers 200 and puts the failure inside the result, with `isError` set to `true` and the reason as text. The agent reads a message where you might expect a status line.

**A wrong key surfaces as tool output, not as a failed connection.** `tools/list` accepts any non-empty key and returns the tool, so the client completes its handshake and shows green. The first tool call then comes back with `isError: true` and the text `HasData API error: 401 Unauthorized`. Watch for that string, because nothing earlier in the flow reports the problem.

**A missing key is the one real HTTP error.** Authorization runs before any tool, and the connection itself fails with 401. CORS headers are present, and a browser client reads the status and not an opaque network failure.

**An argument that breaks the tool's schema is rejected before it becomes a scrape.** The server answers with `isError: true` and the text `MCP error -32602: Input validation error`, naming the offending field. Nothing is fetched and nothing is charged.

**A term with too little search volume returns a successful result with the data arrays empty**, not an error. Google Trends has nothing to show for a rare term, and `requestMetadata.status` still reads `ok`. Test for the points before you chart them.

**An identifier the platform rejects returns 400** with `requestMetadata.status` set to `error`. An unknown `geo` or `cat` value is the usual way to see this.

Results that carry data also carry a `requestMetadata.id` worth quoting in support.

## Pricing, free tier and limits

Every Google Trends call costs **5 credits per successful call**. Response size does not change the price. A five-year weekly series costs the same as a single week.

The free trial is **1,000 credits over 30 days with no card**, which is 200 Google Trends calls. After that an active account keeps getting 100 credits topped up each day whenever its balance drops below 100, so a low-volume agent runs on the free tier indefinitely.

Paid plans start at **$49 a month** for 200,000 credits, which is 40,000 calls. The unit price falls with volume, from **$1.23 per 1,000 calls** on the entry plan to **$0.50** on Business, **$0.42** on Growth and **$0.37** on the largest [high-volume plans](https://hasdata.com/prices?utm_source=github&utm_medium=syndication&utm_campaign=google-trends-mcp).

Your plan also sets concurrency. The free trial allows 1 request at a time, Startup 15, Business 30, Growth 50, and the high-volume plans run from 200 to 1,500. Handle the overflow case defensively in anything unattended.

A request that comes back non-200 is not billed. A successful call that finds nothing is still a call.

## Tool selection

The `apis` query parameter decides which tools your agent sees. Fewer tools means less context spent on tool definitions, and fewer chances for the model to reach for the wrong one.

```
?apis=google_trends              the one tool in this repo
?apis=google_trends,google_serp  add Google search
?apis=google_trends,youtube      trends plus YouTube
```

The parameter takes provider names like `google_trends` and individual API names. Misspelled names are ignored. If every name is wrong the request fails with 400, and the body lists both what it did not recognise and every valid value. Drop the parameter and the same endpoint exposes all 57 HasData tools.

## How it compares

Google does not publish a public Trends API. The two common routes are the unofficial `pytrends` library, which reverse-engineers the same internal endpoints and breaks when Google changes them or rate-limits the caller, and rolling your own scraper. This server does that work behind a stable schema.

| | pytrends / DIY | This server |
| :--- | :--- | :--- |
| Official support | None, Google ships no Trends API | Maintained schema over the same data |
| Rate limits and 429s | Frequent and yours to manage | Handled behind the endpoint |
| Output | Pandas frames or raw payloads to reshape | Structured JSON, values pre-parsed |
| Setup | A Python environment and upkeep as it breaks | One key and one URL |
| Cost | Free, when it works | Paid past the trial, 5 credits a call |

If you already run `pytrends` at low volume and do not mind fixing it when it breaks, that stays the free answer. This server is for agents and pipelines that need the data to arrive the same shape every time.

## FAQ

### Is there an official Google Trends API?

No. Google has never shipped a public Trends API. Every option reads the same internal endpoints that the trends.google.com site uses. This one is maintained by HasData and returns the result as structured JSON.

### What is a Google Trends MCP server?

A server that exposes Google Trends as a tool an AI client can call. The client sends a tool call over the Model Context Protocol, the server fetches the data and returns structured JSON, and the model works with the result. This one exposes a single tool and runs remotely, so the client connects to a URL and starts no local process.

### Do the numbers mean absolute search volume?

No, and neither does Google Trends itself. The values are relative interest scaled 0 to 100 within the query, window and region you asked for. Use them for shape and comparison, not as a count of searches.

### Why is the last data point lower than the rest?

The most recent bucket is usually still filling in and comes back with `isPartial: true`. Drop it before computing a trend.

### Can I compare several terms at once?

Yes, in `timeseries` and `geoMap`. Pass the terms comma-separated in `q`. The related-query and related-topic types take a single term.

### Can I use this together with other HasData APIs?

Yes. The `apis` parameter takes a list, and `?apis=google_trends,google_serp` gives your agent Trends plus Google search. [Drop the parameter](#tool-selection) and you get everything.

### Compliance and personal data

HasData accesses publicly available data only. A platform's terms may restrict automated access, and you are responsible for your own compliance.

## HasData links

| | |
| :--- | :--- |
| Product page and request builder | [Google Trends API](https://hasdata.com/apis/google-trends-api?utm_source=github&utm_medium=syndication&utm_campaign=google-trends-mcp) |
| Server documentation | [MCP server docs](https://docs.hasdata.com/mcp-server?utm_source=github&utm_medium=syndication&utm_campaign=google-trends-mcp) |
| All 57 tools in one server | [HasData/hasdata-mcp](https://github.com/HasData/hasdata-mcp) |
| Client walkthroughs | [MCP clients and integrations](https://hasdata.com/integrations/mcp?utm_source=github&utm_medium=syndication&utm_campaign=google-trends-mcp) |
| Everything else we scrape | [Google Trends API and 54 more](https://hasdata.com/apis/?utm_source=github&utm_medium=syndication&utm_campaign=google-trends-mcp) |
| Plans and credit costs | [Plans and credit costs](https://hasdata.com/prices?utm_source=github&utm_medium=syndication&utm_campaign=google-trends-mcp) |
| Keys and usage | [HasData dashboard](https://app.hasdata.com?utm_source=github&utm_medium=syndication&utm_campaign=google-trends-mcp) |
| Node launcher on npm | [@hasdata/google-trends-mcp](https://www.npmjs.com/package/@hasdata/google-trends-mcp) |
| Python launcher on PyPI | [hasdata-google-trends-mcp](https://pypi.org/project/hasdata-google-trends-mcp/) |

## Development

This repository is configuration and documentation for a remote server. There is no build step and nothing to containerize.

The tests in `test/` assert the tool contract, the part that can break without a commit here. They check that `?apis=google_trends` returns exactly one tool, that it still declares its required parameter, that the name has not changed, and that the key in use is actually accepted. That last check calls the tool for real and costs 5 credits, which is the price of a canary that can fail for the right reason.

```bash
# macOS and Linux
HASDATA_API_KEY=your_key_here npm test

# Windows PowerShell
$env:HASDATA_API_KEY="your_key_here"; npm test
```

The same suite runs in CI on every push and once a week on a schedule, because the upstream tool list can change without anyone touching this repository. A failure means the tool list moved, the key stopped working, or the endpoint was unreachable, and the assertion message says which.

## Contributing

Corrections to the parameter table and the response sample are the most useful contribution, because those are the parts that drift. Include the call you made and the response you got. Pull requests from forks run the suite without a key, and the live checks skip instead of going red.

## License

MIT. See [LICENSE](LICENSE).
