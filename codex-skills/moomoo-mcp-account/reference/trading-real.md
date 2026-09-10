# Real Trading — Account/Funds/Positions/Orders/Fills


### account_authorized_trd_accs — Authorized Account List

Retrieve the list of all business accounts that the currently logged-in user is authorized to operate.

**Parameters:** None

**Response envelope:**
| Field | Type | Description |
|-------|------|-------------|
| s | string | Status: `"ok"`=success, `"error"`=failure |
| d.accounts[] | array | Authorized account list (on success) |
| errcode | int | Error code (on failure) |
| errmsg | string | Error message (on failure) |

**accounts[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| account_id | string | Business account ID (used as acc_id in subsequent trading APIs) |
| security_firm | string | Broker identifier (e.g., `FUTUINC`) |
| enable_market | int[] | Supported trading market list (see enum below) |
| acc_type | string | Account type: `cash`=cash account, `margin`=margin account |
| univs_account_card_number | string | Universal account card number (16 digits) |
| account_card_number | string | Business account card number (16 digits) |

**enable_market enum:**
| Value | Meaning | Value | Meaning |
|-------|---------|-------|---------|
| 1 | Hong Kong stocks | 9 | Forex |
| 2 | US stocks | 10 | Bonds |
| 4 | China Stock Connect | 11 | Malaysia |
| 5 | Futures | 12 | Canada |
| 6 | Singapore | 14 | Funds |
| 7 | Cryptocurrencies | 15 | Japan |
| 8 | Australia | 16 | Structured notes |
| 17 | Event-driven contracts | 18 | South Korea |

**Error codes:**
| errcode | Trigger condition | Recommended action |
|---------|-------------------|-------------------|
| -1200 | General error (see errmsg for details) | Handle based on errmsg content |

## Funds and Positions

### account_funds — Account Funds

Retrieve trading account fund details, including buying power, total assets, cash, market value, profit and loss, margin, etc.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| acc_id | string | Yes | Account ID (path parameter) |
| currency | string | Yes | Settlement currency: HKD/USD/CNH/JPY/SGD/KRW; only effective for futures accounts and comprehensive securities accounts, ignored for other account types |

**Response envelope:**
| Field | Type | Description |
|-------|------|-------------|
| s | string | Status: `"ok"`=success, `"error"`=failure |
| d | object | Fund data (on success) |
| errcode | int | Error code (on failure) |
| errmsg | string | Error message (on failure) |

**Response fields (within d):**

| Field | Type | Description |
|-------|------|-------------|
| power | string | Maximum buying power |
| max_power_short | string | Short selling buying power |
| total_assets | string | Total net assets |
| securities_assets | string | Securities assets |
| funds_assets | string | Fund assets |
| cash | string | Cash |
| market_val | string | Securities market value |
| long_mv | string | Long market value |
| short_mv | string | Short market value |
| pending_asset | string | Pending assets |
| frozen_cash | string | Frozen funds |
| max_withdrawal | string | Maximum withdrawable amount |
| currency | string | Query currency |
| available_funds | string | Available funds |
| unrealized_pl | string | Unrealized profit/loss |
| realized_pl | string | Realized profit/loss |
| risk_status | string | Risk status: NONE/LEVEL1~LEVEL9 |
| initial_margin | string | Initial margin |
| margin_call_margin | string | Margin call margin |
| maintenance_margin | string | Maintenance margin |
| hk_cash | string | HKD cash |
| hk_avl_withdrawal_cash | string | HKD withdrawable |
| hk_net_cash_power | string | HKD cash buying power |
| us_cash | string | USD cash |
| us_avl_withdrawal_cash | string | USD withdrawable |
| us_net_cash_power | string | USD cash buying power |
| jp_cash | string | JPY cash |
| jp_avl_withdrawal_cash | string | JPY withdrawable |
| jp_net_cash_power | string | JPY cash buying power |
| cn_cash | string | CNY cash |
| cn_avl_withdrawal_cash | string | CNY withdrawable |
| cn_net_cash_power | string | CNY cash buying power |
| sg_cash | string | SGD cash |
| sg_avl_withdrawal_cash | string | SGD withdrawable |
| sg_net_cash_power | string | SGD cash buying power |
| kr_cash | string | KRW cash |
| kr_avl_withdrawal_cash | string | KRW withdrawable |
| kr_net_cash_power | string | KRW cash buying power |

**Special behavior:**
- The `currency` parameter is only effective for futures accounts and comprehensive securities accounts; ignored for other account types
- All amount fields in the response (except those explicitly labeled with currency such as `hk_cash`/`us_cash`, etc.) are converted based on the requested `currency`
- All amount fields are string type (preserving decimal precision)

**Error codes:**
| errcode | Trigger condition | Recommended action |
|---------|-------------------|-------------------|
| -1200 | General error | Handle based on errmsg content |

---

### account_positions — Real Positions

Retrieve the position list for a specified trading account.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| acc_id | string | Yes | Account ID (path parameter) |
| code | string | No | Filter by code; for futures, pass the contract code including month — filtering by main contract code is not supported |
| pl_ratio_min | string | No | P&L ratio lower bound; pass `10` for >= +10% |
| pl_ratio_max | string | No | P&L ratio upper bound; pass `20` for <= +20% |

**Response envelope:**
| Field | Type | Description |
|-------|------|-------------|
| s | string | Status: `"ok"`=success, `"error"`=failure |
| d | array | Position list (on success) |
| errcode | int | Error code (on failure) |
| errmsg | string | Error message (on failure) |

**d[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| position_side | string | Position side: `NONE` (unknown) / `LONG` (long, default) / `SHORT` (short) |
| code | string | Security code |
| stock_name | string | Security name |
| qty | string | Position quantity |
| can_sell_qty | string | Sellable quantity |
| currency | string | Trading currency |
| nominal_price | string | Market price |
| cost_price | string | Diluted cost (securities account) / Average open price (futures account) |
| cost_price_valid | bool | Whether cost price is valid |
| market_val | string | Market value |
| pl_ratio | string | P&L ratio |
| pl_ratio_valid | bool | Whether P&L ratio is valid |
| pl_val | string | P&L amount |
| pl_val_valid | bool | Whether P&L amount is valid |
| today_pl_val | string | Today's P&L |
| today_trd_val | string | Today's turnover |
| today_buy_qty | string | Today's total buy quantity |
| today_buy_val | string | Today's total buy amount |
| today_sell_qty | string | Today's total sell quantity |
| today_sell_val | string | Today's total sell amount |
| unrealized_pl | string | Unrealized profit/loss |
| realized_pl | string | Realized profit/loss |

**Special behavior:**
- When `code` is omitted, all positions are returned
- All amount fields are string type
- When validity flags (`*_valid`) are false, the corresponding values are unreliable

**Error codes:**
| errcode | Trigger condition | Recommended action |
|---------|-------------------|-------------------|
| -1200 | General error | Handle based on errmsg content |

---

## Order Queries

### account_orders_active — Current Active Orders

Retrieve the list of uncompleted orders for a specified account. Uncompleted orders include: any unfinished orders, as well as orders filled or cancelled within the past 24 hours.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| acc_id | string | Yes | Account ID (path parameter) |
| trd_market | string | Yes | Trading market: HK/US/HKCC/SG/CA/FUTURES/JP/KR |
| page_flag | string | Yes | Pagination cursor; pass empty string for the first request, then pass the `page_flag` returned by the server |
| page_size | int | No | Page size, default 50, range 10~100 |

**Response envelope:**
| Field | Type | Description |
|-------|------|-------------|
| s | string | Status: `"ok"`=success, `"error"`=failure |
| d | object | Data (on success) |
| errcode | int | Error code (on failure) |
| errmsg | string | Error message (on failure) |

**Fields within d:**

| Field | Type | Description |
|-------|------|-------------|
| orders[] | array | Order list (default 50 entries, sorted by time descending) |
| page_flag | string | Next page pagination cursor |
| completed | bool | When `true`, all orders have been returned; stop paginating |

**orders[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| order_id | string | Order ID |
| code | string | Security code (with market prefix, e.g., `US.AAPL`) |
| stock_name | string | Security name |
| security_type | string | Security type (e.g., `STOCK`) |
| side | string | Side: BUY/SELL/SELL_SHORT/BUY_BACK |
| order_type | string | Order type: LIMIT/MARKET/STOP/STOP_LIMIT, etc. |
| order_status | string | Order status (e.g., `FILLED_ALL`/`CANCELLED`/`PENDING`, etc.) |
| qty | string | Order quantity |
| price | string | Order price |
| aux_price | string | Auxiliary price (e.g., trigger price) |
| dealt_qty | string | Filled quantity |
| dealt_avg_price | string | Average fill price |
| currency | string | Currency |
| time_in_force | string | Time in force: DAY/GTC |
| session | string | Trading session (e.g., `RTH`) |
| create_time | int64 | Creation time (microsecond timestamp) |
| updated_time | int64 | Last updated time (microsecond timestamp) |
| last_err_msg | string | Latest error message (empty if none) |
| remark | string | Remark (empty if none) |
| trail_type | string | Trailing type (e.g., `NONE`) |
| trail_value | string | Trailing value |
| trail_spread | string | Trailing spread |

**Pagination behavior:**
- Pass empty string for `page_flag` on the first request
- Pass the server-returned `page_flag` for subsequent requests
- Stop paginating when `completed=true`

**Error codes:**
| errcode | Trigger condition | Recommended action |
|---------|-------------------|-------------------|
| -1200 | General error | Handle based on errmsg content |

### account_orders_history — Historical Orders

Retrieve the historical order list for a specified account.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| acc_id | string | Yes | Account ID (path parameter) |
| trd_market | string | Yes | Trading market: HK/US/HKCC/SG/CA/FUTURES/JP/KR |
| page_flag | string | Yes | Pagination cursor; pass empty string for the first request, then pass the `page_flag` returned by the server |
| start | uint64 | No | Creation time start (microsecond timestamp) |
| end | uint64 | No | Creation time end (microsecond timestamp), must be later than start |
| code | string | No | Filter by code; omit to return all |
| page_size | int | No | Page size, default 50, range 10~100 |

**start/end combination rules:**
| start | end | Behavior |
|-------|-----|----------|
| >0 | >0 | Use the specified start and end times |
| 0 | >0 | start defaults to 90 days before end |
| >0 | 0 | end defaults to 90 days after start |
| 0 | 0 | Defaults to querying the last 90 days |

**Response `d`:**

| Field | Type | Description |
|-------|------|-------------|
| orders[] | array | Order list |
| page_flag | string | Next page pagination cursor |
| completed | bool | When `true`, all orders have been returned; stop paginating |

**orders[] elements:** Same structure as orders returned by `account_orders_active` (including order_id/code/stock_name/security_type/side/order_type/order_status/qty/price/aux_price/dealt_qty/dealt_avg_price/currency/time_in_force/session/create_time/updated_time/last_err_msg/remark/trail_type/trail_value/trail_spread)

### account_orders_detail — Order Details

Batch retrieve detailed information for specified orders. All orders in the same request must belong to the same exchange.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| acc_id | string | Yes | Account ID (path parameter) |
| order_ids | string[] | Yes | Order ID list, up to 50 (request body) |
| exchange | string | Yes | Exchange identifier (request body); all orders in the same request must belong to the same exchange |

**Response `d[]`:** Order details array; each element has the same structure as orders[] elements from `account_orders_active` (including order_id/code/stock_name/security_type/side/order_type/order_status/qty/price/aux_price/dealt_qty/dealt_avg_price/currency/time_in_force/session/create_time/updated_time/last_err_msg/remark/trail_type/trail_value/trail_spread)

**Error codes:**
| errcode | Trigger condition | Recommended action |
|---------|-------------------|-------------------|
| -1200 | General error | Handle based on errmsg content |

---

## Fill Records

### account_order_fills_today — Today's Fills

Retrieve today's fill records for a specified account.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| acc_id | string | Yes | Account ID (path parameter) |
| trd_market | string | Yes | Trading market: HK/US/HKCC/SG/CA/FUTURES/JP/KR |
| page_flag | string | Yes | Pagination cursor; pass empty string for the first request, then pass the `page_flag` returned by the server |
| page_size | int | No | Page size, default 50, range 10~100 |

**Response `d`:**

| Field | Type | Description |
|-------|------|-------------|
| order_fills[] | array | Fill records list (default 50 entries, sorted by time descending) |
| page_flag | string | Next page pagination cursor |
| completed | bool | When `true`, all records have been returned; stop paginating |

**order_fills[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| deal_id | string | Fill ID |
| order_id | string | Associated order ID |
| code | string | Security code (with market prefix, e.g., `US.AAPL`) |
| stock_name | string | Security name |
| trd_side | string | Trading side (e.g., `BUY`/`SELL`) |
| qty | string | Fill quantity |
| price | string | Fill price |
| create_time | int64 | Creation time (microsecond timestamp) |
| updated_time | int64 | Updated time (microsecond timestamp) |
| counter_broker_id | int | Counterparty broker ID |
| counter_broker_name | string | Counterparty broker name |
| status | string | Fill status (e.g., `OK`) |

**Error codes:**
| errcode | Trigger condition | Recommended action |
|---------|-------------------|-------------------|
| -1200 | General error | Handle based on errmsg content |

### account_fills_history — Historical Fills

Retrieve the historical fill records for a specified account.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| acc_id | string | Yes | Account ID (path parameter) |
| trd_market | string | Yes | Trading market: HK/US/HKCC/SG/CA/FUTURES/JP/KR |
| page_flag | string | Yes | Pagination cursor; pass empty string for the first request, then pass the `page_flag` returned by the server |
| start | uint64 | No | Updated time start (microsecond timestamp) |
| end | uint64 | No | Updated time end (microsecond timestamp), must be later than start |
| code | string | No | Filter by code; omit to return all |
| page_size | int | No | Page size, default 50, range 10~50 |

**start/end combination rules:**
| start | end | Behavior |
|-------|-----|----------|
| >0 | >0 | Use the specified start and end times |
| 0 | >0 | start defaults to 90 days before end |
| >0 | 0 | end defaults to 90 days after start |
| 0 | 0 | Defaults to querying the last 90 days |

**Response `d`:**

| Field | Type | Description |
|-------|------|-------------|
| order_fills[] | array | Fill records list (default 50 entries, sorted by time descending) |
| page_flag | string | Next page pagination cursor |
| completed | bool | When `true`, all records have been returned; stop paginating |

**order_fills[] elements:** Same structure as order_fills[] elements from `account_order_fills_today` (including deal_id/order_id/code/stock_name/trd_side/qty/price/create_time/updated_time/counter_broker_id/counter_broker_name/status)

**Error codes:**
| errcode | Trigger condition | Recommended action |
|---------|-------------------|-------------------|
| -1200 | General error | Handle based on errmsg content |

---

## Trading Capacity

### account_trading_info — Max Buy/Sell Quantity

Query the maximum buy/sell quantity for a specified account, or query the maximum modifiable quantity for a specified order.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| acc_id | string | Yes | Account ID (path parameter) |
| code | string | Yes | Code, format `Exchange.Code` (e.g., `US.AAPL`, `SEHK.00700`) |
| order_type | string | Yes | Order type: LIMIT/MARKET/AUCTION/AUCTION_LIMIT/STOP/STOP_LIMIT/MARKET_IF_TOUCHED/LIMIT_IF_TOUCHED |
| price | string | No | Reference price (fill in for non-market orders); precision up to 3 decimal places for securities accounts, 9 for futures accounts, excess is truncated |
| order_id | string | No | Original order ID when modifying an order; leave empty to query max quantity for a new order; when filled, returns the max modifiable quantity for that order |

**Response `d`:**

| Field | Type | Description |
|-------|------|-------------|
| max_cash_buy | string | Cash buy quantity |
| max_cash_and_margin_buy | string | Margin buy quantity |
| max_position_sell | string | Position sell quantity |
| max_sell_short | string | Short sell quantity |
| max_buy_back | string | Buy back/close short quantity |
| long_required_im | string | Initial margin change required for buying one contract |
| short_required_im | string | Initial margin change required for selling one contract |

**Special behavior:**
- When querying max modifiable quantity (with `order_id`), wait at least 0.5 seconds after order placement before calling
- When `order_id` is not provided, queries the max available quantity for a new order

**Error codes:**
| errcode | Trigger condition | Recommended action |
|---------|-------------------|-------------------|
| -1200 | General error | Handle based on errmsg content |
