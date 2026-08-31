# Short Selling / Brokers / Stock Basic Info
## quote_short_interest — Short Interest

Get the stock's short interest (open short position) data. HK returns short position details; US returns monthly short interest reports. Data is sorted in reverse chronological order.

**Supported markets:** HK/US common stocks and ETFs only. Other markets/categories return unsupported.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| symbol | string | Yes | — | Stock symbol, e.g. `HK.00700`, `US.AAPL` |
| count | int | No | 30 | Number of records, range 1~90 |

**Response `data.items[]` (HK):**

| Field | Type | Description |
|-------|------|-------------|
| timestamp | int64 | Data timestamp (**milliseconds**) |
| timestamp_str | string | Data date (yyyy-MM-dd) |
| aggregated_short | string | Short open interest shares (int64 serialized as string) |
| aggregated_short_ratio | float | Percentage of float shares (%) |
| close_price | float | Close price for the day |
| last_close_price | float | Previous trading day close price |
| avg_cost | float | Average short position cost |
| avg_daily_short_volume | string | Average daily short volume |

**Response `data.items[]` (US):**

| Field | Type | Description |
|-------|------|-------------|
| timestamp | int64 | Data timestamp (**milliseconds**) |
| timestamp_str | string | Data date (yyyy-MM-dd) |
| shares_short | string | Short open interest shares (int64 serialized as string) |
| short_percent | float | Short percentage (%) |
| avg_daily_share_volume | string | Average daily volume |
| days_to_cover | float | Days to cover |
| close_price | float | Close price |
| last_close_price | float | Previous trading day close price |
| avg_daily_short_volume | string | Average daily short volume |

**HK vs US field differences:**
| Difference | HK | US |
|------------|----|----|
| Short shares | `aggregated_short` | `shares_short` |
| Ratio | `aggregated_short_ratio` | `short_percent` |
| Average cost | `avg_cost` (available) | N/A |
| Days to cover | N/A | `days_to_cover` (available) |
| Average daily volume | N/A | `avg_daily_share_volume` (available) |
| Common fields | `timestamp`, `timestamp_str`, `close_price`, `last_close_price`, `avg_daily_short_volume` | Same |

**Error codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|---------------------|
| -3 | count out of range or invalid symbol format | Fix parameters and retry |
| -7 | Symbol format is valid but security does not exist | Confirm symbol |
| -8 | Market or category not in supported range | Only call for HK/US common stocks and ETFs |
| -10 | Security is valid but no short interest data | Normal empty result |
| -5 | Gateway/backend internal error | Retry |

---

## quote_daily_short_volume — Daily Short Volume

Get daily short selling volume data (HK is turnover-based, US is position-based). Data is sorted in reverse chronological order.

**Supported markets:** HK/US shortable securities only (common stocks, ETFs, REITs, etc.). Options, futures, warrants, and indices are not supported.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| symbol | string | Yes | — | Stock symbol, e.g. `HK.00700`, `US.AAPL` |
| count | int | No | 30 | Number of records, range 1~90 |

**Response `data` top-level fields (HK):**

| Field | Type | Description |
|-------|------|-------------|
| market | string | Market identifier, fixed as `HK` |
| aggregated_short | int | Accumulated short position shares |
| aggregated_short_ratio | float | Accumulated short position as percentage of float shares (%) |
| new_time | string | Data update time (dd/MM/yyyy) |

**Response `data.items[]` (HK):**

| Field | Type | Description |
|-------|------|-------------|
| timestamp | int64 | Data timestamp (**milliseconds**) |
| timestamp_str | string | Data date (yyyy-MM-dd) |
| shares_traded | int | Total shares traded for the day |
| turnover | float | Total turnover for the day (HKD) |
| short_sell_shares_traded | int | Short selling shares traded for the day |
| short_sell_turnover | float | Short selling turnover for the day (HKD) |
| open_price | float | Open price |
| close_price | float | Close price |
| last_close_price | float | Previous trading day close price |
| daily_trade_avg_ratio | float | Average daily trading ratio (%) |

**Response `data.items[]` (US):**

| Field | Type | Description |
|-------|------|-------------|
| timestamp | int64 | Data timestamp (**milliseconds**) |
| timestamp_str | string | Data date (yyyy-MM-dd) |
| total_shares_short | int | Total short shares for the day |
| nasdaq_shares_short | int | NASDAQ short shares |
| nyse_shares_short | int | NYSE short shares |
| short_percent | float | Short volume percentage (%) |
| volume | int | Total volume for the day |
| close_price | float | Close price |
| last_close_price | float | Previous trading day close price |
| daily_trade_avg_ratio | float | Average daily trading ratio (%) |

**HK vs US field differences:**
| Difference | HK | US |
|------------|----|----|
| Data dimension | Turnover-based | Position-based |
| data top-level extra fields | `aggregated_short`, `aggregated_short_ratio`, `new_time` | None |
| Short volume fields | `short_sell_shares_traded`, `short_sell_turnover` | `total_shares_short`, `nasdaq_shares_short`, `nyse_shares_short`, `short_percent` |
| Turnover | `turnover` (HKD) | N/A (use `volume` instead) |
| Open price | `open_price` (available) | N/A |
| Common fields | `timestamp`, `timestamp_str`, `close_price`, `last_close_price`, `daily_trade_avg_ratio` | Same |

**Error codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|---------------------|
| -3 | count out of range or invalid symbol format | Fix parameters and retry |
| -7 | Symbol cannot be resolved | Confirm symbol |
| -8 | Market or category not supported | Only call for HK/US shortable securities |
| -10 | Security is valid but no short volume data | Normal empty result |

---

## quote_top_ten_brokers — Top Ten Brokers (Real-time)

Get the top ten net-buy/net-sell brokers for the current day (HK only), including broker ID, net volume, and average price. Real-time data is only returned during HK trading hours.

**Supported markets:** HK common stocks, ETFs, and REITs only.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| symbol | string | Yes | — | HK stock symbol, e.g. `HK.00700` |
| date | string | No | Latest trading day | Trading day, format YYYY-MM-DD |

**Response `data`:**

| Field | Type | Description |
|-------|------|-------------|
| is_real_time | bool | `true`=real-time data, `false`=historical data |
| data_time | int64 | Data update timestamp (**milliseconds**) |
| data_time_str | string | Data update time (readable string) |
| sec_volume | int | Total volume for the day (shares) |
| sec_turnover | float | Total turnover for the day |
| buy_brokers[] | array | Net-buy broker list (up to 10) |
| sell_brokers[] | array | Net-sell broker list (up to 10) |

**buy_brokers[] / sell_brokers[] elements (real-time):**

| Field | Type | Description |
|-------|------|-------------|
| broker_id | int | Broker ID |
| net_vol | int | Net volume (positive for buy, negative for sell) |
| avg_price | float | Average transaction price |

---

## quote_top_ten_brokers_history — Top Ten Brokers (Historical)

Get the top ten net-buy/net-sell broker holdings for a specified historical trading day.

**Supported markets:** HK common stocks, ETFs, and REITs only.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| symbol | string | Yes | HK stock symbol, e.g. `HK.00700` |
| days_before | int | Yes | Number of days before the current trading day, must be >0, range 1~365 |

**Response `data`:**

| Field | Type | Description |
|-------|------|-------------|
| is_real_time | bool | Fixed as `false` |
| data_time | int64 | Data timestamp (**milliseconds**) |
| data_time_str | string | Data time (readable string) |
| buy_brokers[] | array | Net-buy broker list (up to 10) |
| sell_brokers[] | array | Net-sell broker list (up to 10) |

**buy_brokers[] / sell_brokers[] elements (historical):**

| Field | Type | Description |
|-------|------|-------------|
| broker_name | string | Broker name |
| broker_code | string | Broker code |
| net_vol | int | Net volume (positive for buy, negative for sell) |
| hold_ratio | float | Holding ratio (%) |

**Real-time vs Historical field differences:**
| Field | Real-time | Historical |
|-------|-----------|------------|
| broker_id | Available | N/A |
| broker_name | N/A | Available |
| broker_code | N/A | Available |
| avg_price | Available | N/A |
| hold_ratio | N/A | Available |
| sec_volume / sec_turnover | Available | N/A |
| net_vol | Available (common) | Available (common) |

**Error codes (common to both interfaces):**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|---------------------|
| -3 | Invalid symbol format or days_before out of range | Fix parameters and retry |
| -7 | Symbol cannot be resolved | Confirm symbol via search API |
| -8 | Not HK market | Only pass HK symbols |
| -2/-4/-6 | Gateway internal error | Retry |

---

## quote_stock_basicinfo — Stock Basic Info

Batch get static reference information for stocks/ETFs/indices/warrants/options/futures/bonds/forex/funds, etc. Returns only static attributes and suspension flag; no quotes/K-lines/fundamentals.

**Supported market prefixes:** HK, US, SH, SZ, SG, JP, AU, CA, MY, FX, CC, BMD, SGX, HKEX, and other registered markets.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| code_list | string[] | Yes | Symbol list, 1~400 items, e.g. `["HK.00700", "US.AAPL"]` |

**Response `data.basic_list[]`:**

| Field | Type | Description |
|-------|------|-------------|
| code | string | Stock symbol, e.g. `HK.00700` |
| name | string | English name |
| sc_name | string | Simplified Chinese name |
| tc_name | string | Traditional Chinese name |
| stock_id | int64 | Internal numeric identifier |
| lot_size | int | Board lot size (options=contract shares; futures=contract multiplier) |
| stock_type | string | Security type: `STOCK` / `DRVT` (derivatives/options/warrants) / `FUTURE` / etc. |
| listing_date | int64 | Listing timestamp (**milliseconds**); 0 for futures/options/indices with no listing date |
| suspension | bool | Whether suspended (suspension flag only; see `state` for full lifecycle) |
| state | string | Security lifecycle state: `NORMAL` / etc. |
| contract_size | int\|null | Option contract shares (shares per contract for the underlying); null for non-options |
| main_contract | bool | Whether this is a main continuous contract (futures) |
| stock_child_type | string | Warrant sub-type: `N/A`=not applicable / etc. |
| stock_owner | string | Underlying stock symbol (for warrants/options), e.g. `HK.00700`; not returned for non-derivatives |

**Special behavior:**
- When some symbols cannot be resolved, valid symbols are returned normally while invalid ones are silently skipped (ret_code is still 0)
- When all symbols cannot be resolved, returns ret_code=-7

**Error codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|---------------------|
| 0 | Success (including some invalid symbols being skipped) | Compare request and response symbol lists to identify invalid items |
| -3 | code_list missing, empty, or exceeds 400 | Fix request body and retry |
| -7 | All symbols cannot be resolved | Confirm symbol validity via search API |
| -2 | Backend error | Retry |
