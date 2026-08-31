# Screening & Sector Tool Reference

## quote_stock_screen — Conditional Stock Screening

Multi-factor combination screening across the whole market, supporting valuation / price change / financials / technical patterns / broker holdings / options dimensions, returning matching results with specified factor values via pagination.

**Supported Markets:** HK/US/CN/SG/CA/AU/JA/MY (KR not supported)

**Parameters:**
| Parameter | Type | Required | Default | Description |
|------|------|------|------|------|
| screen_queries | object[] | Yes | — | Filter condition array, multiple conditions are AND-ed (each element fills exactly one of 11 query types) |
| retrieve_queries | object[] | No | — | Return column definitions (each element fills exactly one of 9 retrieval types), response results[] aligns with this array |
| sort | object | No | — | Single-field sort; mutually exclusive with sorts, sorts takes priority |
| sorts | object[] | No | — | Multi-field sort, array order defines priority |
| limit | int | No | 200 | Page size, max 300 |
| next_key | string | No | — | Pagination cursor; leave empty for first request, then pass back `pagination.next_key` |
| user_stock_list_mode | int | No | 0 | 0=unrestricted, 1=watchlist only, 2=holdings only |
| watchlist_stock_ids | int[] | No | — | Watchlist stock_id list; use with `user_stock_list_mode=1` |
| holding_stock_ids | int[] | No | — | Holding stock_id list; use with `user_stock_list_mode=2` |

**Returns `data`:**

| Field | Type | Description |
|------|------|------|
| items[] | array | Screening result list |
| pagination.total | int | Total match count |
| pagination.has_more | bool | Whether there are more results |
| pagination.next_key | string | Next page cursor |

**items[] Element:**

| Field | Type | Description |
|------|------|------|
| code | string | Stock code (Market.Code, e.g. `HK.00700`) |
| name | string | English name |
| sc_name | string | Simplified Chinese name |
| tc_name | string | Traditional Chinese name |
| results[] | array | Return value list (aligned with retrieve_queries) |

**results[] Element:**

Each element wraps the result for the corresponding retrieve_queries type:

| Field | Type | Description |
|------|------|------|
| res.ival | string(int64) | Raw integer value (pre-multiplied by precision factor) |
| res.dval | float | Floating-point value |
| res.sval | string | String value (e.g. name) |
| result_type | int | 1=double, 2=int, 3=string |
| value | string | Raw value in string form |

---

### screen_queries Condition Types (11 types, pick one per element)

**1. simple_field_query — Discrete Value Filter (IN)**
```json
{"simple_field_query": {"simple_field": 1, "screen_value_list": [2]}}
```

| simple_field | Meaning | screen_value_list Values |
|---|---|---|
| 1 | Market | 1=HK, 2=US, 3=CN, 4=SG, 5=CA, 6=AU, 7=JA, 8=MY |
| 2 | Exchange | Exchange ID |
| 3 | Index constituent | Index ID |
| 4 | Use watchlist stocks | — |
| 5 | Has ADR | 0/1 |
| 6 | Has options | 0/1 |
| 7 | Has warrants | 0/1 |
| 8 | Has futures | 0/1 |
| 9 | Has AH stock | 0/1 |
| 10 | Islamic compliant | 0/1 |
| 11 | Northbound holding ID | — |
| 12 | Market maker exclusive ID | — |

**2. plate_query — Sector Filter**
```json
{"plate_query": {"plate_list": [{"parent_plate_id": 0, "plate_id_list": [12345]}]}}
```

**3. simple_property_query — Market / Valuation Range**
```json
{"simple_property_query": {"property": {"name": 2303}, "upper": {"value": 1500000, "includes": true}}}
```

| name | Meaning | Precision Factor |
|------|------|----------|
| 2201 | Last price | x1e3 |
| 2301 | Total market cap | x1e3 |
| 2302 | Static PE | x1e5 |
| 2303 | PE_TTM | x1e5 |
| 2304 | PB | x1e5 |
| 2305 | Dividend yield | x1e3 |
| 2306 | Listing timestamp (seconds) | — |

**4. cumulative_property_query — Cumulative Price Change / Turnover**
```json
{"cumulative_property_query": {"property": {"name": 3102, "days": 5}, "lower": {"value": 5000, "includes": true}}}
```

| name | Meaning | Precision Factor |
|------|------|----------|
| 3101 | Price change amount | x1e3 |
| 3102 | Price change rate | x1e3 |
| 3103 | Amplitude | x1e3 |
| 3104 | Average volume | — |
| 3105 | Average turnover | x1e3 |
| 3106 | Turnover rate | x1e3 |

- `days`: Number of days (default 1, e.g. 5 = 5-day change rate)
- `period_average`: bool, whether to take daily average

**5. financial_property_query — Financial Indicators**
```json
{"financial_property_query": {"property": {"name": 4110, "term": 100}, "lower": {"value": 15000, "includes": true}}}
```

| name | Meaning | Precision Factor |
|------|------|----------|
| 4101 | Net profit | x1e3 |
| 4102 | Profit growth rate | x1e3 |
| 4105 | Revenue | x1e3 |
| 4106 | Revenue growth rate | x1e3 |
| 4107 | Net profit margin | x1e3 |
| 4108 | Gross profit margin | x1e3 |
| 4109 | Debt-to-asset ratio | x1e3 |
| 4110 | ROE | x1e3 |
| 4801 | Basic EPS | x1e3 |
| 4903 | Free-float market cap | x1e3 |
| 4904 | PS_TTM | x1e5 |

- `term`: 1=Q1, 2=Q2, 3=Q3, 4=Q4, 6=Interim, 9=Q9 cumulative, 10=Latest single quarter, 100=Annual, 200~204=Beat estimates
- `year`: Specify year (optional)
- `duration`: Duration (optional)
- `period_average`: Whether to take daily average (optional)
- `future_duration`: Future periods (optional)

**6. indicator_positional_query — Technical Indicator Positional Relationship**

```json
{"indicator_positional_query": {"first_indicator_name": 11, "second_indicator_name": 12, "period_type": 11, "position": 3}}
```

| Field | Description |
|------|------|
| first_indicator_name | First indicator (see enumeration below) |
| second_indicator_name | Second indicator |
| period_type | K-line period: 1=1min, 2=3min, 3=5min, 4=15min, 5=1hour, 6=30min, 11=daily, 21=weekly, 31=monthly |
| position | 1=Above, 2=Below, 3=Cross above, 4=Cross below |
| continuous_period | Continuous period count (optional) |

Indicator enumeration: 1=PRICE, 11~17=MA(5/10/20/30/60/120/250), 21~27=EMA(5/10/20/30/60/120/250), 31/32/33=KDJ(K/D/J), 41/42/43=MACD(DIF/DEA/MACD), 51=RSI, 61/62/63=BOLL(Upper/Middle/Lower)

**7. indicator_pattern_query — Technical Patterns**

```json
{"indicator_pattern_query": {"name": 1, "period_type": 11}}
```

| name | Meaning |
|------|------|
| 1 | MA bullish alignment |
| 2 | MA bearish alignment |
| 3 | EMA bullish alignment |
| 4 | EMA bearish alignment |
| 11/12 | KDJ golden cross / death cross |
| 13/14 | KDJ top / bottom divergence |
| 21/22 | MACD golden cross / death cross |
| 23/24 | MACD top / bottom divergence |
| 31/32 | RSI cross above / cross below |
| 33/34 | RSI top / bottom divergence |
| 41/42 | Bollinger cross above upper band / cross below lower band |
| 43/44 | Bollinger cross above middle band / cross below middle band |
| 100 | Bullish pattern group |
| 101 | Bearish pattern group |

**8. featured_property_query — Featured Indicators**
```json
{"featured_property_query": {"property": {"name": 5203}, "intervals": [{"lower": {"value": 80000}, "upper": {"value": 120000}}]}}
```

| name | Meaning | Precision Factor |
|------|------|----------|
| 5101 | Chip profit ratio | x1e3 |
| 5102 | Chip concentration | x1e3 |
| 5203 | Beta | x1e5 |
| 5211 | Trading heat | x1e5 |
| 5212 | Search heat | x1e5 |
| 5214 | Combined heat | x1e5 |
| 5320 | Institutional holding ratio | x1e3 |
| 5401 | Analyst rating | — |
| 5403 | Target price | x1e9 |
| 5407 | Morningstar rating | — |

- `period`/`range_period`/`first_custom_param`: Optional parameters
- Use `intervals` array (range list) or `value_set` (discrete value list)

**9. broker_holdings_query — Broker Holdings (HK stocks only)**
```json
{"broker_holdings_query": {"property": {"name": 6101, "days": 5}, "intervals": [{"lower": {"value": 5000}}]}}
```

| name | Meaning |
|------|------|
| 6101 | Concentration |
| 6102 | Broker change |
| 6103 | Broker count |
| 6104 | Broker ranking |
| 6105 | Broker holding ratio |
| 6106 | CCASS ratio |
| 6107 | CCASS change |

**10. kline_shape_query — K-line Patterns (HK stocks only)**
```json
{"kline_shape_query": {"property": {"name": 1, "period": 11}}}
```

| name | Meaning | name | Meaning |
|------|------|------|------|
| 1 | W bottom | 1001 | W top |
| 2 | Triple bottom | 1002 | Triple top |
| 3 | Head & shoulders bottom | 1003 | Head & shoulders top |
| 4 | Rounding bottom | 1004 | Rounding top |
| 5 | Megaphone bottom | 1005 | Megaphone top |
| 6 | Bull flag | 1006 | Bear flag |
| 7 | Bullish symmetrical triangle | 1007 | Bearish symmetrical triangle |
| 8 | Bullish diamond | 1008 | Bearish diamond |
| 9 | Bullish wedge | 1009 | Bearish wedge |
| 10 | Bullish triangle | 1010 | Bearish triangle |
| 2000 | Bullish group | 2001 | Bearish group |

- `period`: 11=daily, 5=1 hour (only these two are supported)

**11. option_query — Option Indicators**
```json
{"option_query": {"property": {"name": 1000}, "intervals": [{"lower": {"value": 200000}}]}}
```

| name | Meaning | Precision Factor |
|------|------|----------|
| 1000 | Underlying IV | x1e6 |
| 1001 | IV Rank | — |
| 1002 | IV percentile | — |
| 1003 | Earnings IV | — |
| 1004 | IV change | — |
| 1005 | IV change rate | — |
| 1006 | HV | — |
| 1007 | IV-HV | — |
| 1008 | IV/HV | — |
| 1009 | Option volume | — |
| 1010 | Option open interest | — |

---

### retrieve_queries Return Columns (9 types)

Each element fills exactly one retrieval object type; response `items[].results[]` aligns with this array in order:

```json
[{"basic_property": {"name": 1102}}, {"simple_property": {"name": 2303}}]
```

| Type | Structure | Description |
|------|------|------|
| basic_property | `{name: int}` | 1101=Code, 1102=Name |
| simple_property | `{name: int}` | Same name values as simple_property_query |
| cumulative_property | `{name: int, days: int, period_average?: bool}` | Same as cumulative_property_query |
| financial_property | `{name: int, term: int, year?: int, ...}` | Same as financial_property_query |
| featured_property | `{name: int, period?: int, range_period?: int, first_custom_param?: int}` | Same as featured_property_query |
| indicator_property | `{name: int, period: int, indicator_params?: [int]}` | Technical indicator values |
| broker_property | `{name: int, days: int, param?: int}` | Broker factor |
| kline_shape_property | `{name: int, period: int}` | K-line pattern |
| option_property | `{name: int, param?: int, period?: int}` | Option factor |

---

### sort / sorts Sorting

```json
{"sort": {"simple_property": {"name": 2301}, "direction": 2}}
```

Sort objects can use `simple_property` / `cumulative_property` / `financial_property` / `featured_property`.

| direction | Meaning |
|-----------|------|
| 1 | Ascending |
| 2 | Descending |
| 3 | Ascending by absolute value |
| 4 | Descending by absolute value |

`sorts` is an array where array order defines sort priority (mutually exclusive with `sort`; `sorts` takes priority).

---

### Range Value Notes

All range `lower`/`upper` `value` fields must be pre-multiplied by the precision factor. For example:
- PE_TTM <= 15 -> `upper.value = 1500000` (15 x 1e5)
- Change rate >= 5% -> `lower.value = 5000` (5 x 1e3)
- `includes`: bool, whether to include the boundary value
- `lower`/`upper` can be used individually (open-ended range)

---

### Error Codes

| ret_code | Trigger Condition | Recommended Action |
|----------|----------|----------|
| 0 | Success (including empty results) | — |
| -3 | screen_queries missing; limit>300; next_key invalid; user_stock_list_mode not 0/1/2 | Fix parameters and retry |
| -5 | Backend rejection (invalid simple_field enum / property name does not exist / market value out of range, etc.) | Check query structure and factor name validity |
| -6 | Gateway output mapping failure | Retry |

---

## quote_plate_list — Sector List

Retrieve all sectors/industry lists under a specified market and sector class. The sector code (plate_code) is itself a stock_id and can be used for quote subscription.

**Supported Markets:** HK / US / SH / SZ / SG / JP / AU / CA / MY / KR (SH and SZ share the A-share sector set)

**Parameters:**
| Parameter | Type | Required | Description |
|------|------|------|------|
| market | string | Yes | Market prefix: HK/US/SH/SZ/SG/JP/AU/CA/MY/KR |
| plate_class | string | Yes | Sector class (see enumeration below, case-sensitive) |

**plate_class Enumeration:**
| Value | Meaning |
|----|------|
| ALL | All sector types |
| INDUSTRY | Industry sectors |
| REGION | Region sectors (**SH/SZ only**, other markets return unsupported) |
| CONCEPT | Concept sectors |
| OTHER | Other sectors |

**Returns `data.plate_list[]`:**

| Field | Type | Description |
|------|------|------|
| code | string | Sector code (with market prefix), e.g. `HK.LIST23618` |
| plate_id | string | Sector ID (without market prefix), e.g. `LIST23618` |
| plate_name | string | English name |
| sc_name | string | Simplified Chinese name |
| tc_name | string | Traditional Chinese name |

**Special Behavior:**
- Returns an empty `plate_list` array when the backend has no data, ret_code is still 0
- Parameter values are case-sensitive (e.g. `INDUSTRY` is valid, `industry` is invalid)

**Error Codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|----------|----------|
| 0 | Success (including empty list) | — |
| -3 | market or plate_class missing / value not in enumeration | Fix parameters and retry |
| -5 | Backend call failure (network/timeout) | Retry |
| -8 | plate_class=REGION but market is not SH/SZ | Use REGION only for SH/SZ |

---

## quote_plate_stock — Sector Constituent Stocks

Retrieve the constituent stock list for a specified sector, supporting approximately 120 sort fields and pagination.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|------|------|------|------|------|
| plate_code | string | Yes | — | Sector code (from plate_list), e.g. `HK.LIST1045` |
| sort_field | string | No | NONE | Sort field (see common enumerations below; full list of ~120 defined in the naming dictionary) |
| ascend | bool | No | true | true=ascending, false=descending |
| price_type | string | No | NORMAL | Price sort scope: NORMAL/BEFORE/AFTER/OVERNIGHT (only affects price-related sort fields) |
| leverage_direction | int | No | 0 | ETF leverage direction filter (ETF sectors only): 0=all, 1=long, 2=short |
| leverage_multiple | int | No | 0 | ETF leverage multiple filter (x1e3, e.g. 2000=2x); 0=all (ETF sectors only) |
| limit | int | No | 200 | Page size, max 1000 |
| next_key | string | No | — | Pagination cursor; leave empty for first request, then pass back `pagination.next_key` |

**sort_field Common Enumerations:**
| Value | Meaning | Value | Meaning |
|----|------|----|------|
| NONE | No sorting | CUR_PRICE | Last price |
| CODE | Code | CHANGE_RATE | Change rate |
| NAME | Name | VOLUME | Volume |
| TURNOVER | Turnover | MARKET_VAL | Total market cap |
| PE | PE ratio | PB | PB ratio |
| TURNOVER_RATIO | Turnover rate | AMPLITUDE | Amplitude |
| PRICE_CHANGE_VAL | Price change amount | CIRC_MARKET_VALUE | Free-float market cap |
| DIVIDEND_RATIO_TTM | TTM dividend yield | LIST_TIME | Listing time |
| CHANGE_RATIO_5_DAYS | 5-day change | CHANGE_RATIO_20_DAYS | 20-day change |
| CHANGE_RATIO_60_DAYS | 60-day change | CHANGE_RATIO_250_DAYS | 250-day change |
| HSG_HOLD_RATIO | Northbound holding ratio | HSG_DAY_FLOW | Northbound daily inflow |
| PRE_CUR_PRICE | Pre-market price | PRE_CHANGE_RATE | Pre-market change rate |
| AFTER_CUR_PRICE | After-hours price | AFTER_CHANGE_RATE | After-hours change rate |
| OVERNIGHT_PRICE | Overnight price | OVERNIGHT_CHANGE_RATE | Overnight change rate |
| HOT | Heat | ETF_LEVERAGE | ETF leverage |

**Returns `data`:**

| Field | Type | Description |
|------|------|------|
| stock_list[] | array | Constituent stock list |
| pagination.total | int | Total constituent stock count |
| pagination.has_more | bool | Whether there are more results |
| pagination.next_key | string | Next page cursor |

**stock_list[] Element:**

| Field | Type | Description |
|------|------|------|
| code | string | Stock code (with market prefix), e.g. `HK.02337` |
| stock_id | int64 | Internal numeric identifier |
| stock_name | string | English name |
| sc_name | string | Simplified Chinese name |
| tc_name | string | Traditional Chinese name |
| stock_type | string | Security type: STOCK/ETF/INDEX/DRVT etc. |
| lot_size | int | Lot size (options = contract shares, futures = contract multiplier) |
| list_time | int64 | Listing timestamp (milliseconds); 0 when no data |

**Special Behavior:**
- `leverage_direction` and `leverage_multiple` only apply to ETF sectors; passed values are ignored for other sectors
- `price_type` only affects the value scope of price-related sort fields
- Returns an empty `stock_list` when a valid sector has no constituent stocks, ret_code is still 0

**Error Codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|----------|----------|
| 0 | Success (including empty list) | — |
| -3 | plate_code missing / invalid market prefix / invalid sort_field / invalid price_type / limit>1000 | Fix parameters and retry |
| -4 | Gateway request assembly failure | Retry |
| -7 | plate_code cannot be resolved to a valid sector | Re-fetch valid sector codes via the plate_list API |

---

## quote_owner_plate — Stock Sector Membership

Retrieve the list of industry/concept sectors that a single stock belongs to. Only supports common stocks and ETFs; indices/warrants/options/futures/bonds typically have no sector membership and return an empty array.

**Supported Markets:** HK / US / SH / SZ / SG / JP / AU / CA / MY

**Parameters:**
| Parameter | Type | Required | Description |
|------|------|------|------|
| symbol | string | Yes | Stock code (path parameter), e.g. `HK.00700`, `US.FUTU` |

**Returns `data.sectors[]`:**

| Field | Type | Description |
|------|------|------|
| name | string | Queried security English name (not the sector name) |
| sc_name | string | Queried security Simplified Chinese name |
| tc_name | string | Queried security Traditional Chinese name |
| plate_code | string | Sector code (with market prefix, can be used for subscription), e.g. `HK.LIST1284` |
| plate_name | string | Sector English name |
| plate_sc_name | string | Sector Simplified Chinese name |
| plate_tc_name | string | Sector Traditional Chinese name |
| plate_type | string | Sector type (see enumeration below) |

**plate_type Enumeration:**
| Value | Meaning |
|----|------|
| INDUSTRY | Industry sector |
| CONCEPT | Concept sector |
| OTHER | Other sector |

**Special Behavior:**
- Single query only, batch not supported; batch requires multiple calls
- Securities with no sector membership (e.g. indices) return an empty `sectors` array, ret_code is still 0

**Error Codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|----------|----------|
| 0 | Success (including empty array) | — |
| -3 | symbol missing or invalid format | Confirm format is `MARKET.CODE` |
| -4 | Gateway/backend call failure | Retry |
| -7 | Code format is valid but security does not exist | Confirm code via search API |
