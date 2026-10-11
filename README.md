<p align="center"><img src="assets/krypton-bot.svg" width="72" alt="Krypton Bot"></p>

# Krypton Bot MCP server

Trade on Polymarket from Claude, Cursor or any MCP client: perps, prediction markets and copy trading.
The [Krypton Bot](https://krypton.bot) MCP server lets your AI trade on Polymarket with the same tools Krypton Bot uses:
live markets, perps and prediction-market orders, and copy-trading bots.
Every call runs inside your own risk limits and kill switch.

- **Endpoint:** `https://krypton.bot/api/mcp` (Streamable HTTP, JSON-RPC 2.0)
- **Auth:** `Authorization: Bearer kr_...` (your personal token)
- **Docs:** https://krypton.bot/mcp

This repository holds client setup and docs. The server is hosted by Krypton Bot, so there is nothing to install or run.

## 1. Get your token

1. Sign in at [krypton.bot](https://krypton.bot) (free plan works).
2. Open **Dashboard › Krypton Bot › MCP server**.
3. Switch it on and copy your token.

The server is off until you switch it on. Rotate the token any time; old tokens stop working at once.

## 2. Add the server

**Claude Code**

```bash
claude mcp add --transport http krypton https://krypton.bot/api/mcp --header "Authorization: Bearer kr_..."
```

**Cursor** (`~/.cursor/mcp.json`): see [examples/cursor-mcp.json](examples/cursor-mcp.json).

**Claude Desktop** (`claude_desktop_config.json`, through [mcp-remote](https://www.npmjs.com/package/mcp-remote)): see [examples/claude-desktop.json](examples/claude-desktop.json).

Any other client that speaks Streamable HTTP works the same way: the URL plus the bearer header.

## 3. Ask your AI

> Find a mispriced election market and buy $10 of Yes.

New accounts trade in paper mode ($10,000 in paper USD) until you go live with your own wallet.

## Tools

25 tools. Full reference with every parameter, type, limit and an example prompt: **[docs/TOOLS.md](docs/TOOLS.md)** (also at [krypton.bot/mcp](https://krypton.bot/mcp#tools-heading)).

| Group | Tools |
| --- | --- |
| Perps | `getInstruments`, `getTicker`, `getBook`, `getCandles`, `placeOrder`, `closePosition`, `cancelOrder`, `cancelAll`, `setLeverage` |
| Prediction markets | `searchMarkets`, `getPredictionMarket`, `buyOutcome`, `sellOutcome`, `getPredictionPortfolio` |
| Copy trading | `topTraders`, `getTraderTrades`, `followTrader`, `unfollowTrader`, `listCopyBots` |
| Account | `getPortfolio`, `getFills`, `getLimits` |
| Research | `webSearch`, `fetchPage`, `comment` |

Call `tools/list` for the live JSON schemas.

### Example workflows

**Prediction market trade**

> Find the most traded crypto market this week, check the latest news, and buy $10 of the side you favour if the odds look wrong.

The AI calls `searchMarkets` → `webSearch` / `fetchPage` → `getPredictionMarket` (odds and book) → `buyOutcome`.

**Perps trade with a stop in mind**

> Check BTC funding and the 1h trend, then go long $50 at 3x if momentum is up. Close it if it drops 2%.

`getTicker` → `getCandles` → `getLimits` → `placeOrder`, later `closePosition`.

**Copy a top trader**

> Show me the top 5 Polymarket traders this month and copy the best one at 10% of their size, max $25 per trade.

`topTraders` → `getTraderTrades` → `followTrader`. Stop any time with `unfollowTrader`.

### Errors

Tools return an error message instead of failing silently, so your AI can explain it: limit exceeded (size, leverage, daily loss), kill switch on, live trading not connected, region blocked, unknown instrument or token, or rate limit reached.

## Plans

| | Free (paper) | Pro |
| --- | --- | --- |
| Rate limit | 20 requests / minute | 120 requests / minute |
| Trading | Paper | Paper and live |
| Copy bots | 1 active | Unlimited |

## Security

- Token-only access: browser sessions and cookies are not accepted on the MCP endpoint.
- Each token is scoped to one account and can only see and act on that account.
- Rotating or switching off MCP revokes every existing token immediately.
- Orders go through the same risk limits as the app. The server can never withdraw funds.
- Live trades sign with your own wallet. Krypton Bot never holds your money.

Found a security issue? Email hello@krypton.bot rather than opening a public issue.

## License

MIT
