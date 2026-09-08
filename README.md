# X Twitter Brand Presence Mapper MCP Server

[![Smithery](https://smithery.ai/badge/mambabuilt/mcp-x-brand-presence-mapper)](https://smithery.ai/servers/mambabuilt/mcp-x-brand-presence-mapper) [![Glama score](https://glama.ai/mcp/servers/mambalabsdev/mcp-x-brand-presence-mapper/badges/score.svg)](https://glama.ai/mcp/servers/mambalabsdev/mcp-x-brand-presence-mapper) [![MCP Registry](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fregistry.modelcontextprotocol.io%2Fv0%2Fservers%3Fsearch%3Dcom.mambabuilt%252Fmcp-x-brand-presence-mapper%26limit%3D1&query=%24.servers%5B0%5D._meta%5B%22io.modelcontextprotocol.registry%2Fofficial%22%5D.status&label=mcp%20registry&color=blue)](https://registry.modelcontextprotocol.io/v0/servers?search=com.mambabuilt/mcp-x-brand-presence-mapper&limit=1) [![npm version](https://img.shields.io/npm/v/@mambalabsdev/mcp-x-brand-presence-mapper)](https://www.npmjs.com/package/@mambalabsdev/mcp-x-brand-presence-mapper) [![npm downloads](https://img.shields.io/npm/dm/@mambalabsdev/mcp-x-brand-presence-mapper)](https://www.npmjs.com/package/@mambalabsdev/mcp-x-brand-presence-mapper) [![license](https://img.shields.io/github/license/mambalabsdev/mcp-x-brand-presence-mapper)](https://github.com/mambalabsdev/mcp-x-brand-presence-mapper/blob/main/LICENSE) [![mcpservers.org](https://img.shields.io/badge/mcpservers.org-listed-blue)](https://mcpservers.org/servers/mambalabsdev/mcp-x-brand-presence-mapper)

An MCP server that resolves a company domain to its X (Twitter) handle, profile URL and follower metrics. It wraps the Mamba Labs X Twitter Brand Presence Mapper actor on Apify and returns a Clay-ready flat JSON row to any MCP client.

## What's Inside

- [What it does](#what-it-does)
- [Quick start](#quick-start)
- [Prerequisites](#prerequisites)
- [Example prompts](#example-prompts)
- [Inputs](#inputs)
- [Output](#output)
- [Example output](#example-output)
- [Features](#features)
- [Full actor documentation](#full-actor-documentation)
- [Mamba Labs GTM Suite](#mamba-labs-gtm-suite)
- [License](#license)

## What it does

Give it a company domain and it returns that company's official X handle and profile URL, with the follower, following and post counts, verification status, bio and account creation date where X serves them. One flat row per company.

It runs keyless out of the box. X rate limits the public route aggressively, so on a large batch some rows come back with the handle and URL populated and the counts marked `not_extractable`. Supplying your own X API bearer token removes that limit and returns full metrics at any batch size. The key raises effectiveness rather than unlocking the tool.

A guessed handle that fails the identity check is reported as `identity_mismatch` rather than returned as the company's. All of the lookup runs on Apify. This package is a thin client that calls the actor and hands back the result unchanged.

## Quick start

You need Node.js 18 or newer and an Apify account with an API token.

Add this to your Claude Desktop config:

```json
{
  "mcpServers": {
    "mamba-x-brand-presence-mapper": {
      "command": "npx",
      "args": ["-y", "@mambalabsdev/mcp-x-brand-presence-mapper"],
      "env": {
        "APIFY_TOKEN": "your-apify-token"
      }
    }
  }
}
```

Get your token at https://console.apify.com/account/integrations, paste it in, and restart Claude Desktop. The `map_x_brand_presence` tool will be available.

## Prerequisites

- Node.js 18 or newer
- An Apify account with an API token
- Optional: your own X API v2 bearer token, free to create at developer.x.com, if you want full metrics on a large batch

## Example prompts

- "Find the X account for shopify.com and give me its follower count."
- "What is the X handle for stripe.com, and when was the account created?"
- "Resolve the X profile URL for figma.com without fetching follower counts."
- "Look up the X handle Shopify and return the bio and post count."

## Inputs

- `company_domain` (optional): bare company domain, for example `shopify.com`. Supply this or a handle. With a domain the actor runs full discovery; with a handle it skips straight to the fetch.
- `company_name` (optional): improves search accuracy and is what the identity gate checks a discovered profile against, so supplying it reduces wrong matches.
- `handle` (optional): the X handle with or without the leading `@`, for example `Shopify`. Supplying it skips discovery and goes straight to the fetch.
- `includeFollowerCounts` (optional): when true (the default) the profile page is fetched and the counts are extracted. Set false to resolve the profile URL only, which is cheaper and needs no proxy.
- `skipCache` (optional): when false (the default) a successful lookup is cached for seven days and reused. Set true to force a fresh fetch.
- `xApiBearerToken` (optional): your own X API v2 bearer token, free to create at developer.x.com. Without one the tool still resolves handles, profile URLs and follower counts, but X rate limits the public route so some rows in a large batch return `not_extractable` instead of counts. Your token is used for your run only, is never stored, and is never shared with another run.

Supply either `company_domain` or `handle`.

## Output

The tool returns the actor's flat JSON row for the company, with 18 snake_case fields and no nested objects. Read `x_status` first: `ok`, `not_found`, `not_extractable` and `identity_mismatch` are different answers and the wrapper never collapses them. `x_fetch_route` says which route produced the counts and `x_discovery` says how the handle was found. See the Apify Store page for the full output schema.

## Example output

```json
{
  "degraded": false,
  "degradation_reason": null,
  "company_domain": "shopify.com",
  "company_name": "Shopify",
  "x_url": "https://x.com/Shopify",
  "x_handle": "Shopify",
  "x_followers": 452243,
  "x_followers_exact": true,
  "x_following": 3423,
  "x_tweet_count": 37167,
  "x_verified": false,
  "x_display_name": "Shopify",
  "x_bio": "The entrepreneurship company",
  "x_created_at": "2008-11-03T18:33:14.000Z",
  "x_fetch_route": "syndication",
  "x_discovery": "homepage_sameas",
  "x_status": "ok",
  "run_date": "2026-08-22T19:23:45.055Z"
}
```

## Features

- Resolves an official X handle and profile URL starting from a company domain
- Follower, following and post counts, plus verification status
- Account creation date, bio and display name
- Runs keyless, with your own X API key available for scale
- Domain first discovery from the homepage `sameAs` markup
- The fetch route is reported per row in `x_fetch_route`
- 18 flat snake_case fields, one row per company

## Full actor documentation

This server is a thin client and holds no lookup logic. For the complete input and output reference, pricing, and run history, see the Apify Store page:

https://apify.com/mambalabs/x-brand-presence-mapper

---

## Mamba Labs GTM Suite

This server is one of the Mamba Labs GTM Suite MCP servers. Every actor in the suite takes a domain or a company and returns one flat row, so they stack in the same Clay table without reshaping anything. The actor behind this server is the X Twitter Brand Presence Mapper, immutable Apify actor ID `oLOgadhHUkNAyDA3w`.

> Built by [Mamba Labs](https://github.com/mambalabsdev) | [npm](https://www.npmjs.com/org/mambalabsdev) | [Apify Store](https://apify.com/mambalabs)

## License

MIT

Built by Mamba Labs. https://apify.com/mambalabs
