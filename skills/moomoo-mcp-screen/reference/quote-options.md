# Derivatives Tool Reference

## quote_option_expiration_date — Option Expiry Date List

Retrieve the option expiry date list for an underlying. Supports HK/US/JP stocks and index options.

**Parameters:**
| Parameter | Type | Required | Description |
|------|------|------|------|
| symbol | string | Yes | Underlying code, e.g. `HK.00700`, `US.AAPL` |
| index_option_type | int | No | Index option type (required only for index underlyings): HK 1=HSI/2=HSCEI/3=Mini-HSI/4=Mini-HSCEI/5=HSTECH; US 1001=VIX/1003=XSP/1007=DJX/1020=RUT etc.; JP 2001=N225/2002=N225M/2003=TOPIX |
| filter_standard | string | No | ALL/STANDARD/NON_STANDARD, default ALL |
| filter_expiration_cycles | string | No | Filter by expiration cycle, comma-separated, e.g. `MONTH,QUARTERLY` |

**Returns `data.expiration_list[]`:**

| Field | Type | Description |
|------|------|------|
| strike_time | string | Expiry date (yyyy-MM-dd) |
| option_expiry_date_distance | int | Days to expiry (negative = expired) |
| expiration_cycle | string | Expiration cycle type |

---

## quote_option_chain — Option Chain

Retrieve the option contract list for an underlying within a specified expiry date range (each entry is a single CALL or PUT).

**Parameters:**
| Parameter | Type | Required | Description |
|------|------|------|------|
| symbol | string | Yes | Underlying code |
| start | string | No | Start expiry date yyyy-MM-dd |
| end | string | No | End expiry date yyyy-MM-dd |
| index_option_type | int | No | Same as above |
| filter_standard | string | No | Same as above |

**Returns `data.option_chain[]`:**

| Field | Type | Description |
|------|------|------|
| code | string | Option contract code, e.g. `HK.TCH260528C230000` |
| stock_id | uint64 | Option contract internal ID |
| name | string | Contract English name |
| sc_name | string | Contract Simplified Chinese name |
| tc_name | string | Contract Traditional Chinese name |
| lot_size | int | Contract size per lot, e.g. 100 |
| stock_type | string | Security type, fixed as `DRVT` |
| option_type | string | Option direction: CALL / PUT |
| stock_owner | string | Underlying code, e.g. `HK.00700` |
| strike_time | string | Expiry date (yyyy-MM-dd) |
| strike_price | number | Strike price (actual value) |
| index_option_type | string | Index option type: NORMAL / SMALL / N/A |
| expiration_cycle | string | Expiration cycle |
| option_standard_type | string | Standard type: STANDARD / NON_STANDARD / N/A |

Returns at most 20 expiry dates per call (nearest first).

---

## quote_option_volatility — Option Volatility Analysis

Retrieve IV/HV time-series analysis and statistical summary for an option contract.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|------|------|------|------|------|
| symbol | string | Yes | — | Option contract code, e.g. `HK.TCH260528C230000` |
| query_time_period | int | No | 2 | Query period: 1=1 week, 2=1 month, 3=3 months, 4=6 months, 5=1 year |
| hv_time_period | int | No | 30 | Historical volatility calculation days, 5~250 |

**Returns `data.item_list[]`:**

| Field | Type | Description |
|------|------|------|
| timestamp | int64 | Timestamp (milliseconds) |
| implied_volatility | float | Implied volatility (percentage, e.g. 28.391 = 28.391%) |
| history_volatility | float | Historical volatility (percentage) |
| volatility_premium | float | Volatility premium (IV - HV, percentage) |

**Returns `data.extra`:**

| Field | Type | Description |
|------|------|------|
| average_impvol | float | Average implied volatility for the period (percentage) |
| impvol_status | string | Volatility status: FLUCTUATING / OVERVALUED / UNDERVALUED |
| analysis | string | Volatility analysis text (multi-line separated by `\n`) |

---

## quote_option_exercise_probability — Option Exercise Probability

Retrieve the historical probability time-series of an option contract finishing in-the-money at expiry.

**Parameters:**
| Parameter | Type | Required | Description |
|------|------|------|------|
| symbol | string | Yes | Option contract code |
| limit | int | No | Maximum number of entries to return, 1~1000 |

**Returns `data.item_list[]`:**

| Field | Type | Description |
|------|------|------|
| timestamp | int64 | Timestamp (milliseconds) |
| security_price | float | Underlying asset price |
| strike_probability | float | Exercise probability (percentage, e.g. 97.915 = 97.915%) |

---

## quote_option_screen — Option Screener

Cross-market multi-dimensional option contract screening, supporting combinations of underlying, contract attributes, market indicators, and more, with sorting and pagination.

**Parameters:**
| Parameter | Type | Required | Description |
|------|------|------|------|
| strategy | object | Yes | Screening strategy (see detailed description below) |
| field_filter | object | No | Specify return fields (set int fields to 1, string fields like option_name/product_code to a string, nested objects like underlying_info use sub-objects). When omitted, only volume/price/chg_ratio/implied_volatility + option_id are returned |
| sort_obj | object | No | Sorting: `{sort_field: {<field_name>: 1}, is_asc: 0 or non-zero}`. Set exactly one field to 1 in sort_field. Defaults to volume descending when omitted |
| limit | int | No | Page size, default 100, max 1000; can be 0 (count only, with request_exact_data=0) |
| next_key | string | No | Pagination cursor, empty for first request, then pass pagination.next_key until has_more=false |
| request_exact_data | int | No | 1=return detail list (default), 0=return only has_more+total (option_list is empty) |

### strategy Structure

```json
{
  "market_category_list": [0],
  "filter_group_list": [
    {"underlying_list": [{"indicator_type": 101, "indicator_value": {"value_list": [205189]}}]},
    {"option_list": [{"indicator_type": 1003, "indicator_value": {"value_list": [1]}}]},
    {"option_list": [{"indicator_type": 1002, "indicator_value": {"value_interval": {"min_value": 7, "max_value": 45}}}]}
  ]
}
```

**market_category_list** — Market category array (union):
- 0=US_STOCK, 1=US_INDEX, 2=US_FUTURE, 3=HK_STOCK, 4=HK_INDEX, 5=JP_STOCK, 6=JP_INDEX

**filter_group_list** — Condition group array, groups are AND-ed. Each group contains one of: `underlying_list` / `option_list` / `chain_list` / `combo_list`. Conditions within a group are OR-ed; for AND across indicators, place each in a separate group.

Each condition structure:

| Field | Type | Description |
|------|------|------|
| indicator_type | int | Indicator type ID (see enumeration tables below) |
| indicator_value | object | Match value, fill one of `value_list` or `value_interval` |
| indicator_value.value_list | int[] | Discrete value matching (e.g. option type 1=CALL) |
| indicator_value.value_interval | object | Range matching |
| indicator_value.value_interval.min_value | int | Range lower bound (omit for no lower bound) |
| indicator_value.value_interval.max_value | int | Range upper bound (omit for no upper bound) |
| indicator_value.value_interval.exclude_min | bool | Whether to exclude lower bound (default false) |
| indicator_value.value_interval.exclude_max | bool | Whether to exclude upper bound (default false) |
| sub_indicator_list | array | Sub-indicator list (required only for composite indicators marked with SUB, same structure as this table) |

**Discrete value example (filter CALL):**
```json
{"indicator_type": 1003, "indicator_value": {"value_list": [1]}}
```

**Range example (7~45 days remaining):**
```json
{"indicator_type": 1002, "indicator_value": {"value_interval": {"min_value": 7, "max_value": 45}}}
```

### underlying_list indicator_type

| indicator_type | Description | Value Description |
|---|---|---|
| 101 | Underlying code (exact match) | value_list passes stock_id |
| 102 | Watchlist/screener ID | value_list passes strategy ID |
| 103 | Sector | Requires additional plate_list field |
| 201 | Total option volume | value_interval |
| 202 | Total open interest | value_interval |
| 203 | IV x1e3 | value_interval |
| 204 | HV x1e3 | value_interval |
| 205 | IV_Rank x1e3 | value_interval |
| 206 | IV percentile x1e3 | value_interval |
| 207 | IV change x1e3 | value_interval |
| 208 | IV change rate x1e3 | value_interval |
| 209 | IV/HV x1e3 | value_interval |
| 210 | IV-HV x1e3 | value_interval |
| 301 | Earnings timestamp (seconds) | value_interval |
| 302 | IV_CRUSH average x1e3 | SUB |
| 401 | Market cap x1e3 | value_interval |
| 402 | Underlying price x1e9 | value_interval |
| 403 | Change rate x1e3 | value_interval |

### option_list indicator_type

| indicator_type | Description | Value Description |
|---|---|---|
| 1001 | Strike price x1e9 | value_interval |
| 1002 | Days remaining | value_interval |
| 1003 | Option type | value_list: 1=CALL, 2=PUT |
| 1004 | Exercise style | value_list: 0=American, 1=European, 2=Bermudan |
| 1005 | Expiration type | value_list: 0=Monthly, 1=Weekly, 2=Month-end, 3=Quarterly |
| 1006 | Product code | value_list passes string |
| 1007 | Expiry date timestamp (seconds) | value_interval |
| 2001 | In-the-money (0/1) | value_list |
| 2002 | Price x1e9 | value_interval |
| 2003 | Mid price x1e9 | value_interval |
| 2004 | Best bid x1e9 | value_interval |
| 2005 | Best ask x1e9 | value_interval |
| 2006 | Bid-ask spread x1e9 | value_interval |
| 2007 | Bid volume | value_interval |
| 2008 | Ask volume | value_interval |
| 2009 | Bid-ask ratio x1e3 | value_interval |
| 2010 | Change rate x1e3 | value_interval |
| 2011 | Volume | value_interval |
| 2012 | Turnover x1e3 | value_interval |
| 2013 | Open interest | value_interval |
| 2014 | Open interest value x1e3 | value_interval |
| 2015 | Open interest change | SUB |
| 2016 | Volume change rate x1e3 | SUB |
| 2017 | Open interest change rate x1e3 | SUB |
| 2018 | Volume/open interest x1e3 | value_interval |
| 2019 | Average volume x1e3 | SUB |
| 2020 | Average open interest x1e3 | SUB |
| 2021 | Premium x1e9 | value_interval |
| 3001 | IV x1e3 | value_interval |
| 3002 | HV x1e3 | value_interval |
| 3003 | IV/HV x1e3 | value_interval |
| 3004 | Delta x1e5 | value_interval |
| 3005 | Gamma x1e5 | value_interval |
| 3006 | Vega x1e5 | value_interval |
| 3007 | Theta x1e5 | value_interval |
| 3008 | Rho x1e5 | value_interval |
| 3009 | Leverage x1e5 | value_interval |
| 3010 | Effective leverage x1e5 | value_interval |
| 3011 | Buy breakeven gain x1e3 | value_interval |
| 3012 | Sell breakeven drop x1e3 | value_interval |
| 3013 | Buy profit probability x1e3 | value_interval |
| 3014 | Sell profit probability x1e3 | value_interval |
| 3019 | Expiry exercise probability x1e3 | value_interval |
| 3021 | Sell annualized return x1e3 | value_interval |
| 3022 | Sell range return x1e3 | value_interval |
| 3023 | Buy breakeven price x1e9 | value_interval |
| 4001 | First expiry after earnings (0/1) | value_list |

### chain_list indicator_type

| indicator_type | Description |
|---|---|
| 20000 | Expected move x1e3 |
| 20001 | IVx x1e3 |
| 20002 | Call volume |
| 20003 | Put volume |
| 20004 | Total volume |

### field_filter Example

```json
{
  "hp_strike_price": 1,
  "option_name": "x",
  "product_code": "x",
  "option_type": 1,
  "exercise_type": 1,
  "expiration_type": 1,
  "in_the_money": 1,
  "left_day": 1,
  "price": 1,
  "mid_price": 1,
  "bid_price": 1,
  "ask_price": 1,
  "bid_ask_spread": 1,
  "change_ratio": 1,
  "volume": 1,
  "turnover": 1,
  "open_interest": 1,
  "implied_volatility": 1,
  "delta": 1,
  "gamma": 1,
  "vega": 1,
  "theta": 1,
  "rho": 1,
  "leverage_ratio": 1,
  "underlying_info": {"price": 1, "iv": 1, "hv": 1, "iv_rank": 1, "volume": 1, "open_interest": 1}
}
```

### sort_obj Example

```json
{"sort_field": {"volume": 1}, "is_asc": 0}
```
- `sort_field`: Set exactly one field to 1 (e.g. `{"volume":1}`, `{"implied_volatility":1}`, `{"open_interest":1}`, `{"delta":1}`)
- `is_asc`: Non-zero = ascending, 0/omitted = descending (default)

**Returns `data.option_list[]` + `pagination`:**

| Field | Type | Description |
|------|------|------|
| pagination.total | int | Total count (capped at 9999 when request_exact_data=0) |
| pagination.has_more | bool | Whether there are more results |
| pagination.next_key | string | Next page cursor (`"-1"` means no more) |

**option_list[] Contract Basic Fields:**

| Field | Type | Description |
|------|------|------|
| code | string | Contract code, e.g. `US.AAPL260115C00200000` |
| option_name | string | Contract name, e.g. `AAPL 260526 312.50C` |
| strike_price | double | Strike price (raw value / 1e9) |
| strike_date | string | Expiry date (yyyyMMdd) |
| strike_date_timestamp | int64 | Expiry date timestamp (milliseconds) |
| option_type | string | Direction: CALL / PUT |
| exercise_type | string | Exercise style: AMERICAN / EUROPEAN |
| expiration_type | string | Expiration type: WEEK / MONTH / QUARTER etc. |
| in_the_money | string | Moneyness: IN_THE_MONEY / OUT_OF_THE_MONEY |
| left_day | int | Days to expiry |
| multiplier | double | Contract multiplier (/ 1e9) |
| contract_share_size | double | Shares per contract (/ 1e9) |
| product_code | string | Option chain product code |

**option_list[] Market & Order Book Fields:**

| Field | Type | Description |
|------|------|------|
| price | double | Last price (/ 1e9) |
| mid_price | double | Bid-ask mid price (/ 1e9) |
| bid_price | double | Best bid (/ 1e9) |
| ask_price | double | Best ask (/ 1e9) |
| bid_ask_spread | double | Bid-ask spread (/ 1e9) |
| bid_volume | int | Best bid volume |
| ask_volume | int | Best ask volume |
| change_ratio | double | Change rate percentage (/ 1e5) |
| volume | int | Volume |
| turnover | double | Turnover (/ 1e3) |
| open_interest | int | Open interest |

**option_list[] Volatility & Greeks Fields:**

| Field | Type | Description |
|------|------|------|
| implied_volatility | double | Implied volatility percentage (/ 1e5) |
| history_volatility | double | Historical volatility percentage (/ 1e5) |
| delta | double | Delta (/ 1e5) |
| gamma | double | Gamma (/ 1e5) |
| vega | double | Vega (/ 1e5) |
| theta | double | Theta (/ 1e5) |
| rho | double | Rho (/ 1e5) |
| leverage_ratio | double | Leverage ratio (/ 1e5) |

**option_list[].underlying_info — Underlying Statistics Sub-object:**

| Field | Type | Description |
|------|------|------|
| stock_id | uint64 | Underlying internal ID |
| volume | int | Total option volume for the underlying |
| open_interest | int | Total option open interest for the underlying |
| iv | double | Underlying IV percentage (/ 1e5) |
| hv | double | Underlying 30-day HV percentage (/ 1e5) |
| iv_rank | double | IV Rank percentage (/ 1e5) |
| price | double | Underlying last price (/ 1e9) |
| change_ratio | double | Underlying change rate percentage (/ 1e5) |

**Common filter conditions (in filter_group_list):**
- option_list: Strike price (1001), days remaining (1002), option type (1003, 1=CALL/2=PUT), expiry date (1007), price (2002), volume (2011), open interest (2013), IV (3001), Delta (3004)
- underlying_list: Underlying code (101), total volume (201), IV (203)

---
