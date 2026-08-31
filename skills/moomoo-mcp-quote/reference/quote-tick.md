# Order Book / Tick-by-Tick / Time-Sharing / Market State

## quote_order_book — Order Book / Bid-Ask Depth

Retrieve the real-time order book (depth) for a stock; subscription required. The number of depth levels depends on the user's market data permission level (LV1/LV2/LV3) and the security type.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| code | string | Yes | Stock code, format `{market}.{code}`, e.g. `HK.00700` |
| num | int | No | Level limit 1–60; omit to get the maximum depth allowed by permission |

**Response `data[]`:**

Note: `data` is always an array (even when querying a single code).

| Field | Type | Description |
|-------|------|-------------|
| code | string | Security code |
| name | string | English name |
| sc_name | string | Simplified Chinese name |
| tc_name | string | Traditional Chinese name |
| books[] | array | Order book array (US LV3 has multiple exchanges; other types have a single element) |

**books[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| exchange | string | Exchange identifier (US LV3: `NASDAQ`/`ARCA`; otherwise empty string) |
| bid_flag | int | Bid validity flag: 1=valid / 0=invalid |
| ask_flag | int | Ask validity flag: 1=valid / 0=invalid |
| exchange_data_time_ms | int64 | Exchange data generation time (millisecond timestamp) |
| server_send_to_client_time_ms | int64 | Server-to-client send time (millisecond timestamp) |
| order_volume_precision | int | Volume precision n (volume is already scaled by 10^n) |
| difference | float | Bid-ask spread (returned for FX only) |
| bid_list[] | array | Bid list (sorted by price descending) |
| ask_list[] | array | Ask list (sorted by price ascending) |

**bid_list[] / ask_list[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| price | double | Price at this level |
| volume | int64 | Volume at this level (combine with order_volume_precision to restore) |
| order_count | int | Number of orders at this level (0 for some markets/permission levels) |

**Depth level rules (determined by market data permission):**

| Market/Type | LV1 | LV2 | LV3 |
|-------------|-----|-----|-----|
| HK stocks/warrants/CBBCs/inline warrants | 1 level | 10 levels | — |
| HK options/futures | 1 level | 10 levels | — |
| US (including ETFs) | 1 level | 60 levels (consolidated) | 60 levels per exchange (NASDAQ/ARCA) |
| US options | 1 level | — | — |
| US futures | — | 40 levels | — |
| A-shares (SH/SZ) | 5 levels | — | — |
| Singapore | — | 40 levels | — |
| Malaysia | 3 levels | 5 levels | 10 levels |
| Japan | — | 10 levels | 40 levels |
| Other markets | 1 level | — | — |

**Error codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|-------------------|
| 0 | Success | — |
| -3 | code missing or num out of range | Fix parameters and retry |
| -4 | Code cannot be resolved or parameter assembly failure | Verify security code is valid |
| -6 | Gateway internal error | Retry |
| -7 | Code format is valid but security does not exist | Verify code via search API |

---

## quote_rt_ticker — Tick-by-Tick Trades

Retrieve tick-by-tick trade data for a stock (latest N ticks); subscription required. Only returns the latest N ticks; time range filtering is not supported.

**Supported markets:** HK (stocks/trusts/REITs/warrants/CBBCs/inline warrants/indices/sectors/ETFs/options), US (stocks/ETFs/indices), SH/SZ (stocks/ETFs/indices/sectors), BJ (stocks/indices)

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| symbol | string | Yes | — | Stock code, e.g. `HK.00700` |
| num | int | No | 500 | Number of ticks, range 1–750 |
| period | string | No | All sessions | Session filter, can be repeated (e.g. `?period=BEFORE&period=AFTER`): NORMAL/BEFORE/AFTER/OVERNIGHT |

**Response `data`:**

| Field | Type | Description |
|-------|------|-------------|
| code | string | Security code |
| name | string | English name |
| sc_name | string | Simplified Chinese name |
| tc_name | string | Traditional Chinese name |
| last_close | float | Previous close price (used for calculating change) |
| volume_precision | int | Volume precision n (volume is already scaled by 10^n; stocks usually 0, event/perpetual contracts may be >0) |
| ticker_list[] | array | Tick-by-tick trade list |

**ticker_list[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| sequence | int64 | Tick sequence number (monotonically increasing; can be used for deduplication/incremental fetching) |
| time | int64 | Trade time (millisecond timestamp) |
| price | float | Trade price |
| volume | int | Volume (combine with volume_precision to restore) |
| turnover | float | Turnover |
| ticker_direction | string | Buy/sell direction (see enum below) |
| tick_type | string | Trade type / matching method (see enum below) |
| period_type | string | Trading session |
| trade_type | string | Exchange trade type (ASCII character, for display; US examples: `P`=pre-market, `T`=Form-T, `U`=cancel; HK/A-shares may be empty) |

**ticker_direction enum:**
| Value | Meaning |
|-------|---------|
| BUY | Buy (active buy) |
| SELL | Sell (active sell) |
| NEUTRAL | Neutral (direction undetermined) |

**tick_type enum:**
| Value | Meaning |
|-------|---------|
| UNKNOWN | Unknown |
| AUTO_MATCH | Auto-match |
| LATE | Late trade |
| NON_AUTO_MATCH | Non-auto-match |
| ODD_LOT | Odd lot trade |
| AUCTION | Auction trade |
| BULK | Block trade |
| OVERSEAS | Overseas trade |
| UNAUTO_MATCH_OFF | Non-auto-match (off-exchange) |
| NON_DIRECT_OFF | Non-direct (off-exchange) |
| OVERSEAS_OFF | Overseas (off-exchange) |
| AUTO_MATCH_OFF | Auto-match (off-exchange) |
| BULK_OFF | Block trade (off-exchange) |
| LATE_OFF | Late trade (off-exchange) |
| AUCTION_OFF | Auction trade (off-exchange) |
| ODD_LOT_OFF | Odd lot trade (off-exchange) |
| EVENING | Evening trade |
| ACCEPT_ELECTRONIC | Electronic board acceptance |
| OUT_HOUR_CONTRACT | After-hours contract trade |
| BANK_CHARGE | CCASS charge |
| ELECTRONIC | Electronic trade |
| HIGH_DENSITY | High density trade |
| INTERMEDIATE_PRICE | Mid-price trade |
| AT_AUCTION | At-auction trade |
| AUCTION_LIMIT | Auction limit order |
| AT_AUCTION_LIMIT | At-auction limit trade |
| ENHANCE_LIMIT | Enhanced limit order |
| HOT_QUOTE | Real-time quote |
| MARKET | Market order |
| ROUND_LOT | Round lot |
| SPECIAL_LOT | Special lot |
| ODD_AND_SPECIAL_LOT | Odd lot and special lot |

**Error codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|-------------------|
| 0 | Success | — |
| -3 | Invalid symbol format, num out of range, invalid period value | Fix parameters and retry |
| -5 | Gateway/backend call failure | Retry |
| -7 | Code format is valid but security does not exist | Verify code via search API |
| >0 | Backend business error (no subscription/insufficient permission, etc.) | Check ret_msg for details |

---

## quote_rt_data — Time-Sharing Data

Retrieve intraday time-sharing (minute-level) data for the current trading day; subscription required. Only returns current day data; no cross-day history.

**Supported markets:** HK (stocks/trusts/REITs/warrants/CBBCs/inline warrants/indices/sectors/ETFs/options), US (stocks/ETFs/indices), SH/SZ (stocks/ETFs/indices/sectors), BJ (stocks/indices)

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| symbol | string | Yes | — | Stock code, e.g. `HK.00700` |
| request_section | string | No | NORMAL | Trading session filter (see enum below) |

**request_section enum:**
| Value | Meaning |
|-------|---------|
| NORMAL | Regular session (default; HK automatically includes dark pool) |
| FULL | Include pre/post-market (US only, excludes overnight) |
| PREMARKET | US pre-market |
| AFTERHOURS | US after-hours |
| HK_DARK | HK dark pool |
| OVERNIGHT | US overnight |

**Response `data`:**

| Field | Type | Description |
|-------|------|-------------|
| volume_precision | int | Volume precision n (volume is already scaled by 10^n; stocks usually 0, event contracts may be >0) |
| section_list[] | array | Trading session array |

**section_list[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| code | string | Security code |
| name | string | English name |
| sc_name | string | Simplified Chinese name |
| tc_name | string | Traditional Chinese name |
| trade_section | string | Session type (see enum below) |
| last_close | float | Reference/previous close price for this session |
| point_list[] | array | Minute data points |

**trade_section enum (response):**
| Value | Meaning |
|-------|---------|
| AUCTION | HK auction session |
| MORNING | HK morning session |
| AFTERNOON | HK afternoon session |
| NIGHT | Overnight session |
| US_PREMARKET | US pre-market |
| US_REGULAR | US regular trading session |
| US_AFTERHOURS | US after-hours |
| US_OVERNIGHT | US overnight |
| HK_DARK | HK dark pool |
| FUT_PART1 | Futures session 1 |
| FUT_PART2 | Futures session 2 |
| STIB_AFTERHOURS | STAR Market after-hours |
| DEFAULT | Default (general market) |
| REGULAR | Regular trading session |
| US_INDEX_OPT_REGULAR | US index option regular session |
| US_INDEX_OPT_GLOBAL | US index option global session |
| US_INDEX_OPT_CURB | US index option CURB session |
| JP_INDEX_OPT_NIGHT | Japan index option night session |
| JP_INDEX_OPT_REGULAR | Japan index option regular session |

**point_list[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| time | int64 | Time (millisecond timestamp) |
| open | float | Open price |
| high | float | High price |
| low | float | Low price |
| cur_price | float | Current price (close price for this minute) |
| volume | int | Volume (divide by 10^volume_precision to restore) |
| turnover | float | Turnover |

**Special behavior:**
- Non-trading days, unsubscribed securities, or unsupported types return empty `section_list`; ret_code remains 0
- When `request_section=NORMAL`, HK automatically includes dark pool data
- `request_section=FULL` is only effective for US, including pre/post-market but excluding overnight

**Error codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|-------------------|
| 0 | Success (including empty section_list) | — |
| -3 | symbol missing or invalid request_section | Fix parameters and retry |
| -4 | Code cannot be resolved or parameter assembly failure | Verify market prefix is valid |
| -5 | Backend call failure (network/timeout) | Retry |
| -6 | Gateway response conversion failure | Retry |
| >0 | Backend business error (no permission/risk control/rate limit, etc.) | Check ret_msg for details |

---

## quote_market_state — Market State

Batch retrieve the current trading state (open/closed/pre-market/after-hours/overnight, etc.) of the market a stock belongs to. This is a market-level state and does not reflect individual stock suspensions.

**Supported market prefixes:** HK, US, SH, SZ, BJ, SG, JP, CA, AU, CC (cryptocurrency). Unsupported market prefixes return `market_state="NONE"` without affecting other valid codes in the same batch.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| code_list | string[] | Yes | — | Stock code list, format `{market}.{code}`, up to 400 |
| is_contain_ba | bool | No | false | Whether to include US pre/post-market; when true, trade_section switches to the next trading day's session after pre/post-market opens |
| is_contain_overnight | bool | No | false | Whether to include US overnight; when true, trade_section switches to the next trading day's session after overnight opens |
| is_need_crypto_multi_broker | bool | No | false | Whether to return multi-broker data for cryptocurrency; when true, the same market_id expands into multiple records (one per broker) |

**Response `data.market_state_list[]`:**

| Field | Type | Description |
|-------|------|-------------|
| code | string | Security code (corresponds to request) |
| stock_name | string | English name |
| sc_name | string | Simplified Chinese name |
| tc_name | string | Traditional Chinese name |
| market_state | string | Market state enum (see below) |
| time_date | string | Trading date and time (Beijing timezone `YYYY-MM-DD HH:MM:SS`); omitted when backend does not provide it |
| traded_seconds | int | Seconds traded in current session; omitted when backend does not provide it |
| total_seconds | int | Total seconds in current trading session; omitted when backend does not provide it |
| trade_section[] | array | Intraday trading session slices (see below); omitted when backend does not provide them |
| broker_id | int | Broker ID (cryptocurrency only + `is_need_crypto_multi_broker=true`) |
| broker_ids | int[] | All broker IDs under this market_id (cryptocurrency composite quotes only) |

**trade_section[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| trade_section_type | int | Session type enum |
| begin_time | string | Session start time (Beijing timezone `HH:MM:SS`) |
| end_time | string | Session end time (Beijing timezone `HH:MM:SS`) |

**market_state enum (common values):**
| Value | Meaning |
|-------|---------|
| NONE | No state / unsupported market |
| MORNING | Morning session |
| AFTERNOON | Afternoon session |
| REST | Midday break |
| PRE_MARKET_BEGIN | Pre-market open |
| PRE_MARKET_END | Pre-market close |
| AFTER_HOURS_BEGIN | After-hours open |
| AFTER_HOURS_END | After-hours close |
| OVERNIGHT_BEGIN | Overnight open |
| OVERNIGHT_END | Overnight close |
| CLOSING_AUCTION | Closing auction |
| MARKET_CLOSE | Market closed |

**Special behavior:**
- Unsupported market prefixes (e.g. MY/DE/FR, etc.) return `market_state="NONE"` without affecting other valid codes in the same batch
- Codes without a market prefix (e.g. `"00700"`) trigger a parameter error
- `is_contain_ba` and `is_contain_overnight` only affect US market entries
- Individual stock suspensions are not reflected in this API; check the `suspension` field in `quote_market_snapshot`

**Error codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|-------------------|
| 0 | Success | — |
| -3 | code_list missing/empty/exceeds 400/elements missing market prefix | Fix request body and retry |
| -5 | Backend business error | Check ret_msg |
| -2/-4/-6 | Gateway internal error | Retry |

---

## quote_trading_days — Trading Calendar

Retrieve the trading calendar for a specified market within the [start, end] range. Non-trading days (full-day closure/weekends/holidays) are automatically filtered out. Not differentiated by security type; returns the market-level calendar.

**Supported markets:** HK / US / SH / SZ / BJ / SG / JP / CA / AU / JP_FUTURE / SG_FUTURE

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| market | string | Yes | Market prefix: HK/US/SH/SZ/BJ/SG/JP/CA/AU/JP_FUTURE/SG_FUTURE |
| start | string | Yes | Start date (inclusive), format `yyyy-MM-dd` |
| end | string | Yes | End date (inclusive), format `yyyy-MM-dd`; must be >= start |

**Response `data.trading_days[]`:**

| Field | Type | Description |
|-------|------|-------------|
| time | string | Trading date (`yyyy-MM-dd`) |
| trade_date_type | string | Trading day type (see enum below) |
| trade_second | int | Total trading seconds for the day; can be used to identify half-day trading (e.g. HK Christmas Eve ~ 9000s, normal full day = 19800s) |

**trade_date_type enum:**
| Value | Meaning |
|-------|---------|
| WHOLE | Full-day trading |
| MORNING | Morning session only (half-day) |

**Special behavior:**
- Both start and end are inclusive
- Non-trading days are automatically filtered and do not appear in results
- Not differentiated by security type; all instruments in the same market share the calendar

**Error codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|-------------------|
| 0 | Success | — |
| -3 | market/start/end missing; invalid market; date format error; start > end | Fix parameters and retry |
| -4/-6 | Gateway internal error | Retry |
