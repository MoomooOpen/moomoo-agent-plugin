# moomoo MCP Plugin

All-in-one MCP plugin for moomoo — quotes, stock screening, news, and trading. Compatible with **Grok Build**, **Claude Code**, **Cursor**, **Codex**, and the **Agent Plugins** open standard.

## Directory Structure

```
moomoo-agent-plugin/
├── plugin.json                        # Agent Plugins open standard manifest
├── mcp.json                           # Cursor MCP config
├── .mcp.json                          # Codex / Claude Code MCP config
├── assets/
│   └── moomoo.png                     # Plugin icon
├── .claude-plugin/
│   ├── plugin.json                    # Claude Code plugin manifest
│   └── marketplace.json               # Claude Code marketplace registration
├── .cursor-plugin/
│   ├── plugin.json                    # Cursor plugin manifest
│   └── marketplace.json               # Cursor Team Marketplace registration
├── .codex-plugin/
│   └── plugin.json                    # Codex plugin manifest
├── .agents/plugins/
│   └── marketplace.json               # Codex marketplace registration
└── skills/
    ├── moomoo-mcp-quote/
    │   ├── SKILL.md                   # Quotes: prices / K-lines / order book / time & sales / sectors / watchlist
    │   └── reference/
    │       ├── quote-realtime.md      # Real-time quote reference
    │       ├── quote-tick.md          # Tick-by-tick / time-sharing reference
    │       └── quote-watchlist.md     # Watchlist / sector reference
    ├── moomoo-mcp-screen/
    │   ├── SKILL.md                   # Screening: stocks / warrants / options / IPO
    │   └── reference/
    │       ├── quote-screening.md     # Stock screening reference
    │       ├── quote-options.md       # Option screening reference
    │       └── quote-futures-warrants.md  # Futures / warrant reference
    ├── moomoo-mcp-news/
    │   ├── SKILL.md                   # Research: news / financials / valuation / shareholders / capital flow
    │   └── reference/
    │       ├── quote-news.md          # News / community / feeds reference
    │       ├── quote-financials.md    # Financial statements reference
    │       ├── quote-research.md      # Research / ratings reference
    │       ├── quote-shareholders.md  # Shareholder holdings reference
    │       ├── quote-capital.md       # Capital flow reference
    │       ├── quote-corporate-actions.md  # Corporate actions reference
    │       └── quote-short-broker.md  # Short selling / broker reference
    └── moomoo-mcp-account/
        ├── SKILL.md                   # Trading: real / simulated orders / positions / funds
        └── reference/
            ├── trading-real.md        # Real trading reference
            ├── trading-real-order.md  # Real order placement reference
            └── trading-sim.md         # Simulated trading reference
```

## Platform Compatibility

| Platform | Manifest | MCP Config | Marketplace |
|----------|----------|------------|-------------|
| **Grok Build** | Root `plugin.json` | `.mcp.json` | xAI Plugin Marketplace (after approval) |
| **Claude Code** | `.claude-plugin/plugin.json` | `.mcp.json` | `.claude-plugin/marketplace.json` |
| **Cursor** | `.cursor-plugin/plugin.json` | `mcp.json` | `.cursor-plugin/marketplace.json` |
| **Codex** | `.codex-plugin/plugin.json` | `.mcp.json` | `.agents/plugins/marketplace.json` |
| **Agent Plugins** | Root `plugin.json` | `mcp.json` | — |

Each platform reads its own manifest and MCP config independently — no conflicts. The `skills/` directory is shared across all platforms with an identical format.

## MCP Server

All platforms connect to the same remote MCP service:

```
https://mcp.moomoo.com/mcp
```

Protocol: Streamable HTTP (some clients display this as HTTP)

## Authentication, Permissions, and Network Access

The hosted MCP server uses an OAuth authorization-code flow with PKCE. Users authenticate on Moomoo-owned pages and authorize access for their own eligible account. Passwords and MFA codes are entered only on Moomoo pages; this repository contains no user credentials, API keys, OAuth tokens, executables, install scripts, or lifecycle hooks. The MCP client stores OAuth tokens according to that client's own credential-storage policy.

Runtime network endpoints:

- `https://mcp.moomoo.com/mcp` — MCP requests and OAuth protected-resource discovery
- `https://webapi.moomoo.com/oauth2/*` — OAuth client registration, authorization, token, and revocation operations
- `https://passport.moomoo.com` — Moomoo sign-in and MFA during authorization
- `https://open.moomoo.com` — product documentation and account authorization pages

The service advertises quote read/write, trade read/write, and account-scoped permissions. The authorization screen is the source of truth for the access requested in a particular client session. Do not continue if the requested access exceeds what you intend to use.

Depending on the tools used, requests may include security symbols and query parameters, or account identifiers, balances, positions, order details, and execution records. This data is sent to Moomoo's hosted MCP service to fulfill the user's request; it is not stored in this repository.

## Trading Safety and Availability

This plugin includes simulated trading and, for eligible users, real-money order placement, modification, cancellation, and confirmation. Trading can result in financial loss and is not investment advice. Product, market-data, and trading availability depend on region, account eligibility, subscriptions, and the OAuth permissions granted by the user.

Before every trading operation, the agent instructions require the client to identify whether the selected account is real or simulated, display the complete order details, and obtain a new explicit confirmation from the user. Blanket, historical, or inferred confirmation must not be reused. When the server returns `need_order_confirm=true`, the operation requires an additional risk-control confirmation before it takes effect. The server remains responsible for enforcing account permissions and risk controls.

## Installation

### Grok Build

After marketplace approval, open `/marketplace` in Grok Build and install `moomoo-mcp`. For local testing from a checked-out copy, start Grok Build with `--plugin-dir /path/to/moomoo-agent-plugin`.

### Claude Code

```bash
/plugin marketplace add MoomooOpen/moomoo-agent-plugin
```

Or submit via the [plugin directory submission](https://clau.de/plugin-directory-submission).

### Cursor

In Dashboard → Plugins → Add Marketplace, enter the repository URL:
```
https://github.com/MoomooOpen/moomoo-agent-plugin.git
```
Cursor automatically reads `.cursor-plugin/marketplace.json`.

### Codex

```bash
codex plugin marketplace add MoomooOpen/moomoo-agent-plugin
```

### Agent Plugins

Any tool supporting the [Agent Plugins](https://agent-plugins.org) standard can discover the plugin via the root `plugin.json`.

## Features

### Market Data (moomoo-mcp-quote)
- Real-time quotes, market snapshots
- Daily / weekly / monthly / minute K-lines
- Order book depth (LV1 / LV2)
- Time-sharing charts, tick-by-tick trades
- Market state, trading calendar
- Watchlist management, sector constituents

### Stock Screening (moomoo-mcp-screen)
- Multi-factor stock screening (valuation / price change / financials / technical patterns / holdings — 120+ dimensions)
- Warrant / CBBC screening
- Option screener
- IPO listings for HK / US / A-share / SG / MY markets

### Research & News (moomoo-mcp-news)
- News / announcements / research report search
- Economic calendar (hot events + search)
- Financial statements, revenue breakdown
- PE / PB / PS valuation analysis
- Analyst ratings, Morningstar reports
- Shareholder / institutional / insider holdings
- Capital flow, short selling data
- Option chain / volatility / exercise probability

### Account & Trading (moomoo-mcp-account)
- Real trading: place / modify / cancel orders with risk-control confirmation
- Simulated trading: full simulated trading workflow
- Account funds, position queries
- Multi-market support (HK / US / A-share / SG / JP / CA / Futures / Options)

## Supported Markets

| Code | Market |
|------|--------|
| HK | Hong Kong |
| US | United States |
| SH / SZ | China A-shares |
| SG | Singapore |
| JP | Japan |
| CA | Canada |
| AU | Australia |
| MY | Malaysia |
| KR | South Korea |

## How It Works

1. When the IDE / CLI starts, it scans the corresponding marketplace or plugin manifest
2. Discovers the moomoo-mcp plugin and pulls it via git from `https://github.com/MoomooOpen/moomoo-agent-plugin.git`
3. The user sees it in the plugin directory and can install / enable it
4. Once installed, loads the MCP config, connects to `https://mcp.moomoo.com/mcp`, and loads skills

## Marketplace Configuration

### Claude Code (.claude-plugin/marketplace.json)

- `plugins[].source` — Pulled via git URL: `https://github.com/MoomooOpen/moomoo-agent-plugin.git`
- Plugin is enabled via `/plugin install` or settings.json

### Codex (.agents/plugins/marketplace.json)

- `source: "git-subdir"` — Pulls the repo via git; `path: "./"` points to the repo root; `ref: "main"` specifies the branch
- `installation: "AVAILABLE"` — User can choose to install
- `authentication: "ON_INSTALL"` — Triggers OAuth authentication on install

### Cursor (.cursor-plugin/marketplace.json)

- `plugins[].source` — Pulled via git URL: `https://github.com/MoomooOpen/moomoo-agent-plugin.git`
- `metadata.pluginRoot: "."` — Plugin root is the repository root

## License

[MIT](LICENSE)
