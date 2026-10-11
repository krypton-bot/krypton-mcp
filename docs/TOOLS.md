# Krypton Bot MCP: tool reference

Every tool the Krypton Bot MCP server exposes to trade on Polymarket, with its parameters. Generated from the live server schemas; `tools/list` returns the same data as JSON Schema.

- Prediction-market prices are 0 to 1 (0.62 = 62% chance).
- Perps sizes are USD margin; notional = margin x leverage.
- `*` = required.
- Orders that break your limits, hit the kill switch or come from a blocked region are refused with a readable error.

### Perps

#### `getInstruments`

List tradable Polymarket Perps instruments with id, symbol, category, max leverage and min notional.

No parameters.

> Try: "List every perp I can trade with its max leverage."

#### `getTicker`

Live mark, index, last price, open interest and funding rate for one instrument.

| Parameter | Type | Details |
|---|---|---|
| `iid` * | `integer` | Perps instrument_id (see getInstruments) · above 0 |

> Try: "What is BTC trading at and what is the funding rate?"

#### `getBook`

Top of the order book (10 levels per side) with spread.

| Parameter | Type | Details |
|---|---|---|
| `iid` * | `integer` | Perps instrument_id (see getInstruments) · above 0 |

> Try: "Show me the ETH order book and the spread."

#### `getCandles`

Recent OHLCV candles. Sparse: bars only exist when trades happened.

| Parameter | Type | Details |
|---|---|---|
| `iid` * | `integer` | Perps instrument_id (see getInstruments) · above 0 |
| `interval` | `"1m" | "5m" | "15m" | "30m" | "1h" | "4h" | "1d"` | default "15m" |
| `count` | `integer` | 5 to 200 · default 60 |

> Try: "Pull 1h candles for SOL over the last two days."

#### `placeOrder`

Open or add to a position. Size by USD margin (notional = usdMargin x leverage). Market fills now; limit rests at price.

| Parameter | Type | Details |
|---|---|---|
| `iid` * | `integer` | Perps instrument_id (see getInstruments) · above 0 |
| `side` * | `"long" | "short"` | Required |
| `usdMargin` * | `number` | above 0 |
| `leverage` * | `integer` | min 1 |
| `type` | `"market" | "limit"` | default "market" |
| `marginType` | `"isolated" | "cross"` | default "isolated" |
| `price` | `number` | Required for limit orders · above 0 |
| `reason` * | `string` | One line why, shown to the user · up to 300 characters |

> Try: "Go long BTC with $50 margin at 3x."

#### `closePosition`

Close the whole position on an instrument at market (reduce-only).

| Parameter | Type | Details |
|---|---|---|
| `iid` * | `integer` | Perps instrument_id (see getInstruments) · above 0 |
| `reason` * | `string` | up to 300 characters |

> Try: "Close my BTC position."

#### `cancelOrder`

Cancel one open order by id.

| Parameter | Type | Details |
|---|---|---|
| `orderId` * | `string` | Required |

> Try: "Cancel order 1842."

#### `cancelAll`

Cancel all open orders, optionally only on one instrument.

| Parameter | Type | Details |
|---|---|---|
| `iid` | `integer` | Perps instrument_id (see getInstruments) · above 0 |

> Try: "Cancel all my open ETH orders."

#### `setLeverage`

Set leverage for an instrument (live mode syncs it to the exchange).

| Parameter | Type | Details |
|---|---|---|
| `iid` * | `integer` | Perps instrument_id (see getInstruments) · above 0 |
| `leverage` * | `integer` | min 1 |

> Try: "Set SOL leverage to 5x."

### Prediction markets

#### `searchMarkets`

Polymarket prediction markets by 24h volume, optionally filtered by text. Returns outcomes with token ids and prices (0-1 = probability).

| Parameter | Type | Details |
|---|---|---|
| `query` | `string` | up to 100 characters |
| `limit` | `integer` | 1 to 30 · default 10 |

> Try: "Find the most traded election markets right now."

#### `getPredictionMarket`

One prediction market by outcome token id, with the live CLOB book (top 10 levels) for that outcome.

| Parameter | Type | Details |
|---|---|---|
| `tokenId` * | `string` | Prediction-market outcome token id (see searchMarkets) |

> Try: "Show the odds and order book for this outcome."

#### `buyOutcome`

Buy shares of a prediction-market outcome for a USD amount. Paper mode fills on the live CLOB book.

| Parameter | Type | Details |
|---|---|---|
| `tokenId` * | `string` | Prediction-market outcome token id (see searchMarkets) |
| `usd` * | `number` | above 0 |
| `limitPrice` | `number` | Never pay above this price · 0.01 to 0.99 |
| `reason` * | `string` | up to 300 characters |

> Try: "Buy $10 of Yes on that market."

#### `sellOutcome`

Sell shares of a prediction-market outcome you hold (default: all).

| Parameter | Type | Details |
|---|---|---|
| `tokenId` * | `string` | Prediction-market outcome token id (see searchMarkets) |
| `shares` | `number` | above 0 |
| `limitPrice` | `number` | Never sell below this price · 0.01 to 0.99 |
| `reason` * | `string` | up to 300 characters |

> Try: "Sell half of my Yes shares."

#### `getPredictionPortfolio`

Prediction-market positions (marked at live midpoint), value, unrealized and realized P/L, recent trades.

No parameters.

> Try: "How are my prediction-market positions doing?"

### Copy trading

#### `topTraders`

Polymarket leaderboard by profit for a window. Use to find traders worth copying.

| Parameter | Type | Details |
|---|---|---|
| `window` | `"DAY" | "WEEK" | "MONTH" | "ALL"` | default "WEEK" |
| `limit` | `integer` | 1 to 50 · default 10 |

> Try: "Who are the top 10 Polymarket traders this month?"

#### `getTraderTrades`

A trader's most recent prediction-market trades.

| Parameter | Type | Details |
|---|---|---|
| `wallet` * | `string` | Polymarket proxy wallet 0x… |
| `limit` | `integer` | 1 to 50 · default 20 |

> Try: "What has this wallet traded today?"

#### `followTrader`

Start copy trading a wallet: each of their trades is mirrored at pct% of their size, capped at maxUsd per trade.

| Parameter | Type | Details |
|---|---|---|
| `wallet` * | `string` | Polymarket proxy wallet 0x… |
| `pct` | `number` | 1 to 100 · default 10 |
| `maxUsd` | `number` | 1 to 10000 · default 50 |
| `label` | `string` | up to 60 characters |

> Try: "Copy this trader at 10% of their size, max $25 a trade."

#### `unfollowTrader`

Stop copy trading a wallet.

| Parameter | Type | Details |
|---|---|---|
| `wallet` * | `string` | Polymarket proxy wallet 0x… |

> Try: "Stop copying that wallet."

#### `listCopyBots`

Traders you currently copy, with size settings.

No parameters.

> Try: "Which traders am I copying?"

### Account and limits

#### `getPortfolio`

Account snapshot in the current mode (paper or live): equity, available margin, positions, open orders.

No parameters.

> Try: "What is my equity and what positions are open?"

#### `getFills`

Recent fills with realized PnL and fees.

| Parameter | Type | Details |
|---|---|---|
| `limit` | `integer` | 1 to 100 · default 20 |

> Try: "Show my last 20 fills with P/L."

#### `getLimits`

Owner risk limits and mode. Respect them; the server rejects orders outside them anyway.

No parameters.

> Try: "Am I in paper or live mode, and what are my limits?"

### Research

#### `webSearch`

Search the web for the latest news and facts (live, with sources). Use before prediction-market buys and news-driven trades. Cite result URLs in your comments.

| Parameter | Type | Details |
|---|---|---|
| `query` * | `string` | up to 200 characters |

> Try: "Search the latest news on the Fed decision."

#### `fetchPage`

Fetch a public web page as plain text (first 6000 chars).

| Parameter | Type | Details |
|---|---|---|
| `url` * | `string` | Required |

> Try: "Read this article and summarise it."

#### `comment`

Post a short one-line thought to the user's live AI panel (what you see, what you wait for). Use often.

| Parameter | Type | Details |
|---|---|---|
| `text` * | `string` | up to 240 characters |

> Try: "Post a note to my Krypton Bot panel."
