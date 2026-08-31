# Real-Time Market Data Tool Reference

## quote_stock_quote — Real-Time Quotes

Batch retrieve real-time stock quotes; subscription required.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| code_list | string[] | Yes | Stock code list, e.g. `["HK.00700","US.AAPL"]`, up to 400 symbols |

**Response `data.quote_list[]`:**

| Field | Type | Description |
|-------|------|-------------|
| code | string | Security code |
| name | string | English name |
| sc_name | string | Simplified Chinese name |
| tc_name | string | Traditional Chinese name |
| data_time | int64 | Exchange quote time (millisecond timestamp) |
| data_date | string | Quote trading date (market timezone YYYY-MM-DD) |
| last_price | double | Latest price |
| open_price | double | Today's open |
| high_price | double | Today's high |
| low_price | double | Today's low |
| prev_close_price | double | Previous close |
| volume | int64 | Volume (shares/contracts) |
| turnover | double | Turnover |
| turnover_rate | double | Turnover rate (percentage, 0.353 means 0.353%) |
| amplitude | double | Amplitude (percentage) |
| sec_status | string | Security status |
| suspension | bool | Whether suspended |
| dark_status | string | Dark pool status |
| listing_date | string | Listing date (YYYY-MM-DD) |

**option_ex_data sub-object (options only):**

| Field | Type | Description |
|-------|------|-------------|
| strike_price | double | Strike price |
| contract_size | int64 | Contract size |
| open_interest | int64 | Open interest |
| implied_volatility | double | Implied volatility (percentage) |
| premium | double | Option premium |
| delta | double | Delta |
| gamma | double | Gamma |
| vega | double | Vega |
| theta | double | Theta |
| rho | double | Rho |
| net_open_interest | int64 | Net open interest |
| contract_nominal_value | double | Contract nominal value |
| owner_lot_multiplier | int64 | Underlying lot multiplier |
| contract_multiplier | int64 | Contract multiplier |
| option_type | string | Option direction |
| index_option_type | int32 | Index option type |
| expiry_date_distance | int64 | Days to expiry (negative = expired) |
| option_area_type | string | Exercise type |

**future_ex_data sub-object (futures only):**

| Field | Type | Description |
|-------|------|-------------|
| last_settle_price | double | Previous settlement price |
| position | int64 | Open interest |
| position_change | int64 | Position change |

**pre_market / after_market / overnight sub-objects:**

| Field | Type | Description |
|-------|------|-------------|
| price | double | Session price |
| high_price | double | Session high |
| low_price | double | Session low |
| volume | int64 | Session volume |
| turnover | double | Session turnover |
| change_val | double | Price change |
| change_rate | double | Change rate (percentage) |
| amplitude | double | Amplitude (percentage) |

---

## quote_market_snapshot — Market Snapshot

The most comprehensive quote API, including 52-week high/low, market cap, PE, PB, dividend yield, warrant/trust/option/futures/pre-market/after-hours/overnight fields. Returns `data.snapshot_list[]`.

**Supported markets:** HK (stocks/trusts/REITs/warrants/CBBCs/inline warrants/indices/sectors/ETFs/options), US (stocks/ETFs/indices), SH/SZ (stocks/ETFs/indices/sectors), BJ (stocks/indices)

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| code_list | string[] | Yes | Stock code list, 1–400 items, e.g. `["HK.00700", "US.AAPL"]` |

**Response `data.snapshot_list[]` — Common fields:**

| Field | Type | Description |
|-------|------|-------------|
| code | string | Security code |
| name / sc_name / tc_name | string | English/Simplified Chinese/Traditional Chinese name |
| update_time | int64 | Quote update time (**millisecond** timestamp) |
| data_date | string | Quote trading date (market timezone YYYY-MM-DD) |
| last_price | double | Latest price |
| open_price | double | Today's open |
| high_price | double | Today's high |
| low_price | double | Today's low |
| prev_close_price | double | Previous close |
| close_price_5min | double | Most recent 5-minute close price |
| volume | int64 | Volume |
| turnover | double | Turnover |
| turnover_rate | double | Turnover rate (%) |
| amplitude | double | Amplitude (%) |
| volume_ratio | double | Volume ratio |
| bid_ask_ratio | double | Bid-ask ratio (%, positive = buy pressure, negative = sell pressure) |
| sec_status | string | Security status |
| dark_status | string | Dark pool status |
| listing_date | int64 | Listing date (**millisecond** timestamp); 0 if no listing date |
| bid_price / ask_price | double | Best bid / best ask price |
| bid_vol / ask_vol | int64 | Best bid / best ask volume |
| price_spread | double | Price spread |
| highest52weeks_price | double | 52-week high (unadjusted) |
| lowest52weeks_price | double | 52-week low (unadjusted) |
| highest_history_price | double | All-time high (unadjusted) |
| lowest_history_price | double | All-time low (unadjusted) |
| suspension | bool | Whether suspended |
| avg_price | double | Average price |
| lot_size | int64 | Shares per lot |

**Category flag fields (used to determine whether the category-specific fields below are meaningful):**

| Field | Type | Description |
|-------|------|-------------|
| equity_valid | bool | Whether it is a stock |
| index_valid | bool | Whether it is an index |
| plate_valid | bool | Whether it is a sector |
| wrt_valid | bool | Whether it is a warrant/CBBC/inline warrant |
| trust_valid | bool | Whether it is a trust/fund/REIT |
| option_valid | bool | Whether it is an option |
| future_valid | bool | Whether it is a futures contract |

**Stock fields (equity_valid=true):**

| Field | Type | Description |
|-------|------|-------------|
| issued_shares | int64 | Total shares outstanding |
| total_market_val | double | Total market cap |
| outstanding_shares | int64 | Free float shares |
| circular_market_val | double | Free float market cap |
| pe_ratio | double | Static PE |
| pe_ttm_ratio | double | PE_TTM |
| pb_ratio | double | PB |
| dividend_ttm | double | TTM dividend per share |
| dividend_ratio_ttm | double | TTM dividend yield (%) |
| dividend_lfy | double | Last fiscal year dividend per share |
| dividend_lfy_ratio | double | Last fiscal year dividend yield (%) |
| net_asset | double | Net assets |
| net_asset_per_share | double | Net asset per share |
| net_profit | double | Net profit |
| ey_ratio | double | Earnings yield EY (%) |
| earning_per_share | double | EPS |

**Index fields (index_valid=true):**

| Field | Type | Description |
|-------|------|-------------|
| index_raise_count | int64 | Number of advancing constituents |
| index_fall_count | int64 | Number of declining constituents |
| index_equal_count | int64 | Number of unchanged constituents |

**Warrant/CBBC fields (wrt_valid=true):**

| Field | Type | Description |
|-------|------|-------------|
| wrt_maturity_date | int64 | Maturity date (**second**-level timestamp) |
| wrt_end_trade | int64 | Last trading day (**second**-level timestamp) |

**Trust/Fund/REIT fields (trust_valid=true):**

| Field | Type | Description |
|-------|------|-------------|
| trust_aum | double | AUM |
| trust_dividend_yield | double | Dividend yield (%) |
| trust_outstanding_units | int64 | Outstanding units |
| trust_netAssetValue | double | Net asset value (NAV) per unit |
| trust_premium | double | Premium (%) |
| trust_assetClass | string | Asset class: `STOCK`/`BOND`/`COMMODITY`/`CURRENCY_MARKET`/`FUTURE`/`SWAP` |

**Option fields (option_valid=true):**

| Field | Type | Description |
|-------|------|-------------|
| option_strike_price | double | Strike price |
| option_contract_size | int64 | Contract size |
| option_open_interest | int64 | Open interest |
| option_implied_volatility | double | Implied volatility (%) |
| delta | double | Delta |
| gamma | double | Gamma |
| vega | double | Vega |
| theta | double | Theta |
| rho | double | Rho |
| option_net_open_interest | int64 | Net open interest |
| option_contract_nominal_value | double | Contract nominal value |
| option_owner_lot_multiplier | double | Underlying lot multiplier |
| option_type | string | Option direction: `CALL`/`PUT` |
| option_contract_multiplier | int64 | Contract multiplier |
| index_option_type | int32 | Index option type |
| option_expiry_date_distance | int64 | Days to expiry (negative = expired) |
| option_area_type | string | Exercise type: `AMERICAN`/`EUROPEAN`/`BERMUDA` |

**Futures fields (future_valid=true):**

| Field | Type | Description |
|-------|------|-------------|
| future_last_settle_price | double | Previous settlement price |
| future_position | int64 | Open interest |
| future_position_change | int64 | Position change |

**Pre-market fields (0 outside pre-market session):**

| Field | Type | Description |
|-------|------|-------------|
| pre_price | double | Pre-market price |
| pre_high_price | double | Pre-market high |
| pre_low_price | double | Pre-market low |
| pre_volume | int64 | Pre-market volume |
| pre_turnover | double | Pre-market turnover |
| pre_change_val | double | Pre-market price change |
| pre_change_rate | double | Pre-market change rate (%) |
| pre_amplitude | double | Pre-market amplitude (%) |

**After-hours fields (0 outside after-hours session):**

| Field | Type | Description |
|-------|------|-------------|
| after_price | double | After-hours price |
| after_high_price | double | After-hours high |
| after_low_price | double | After-hours low |
| after_volume | int64 | After-hours volume |
| after_turnover | double | After-hours turnover |
| after_change_val | double | After-hours price change |
| after_change_rate | double | After-hours change rate (%) |
| after_amplitude | double | After-hours amplitude (%) |

**Overnight fields (0 outside overnight session):**

| Field | Type | Description |
|-------|------|-------------|
| overnight_price | double | Overnight price |
| overnight_high_price | double | Overnight high |
| overnight_low_price | double | Overnight low |
| overnight_volume | int64 | Overnight volume |
| overnight_turnover | double | Overnight turnover |
| overnight_change_val | double | Overnight price change |
| overnight_change_rate | double | Overnight change rate (%) |
| overnight_amplitude | double | Overnight amplitude (%) |

**Special behavior:** When some codes cannot be resolved, valid codes return normally and invalid codes are silently skipped (ret_code remains 0).

**Error codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|-------------------|
| 0 | Success (including silently skipped invalid codes) | Compare requested vs. returned code lists to identify invalid items |
| -3 | code_list missing/empty/exceeds 400 | Fix the request body and retry |
| -7 | All codes are unresolvable | Verify market prefix and code validity |
| -4/-6 | Gateway internal error | Retry |

---

## quote_cur_kline — Latest K-Lines

Retrieve the latest N K-line bars for a specified symbol, supporting multiple periods and adjustment types. Returns the most recent N bars relative to the current time; cannot specify a historical time window.

**Supported markets:** HK (stocks/trusts/REITs/warrants/CBBCs/inline warrants/indices/sectors/ETFs/options), US (stocks/ETFs/indices), SH/SZ (stocks/ETFs/indices/sectors), BJ (stocks/indices)

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| symbol | string | Yes | — | Stock code, e.g. `HK.00700` |
| num | int | Yes | — | Number of K-line bars, range 1–370 |
| ktype | int | No | 2 | K-line period (see enum below) |
| autype | int | No | 1 | Adjustment type (see enum below) |
| extended_time | int | No | 0 | Pre/post-market and overnight toggle (see enum below) |

## quote_history_kline — Historical K-Lines

Retrieve historical K-line data for a specified time range, with pagination toward earlier data. Preferred stocks/SPACs/convertible bonds/CBBCs return empty lists.

**Supported market prefixes:** HK, US, SH, SZ, BJ, SG, CA, AU, FX, JP, CC, etc. Supported types: stocks, ETFs, indices, futures, options, cryptocurrencies.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| symbol | string | Yes | — | Stock code, e.g. `HK.00700`, `US.FUTU` |
| end | string | Yes | — | End date yyyy-MM-dd (inclusive) |
| start | string | No | — | Start date yyyy-MM-dd (inclusive); if omitted, walks backward from end by num bars |
| num | int | No | 370 | Number of K-line bars, range 1–370 |
| ktype | int | No | 2 | K-line period (same enum as cur_kline) |
| autype | int | No | 1 | Adjustment type: 0=unadjusted / 1=forward-adjusted / 2=backward-adjusted |
| extended_time | int | No | 0 | Pre/post-market and overnight toggle (same enum as cur_kline) |

### ktype Enum (K-Line Period)

| Value | Meaning | Value | Meaning |
|-------|---------|-------|---------|
| 1 | 1-minute | 10 | 3-minute |
| 2 | Day (default) | 11 | Quarter |
| 3 | Week | 14 | 120-minute |
| 4 | Month | 15 | 240-minute |
| 5 | Year | 26 | 10-minute |
| 6 | 5-minute | 29 | 180-minute |
| 7 | 15-minute | | |
| 8 | 30-minute | | |
| 9 | 60-minute | | |

### autype Enum (Adjustment Type)

| Value | Meaning |
|-------|---------|
| 0 | Unadjusted |
| 1 | Forward-adjusted (ex-dividend, default) |
| 2 | Backward-adjusted (ex-dividend) |
| 3 | Forward-adjusted (cum-dividend) |
| 4 | Backward-adjusted (cum-dividend) |

### extended_time Enum

| Value | Meaning |
|-------|---------|
| 0 | Exclude pre/post-market (default) |
| 1 | Include pre/post-market (US 1-minute K-lines only) |
| 2 | Include overnight (US) |

### Response — cur_kline: `data.kline_list[]`

| Field | Type | Description |
|-------|------|-------------|
| code | string | Security code |
| name | string | English name |
| sc_name | string | Simplified Chinese name |
| tc_name | string | Traditional Chinese name |
| time_key | int64 | K-line timestamp (**milliseconds**) |
| date | int | K-line date (YYYYMMDD integer; for minute K-lines this is the trading date, for daily and above it is the local date) |
| open_price | float | Open price |
| close_price | float | Close price (latest bar = current latest price) |
| high_price | float | High price |
| low_price | float | Low price |
| last_close_price | float | Previous close price |
| volume | int | Volume (shares) |
| turnover | float | Turnover |
| turnover_rate | float | Turnover rate (%); may be omitted when 0 |
| pe | float | PE ratio |
| change_rate | float | Change rate (%) = (close - last_close) / last_close x 100 |

### Response — history_kline: `data`

**Top-level additional fields:**

| Field | Type | Description |
|-------|------|-------------|
| data.next_time | int64 | Next page start time (millisecond timestamp); convert to date and pass as `end` for pagination |
| data.volume_precision | int | Volume precision n; volume is already scaled by 10^n. Stocks/ETFs/futures/options usually 0; cryptocurrencies/event contracts may be >0 |

**`data.kline_list[]` elements:**

| Field | Type | Description |
|-------|------|-------------|
| time_key | int64 | K-line timestamp (**milliseconds**) |
| date | int | K-line date (YYYYMMDD) |
| time_zone | int | Timezone offset (minutes), e.g. 480=HK, -300=US Eastern daylight |
| open | float | Open price |
| close | float | Close price |
| high | float | High price |
| low | float | Low price |
| last_close | float | Previous close price |
| volume | int | Volume (shares) |
| turnover | float | Turnover |
| turnover_rate | float | Turnover rate (%) |
| pe_ratio | float | PE ratio |
| change_rate | float | Change rate (%) |
| name | string | English name |
| sc_name | string | Simplified Chinese name |
| tc_name | string | Traditional Chinese name |
| open_interest | int | Open interest (futures/options only; 0 or absent for other types) |
| settle_price | float | Settlement price (futures/options daily and above only; stocks/ETFs/indices backfill with close — should be ignored) |
| implied_volatility | float | Implied volatility (%) (options only; 0 or absent for other types) |

**Field name differences between history_kline and cur_kline:**
| cur_kline | history_kline |
|-----------|---------------|
| open_price | open |
| close_price | close |
| high_price | high |
| low_price | low |
| last_close_price | last_close |
| pe | pe_ratio |

### Error Codes

**cur_kline:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|-------------------|
| 0 | Success | — |
| -3 | Required parameter missing, num>370, invalid ktype/autype enum value | Fix parameters and retry |
| -4 | Invalid code or parameter assembly failure | Verify security code exists |
| -5 | Backend call failure (network/timeout) | Retry |
| >0 | Backend business error passthrough (no permission/risk control/rate limit, etc.) | Check ret_msg for details |

**history_kline:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|-------------------|
| 0 | Success (empty kline_list means valid but no data) | — |
| -3 | Missing end, type error, ktype out of range, date format mismatch | Fix parameters and retry |
| -7 | Symbol format is valid but security does not exist | Verify code via search API |
| -8 | Market prefix not in supported range | Verify market prefix is valid |

---
