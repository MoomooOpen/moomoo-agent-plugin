# Simulated Trading Tool Reference

## Account Query

### sim_trade_account_list — Simulated Account List

Retrieve the simulated trading account list for the currently logged-in user. Accounts are automatically created by the backend on the first call.

**Parameters:** None (uid is forwarded via login state)

**Response `data.accounts[]`:**

| Field | Type | Description |
|-------|------|-------------|
| account_id | string | Business account ID (used as acc_id in subsequent simulated trading APIs) |
| broker_id | int | Broker ID |
| market_id | int | Market ID (see enum below) |
| intra_account_id | int | Internal account short ID |
| account_type | int | Account type |
| account_title | string | Account name (e.g., `"HK Simulated Account"`) |

**market_id enum:**
| Value | Meaning |
|-------|---------|
| 1 | Hong Kong stocks (HK) |
| 2 | US stocks (US) |
| 3 | US options (US_OPTION) |
| 9 | China Stock Connect (HKCC) |
| 18 | Canada (CA) |


---

## Funds and Positions

### sim_trade_cash_info — Simulated Account Funds

Retrieve fund details for a simulated trading account, including cash balance, frozen funds, buying power, position market value, and profit/loss.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| acc_id | string | Yes | Simulated account ID (path parameter) |

**Response `data`:**

| Field | Type | Description |
|-------|------|-------------|
| balance | string | Cash balance |
| hold | string | Frozen funds |
| max_power_long | string | Maximum buying power |
| total_asset | string | Net asset value |
| mv | string | Position market value |
| long_mv | string | Long market value |
| short_mv | string | Short market value |
| unrealized_profit | string/null | Unrealized profit/loss (null when no data) |
| realized_profit | string/null | Realized profit/loss (null when no data) |

**Special behavior:**
- All amount fields are string type
- `unrealized_profit`/`realized_profit` may be null (not empty string)


---

### sim_trade_position_list — Simulated Positions

Retrieve the position list for a simulated trading account (code, side, quantity, cost, market value, profit/loss).

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| acc_id | string | Yes | Simulated account ID (path parameter) |
| market | int | No | Market filter: 1=HK stocks, 100=US stocks, 3=US options, 4=China Stock Connect |

**Response `data.positions[]`:**

| Field | Type | Description |
|-------|------|-------------|
| pstn_id | string | Position ID |
| pstn_type | int | Position side: 0=long, 1=short |
| market | int | Market |
| symbol | string | Code (without market prefix, e.g., `00700`) |
| stock_name | string | Name |
| qty | string | Quantity |
| qty_avbl | string | Available quantity |
| cost_price | string | Cost price |
| buy_avg_price | string | Average buy price |
| cur_price | string | Current price |
| mv | string | Market value |
| profit | string | Profit/loss amount |
| profit_ratio | string | Profit/loss ratio (in decimal form, e.g., `-0.0446` means -4.46%) |

**Special behavior:**
- All amount/quantity fields are string type
- `profit_ratio` is in decimal form (not percentage)
- When `market` is omitted, all positions for the account are returned

---

## Trading Capacity

### sim_trade_max_buy_sell — Simulated Max Buy/Sell Quantity

Query the maximum buy/sell quantity for a specified security in simulated trading.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| acc_id | string | Yes | Simulated account ID (path parameter) |
| symbol | string | Yes | Code (without market prefix, e.g., `00700`) |
| order_type | int | Yes | Order type: 1=limit, 3=market |
| price | string | No | Price; required for limit orders (order_type=1) |
| order_id | string | No | Original order ID when modifying an order |

**Response `data`:**

| Field | Type | Description |
|-------|------|-------------|
| max_cash_buy_qty_round_lot | string | Cash buy quantity (round lot) |
| max_margin_buy_qty_round_lot | string/null | Margin buy quantity (round lot); null when not applicable |
| max_sell_qty_round_lot | string | Sell quantity (round lot) |
| max_sell_short_qty | string | Short sell quantity |
| max_buy_back_qty | string | Buy back quantity |
| required_im_long | string/null | Futures long initial margin; null when not applicable |
| required_im_short | string/null | Futures short initial margin; null when not applicable |

**Special behavior:**
- Buy/sell quantities are returned in round lots (conforming to the market's minimum trading unit)
- `max_margin_buy_qty_round_lot`/`required_im_long`/`required_im_short` return null when margin or futures are not applicable

---

## Place Order

### sim_trade_input_order — Simulated Order Placement

Place a simulated trading order. Limit orders require a price; when the order price exceeds the market price, immediate execution occurs. HK stock quantities must be multiples of the lot size.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| acc_id | string | Yes | Simulated account ID (path parameter) |
| market | int | Yes | Market (corresponds to market_id from the account list: 1=HK stocks, 100=US stocks, 3=US options, etc.) |
| symbol | string | Yes | Code (without market prefix, e.g., `00700`) |
| order_type | int | Yes | Order type: 1=limit, 3=market |
| order_side | int | Yes | Side: 1=buy, 2=sell, 3=short sell, 4=buy to cover |
| qty | string | Yes | Quantity |
| price | string | No | Price (required for limit orders, order_type=1) |
| text | string | No | Remark (max 100 bytes) |

**Response `data`:**

| Field | Type | Description |
|-------|------|-------------|
| order_id | string | New order ID |

---

## Modify Order

### sim_trade_modify_order — Simulated Order Modification

Modify the quantity or price of an unfilled order in simulated trading. Only orders with status=2 (submitted) can be modified.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| acc_id | string | Yes | Simulated account ID (path parameter) |
| order_id | string | Yes | Order ID (path parameter) |
| new_qty | string | No | New quantity (must be a multiple of the lot size) |
| new_price | string | No | New price |

> At least one of `new_qty` and `new_price` must be provided.

**Success response:**

| Field | Type | Description |
|-------|------|-------------|
| ret_code | int | Return code (0=success) |
| ret_msg | string | Return message |
| data.order_id | string | Order ID after modification |

**Error response:**

| Field | Type | Description |
|-------|------|-------------|
| ret_code | int | Non-zero error code |
| ret_msg | string | Error description |

**Special behavior:**
- Only orders with status=2 (submitted) can be modified
- `new_qty` must be a multiple of the lot size (conforming to the market's minimum trading unit)
- You can modify quantity only, price only, or both simultaneously

---

## Cancel Order

### sim_trade_cancel_order — Simulated Order Cancellation

Cancel an unfilled order in simulated trading. Only orders with status=2 (submitted) can be cancelled.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| acc_id | string | Yes | Simulated account ID (path parameter) |
| order_id | string | Yes | Order ID (path parameter) |

> The request body is an empty JSON `{}`.

**Success response:**

| Field | Type | Description |
|-------|------|-------------|
| ret_code | int | Return code (0=success) |
| ret_msg | string | Return message |
| data.order_id | string | Cancelled order ID |

**Error response:**

| Field | Type | Description |
|-------|------|-------------|
| ret_code | int | Non-zero error code |
| ret_msg | string | Error description |

**Special behavior:**
- Only orders with status=2 (submitted) can be cancelled
- The request body must be an empty JSON `{}` (Content-Type: application/json)
