# Futures / Warrants & CBBCs
## quote_future_info — Futures Contract Information

Batch retrieve static attributes of futures contracts.

**Parameters:**
| Parameter | Type | Required | Description |
|------|------|------|------|
| code_list | string[] | Yes | Futures code list, e.g. `["HK.HSImain"]`, max 400 |

**Returns `data.future_info_list[]`:**

| Field | Type | Description |
|------|------|------|
| code | string | Contract code, e.g. `HK.HSImain` |
| name | string | Contract name, e.g. `HSI moomoores (JUN6)` |
| owner | string | Underlying code or product code (index code for index futures, product name for commodity futures) |
| exchange | string | Exchange: HKEX / CME / CBOT / NYMEX / COMEX / CBOE / SGX / OSE |
| type | string | Contract type: Equity Index / Single Stock / Metals / Energy / Agricultural / Interest Rates / Cryptocurrency / FX |
| size | float | Contract size value |
| size_unit | string | Contract size unit, e.g. `Index Points×HKD`, `barrels` |
| price_currency | string | Quote currency: HKD / USD / CNH / SGD / JPY |
| price_unit | string | Quote unit, e.g. `Index Point` |
| min_change | float | Minimum tick size value |
| min_change_unit | string | Minimum tick unit, e.g. `Index Point`, `USD/barrels` |
| trade_time | string | Trading hours, e.g. `(09:15 - 12:00), (13:00 - 16:30), (17:15 - 03:00)` |
| time_zone | string | Exchange timezone: CCT / ET / CT / SGT / JST |
| last_trade_time | int | Last trading day (millisecond timestamp), fixed at 0 for main/continuous contracts |
| exchange_format_url | string | Exchange contract specification page link |
| delivery_type | string | Delivery method: UNKNOWN / PHYSICAL / CASH |

---

## quote_referencefuture_list — Related Futures

Retrieve the list of related futures contracts for an underlying (typically an index).

**Parameters:**
| Parameter | Type | Required | Description |
|------|------|------|------|
| symbol | string | Yes | Underlying code, e.g. `HK.800000` (Hang Seng Index) |

**Returns `data.reference_list[]`:**

| Field | Type | Description |
|------|------|------|
| code | string | Futures code, e.g. `HK.HSImain`, `HK.HSI2606` |
| stock_name | string | Contract English name |
| sc_name | string | Contract Simplified Chinese name |
| tc_name | string | Contract Traditional Chinese name |
| stock_type | string | Security type, fixed as `FUTURE` |
| lot_size | int | Lot size (contract multiplier), e.g. 50 |
| future_valid | bool | Futures flag, fixed as true |
| future_main_contract | bool | Whether it is a main continuous contract (true=main continuous, false=monthly contract) |
| future_last_trade_time | string | Last trading day (yyyy-MM-dd), empty string for main continuous contracts |
| list_time | int | Listing time (millisecond timestamp), 0 when no record |

---

## quote_warrant_screen — Warrant / CBBC Screening

Warrant screener supporting dozens of filter dimensions and multi-level sorting. Supported markets: HK stocks (1), Singapore (4), Malaysia (15).

**Parameters:**
| Parameter | Type | Required | Default | Description |
|------|------|------|------|------|
| market_type | int | No | 1 | 1=HK, 4=Singapore, 15=Malaysia |
| stock_owner | string | No | — | Quick filter by underlying code (e.g. `HK.00700`), auto-converted to screen_groups condition |
| screen_groups | object[] | No | — | Filter condition list |
| sorts | object[] | No | — | Sort condition list |
| limit | int | No | 200 | Page size, max 1000 |
| next_key | string | No | — | Pagination cursor, empty for first request |
| only_count | bool | No | false | Return count only without details |
| is_delay | bool | No | false | Whether to use delayed data |

### screen_groups Element Structure

Two usage modes — discrete choice (choices) or range filter (interval):

| Field | Type | Description |
|------|------|------|
| field_id | int | Filter field ID (see enumeration below) |
| choices | object[] | Discrete value matching (multiple = OR), element: `{"content_type": 1, "value": <int>}` |
| interval | object | Range matching |
| interval.lower | object | Lower bound: `{"value": <int>, "is_included": true}` |
| interval.upper | object | Upper bound: `{"value": <int>, "is_included": true}` |

**Note:** The interval value must be pre-multiplied by the precision factor (e.g., leverage 10~100 -> value 10000~100000).

### sorts Element Structure

| Field | Type | Description |
|------|------|------|
| sort_field_id | int | Sort field ID (same as field_id) |
| sort_flag | bool | true=descending, false=ascending |

### Request Example

```json
{
  "market_type": 1,
  "limit": 5,
  "screen_groups": [
    {"field_id": 6, "choices": [{"content_type": 1, "value": 1}]},
    {"field_id": 5, "choices": [{"content_type": 1, "value": 54047868453564}]},
    {"field_id": 19, "choices": [{"content_type": 1, "value": 0}]},
    {"field_id": 52, "choices": [{"content_type": 1, "value": 1}]},
    {"field_id": 16, "interval": {"lower": {"value": 10000, "is_included": true}, "upper": {"value": 100000, "is_included": true}}}
  ],
  "sorts": [{"sort_field_id": 16, "sort_flag": true}]
}
```

### field_id Enumeration

| field_id | Name | Description | Type | Precision |
|----------|------|------|------|------|
| 4 | ISSUER_ID | Issuer ID | choice | — |
| 5 | STOCK_OWNER | Underlying ID | choice (value=stock_id) | — |
| 6 | WARRANT_TYPE | Warrant type | choice: 1=Call, 2=Put, 3=Bull, 4=Bear, 5=Inline | — |
| 7 | CONVERSION_RATIO | Conversion ratio | interval | x1e3 |
| 8 | CURRENT_PRICE | Current price | interval | x1e3 |
| 9 | STREET_RATIO | Street ratio % | interval | x1e3 |
| 10 | VOLUME | Volume | interval | — |
| 11 | MATURITY_DATE | Maturity date | interval (seconds timestamp) | — |
| 12 | STRIKE_PRICE | Strike price | interval | x1e3 |
| 13 | PREMIUM | Premium % (can be negative) | interval | x1e5 |
| 14 | RECOVERY_PRICE | Recovery price (CBBC) | interval | x1e3 |
| 15 | IMPLIED_VOLATILITY | Implied volatility % | interval | x1e2 |
| 16 | LEVERAGE_RATIO | Leverage ratio | interval | x1e3 |
| 17 | PRICE_RECOVERY_RATIO | Underlying-to-recovery price % | interval | x1e5 |
| 18 | DELTA | Delta | interval | x1e3 |
| 19 | STATUS | Status | choice: 0=Normal, 2=Suspended, 3=Pending listing | — |
| 20 | IPO_TIME | Listing time | interval (seconds timestamp) | — |
| 21 | BUY_VOL | Best bid volume | interval | — |
| 22 | SELL_VOL | Best ask volume | interval | — |
| 23 | EFFECTIVE_LEVERAGE | Effective leverage | interval | x1e3 |
| 24 | LAST_CLOSE_PRICE | Previous close | interval | x1e3 |
| 25 | TURNOVER | Turnover | interval | — |
| 26 | SELL_PRICE | Best ask price | interval | x1e3 |
| 27 | BUY_PRICE | Best bid price | interval | x1e3 |
| 28 | HIGH_PRICE | High price | interval | x1e3 |
| 29 | LOW_PRICE | Low price | interval | x1e3 |
| 30 | RATIO_ITM_OTM | ITM/OTM % | interval | x1e5 |
| 31 | BREAK_EVEN_POINT | Break-even point | interval | x1e5 |
| 32 | AMPLITUDE | Amplitude % | interval | x1e5 |
| 33 | SCORE_FAXING | SocGen score | interval | x1e5 |
| 34 | LAST_TRADE_DATE | Last trading day | interval (seconds timestamp) | — |
| 35 | STREET_VOLUME | Street volume | interval | — |
| 36 | LOT_SIZE | Lot size | interval | — |
| 37 | ISSUE_SIZE | Issue size | interval | — |
| 38 | IPO_PRICE | Issue price | interval | x1e3 |
| 39 | LOWER_STRIKE_PRICE | Lower strike price (Inline only) | interval | x1e3 |
| 40 | UPPER_STRIKE_PRICE | Upper strike price (Inline only) | interval | x1e3 |
| 41 | IW_PRICE_STATUS | In-bound/out-of-bound status | choice: 0=In-bound, 1=Above out-of-bound, 2=Below out-of-bound | — |
| 42 | SENSITIVITY | Sensitivity | interval | x1e3 |
| 43 | CONVERSION_PRICE | Conversion price | interval | — |
| 44 | CHANGE_RATE | Change rate % | interval | x1e3 |
| 45 | CHANGE_VALUE | Change amount | interval | — |
| 51 | SCORE | Composite score | interval | x1e5 |
| 52 | FILTER_NO_TRADE | Filter zero-trade | choice: 0=No filter, 1=Filter | — |
| 53 | CURRENCY_CODE | Currency | interval | — |
| 54 | STOCK_OWNER_PRICE | Underlying price | interval | x1e3 |

---

### Returns `data.warrants[]` + `pagination`

**pagination:**

| Field | Type | Description |
|------|------|------|
| total | int | Total matches |
| has_more | bool | Whether there is a next page |
| next_key | string | Next page cursor (`"-1"` means no more) |

**warrants[] Common Fields:**

| Field | Type | Description |
|------|------|------|
| code | string | Warrant code, e.g. `HK.18869` |
| name | string | English name |
| sc_name | string | Simplified Chinese name |
| tc_name | string | Traditional Chinese name |
| stock_owner | string | Underlying code, e.g. `HK.00700` |
| type | string | Type: CALL/PUT/BULL/BEAR/INLINE/N/A |
| issuer | string | Issuer code (2 letters, e.g. JP/GS/UB/HS) |
| status | string | Status: NORMAL/STOP_TRADE/PENDING_LISTING/N/A |
| maturity_time | string | Maturity date (yyyy-MM-dd) |
| maturity_timestamp | int | Maturity date millisecond timestamp |
| list_time | string | Listing date (yyyy-MM-dd) |
| list_timestamp | int | Listing date millisecond timestamp |
| last_trade_time | string | Last trading day (yyyy-MM-dd) |
| last_trade_timestamp | int | Last trading day millisecond timestamp |
| lot_size | int | Lot size |
| issue_size | int | Issue size |

**warrants[] Price & Trading:**

| Field | Type | Description |
|------|------|------|
| cur_price | float | Current price |
| last_close_price | float | Previous close |
| high_price | float | High price |
| low_price | float | Low price |
| bid_price | float | Best bid price |
| ask_price | float | Best ask price |
| bid_vol | int | Best bid volume |
| ask_vol | int | Best ask volume |
| volume | int | Volume |
| turnover | float | Turnover |
| price_change_val | float | Price change amount |
| change_rate | float | Change rate (%) |
| amplitude | float | Amplitude (%) |

**warrants[] Derivative Indicators:**

| Field | Type | Description |
|------|------|------|
| strike_price | float | Strike price |
| conversion_ratio | float | Conversion ratio |
| conversion_price | float | Conversion price |
| break_even_point | float | Break-even point |
| premium | float | Premium (%) |
| ipop | float | ITM/OTM % (positive=ITM, negative=OTM) |
| leverage | float | Leverage ratio |
| effective_leverage | float | Effective leverage |
| delta | float | Delta |
| implied_volatility | float | Implied volatility (%) |
| score | float | Composite score |
| street_rate | float | Street ratio (%) |
| street_vol | int | Street volume |

**warrants[] CBBC / Inline Warrant Specific:**

| Field | Type | Description |
|------|------|------|
| recovery_price | float | Recovery price (CBBC only) |
| price_recovery_ratio | float | Underlying-to-recovery price % (CBBC only) |
| upper_strike_price | float | Upper strike price (Inline only) |
| lower_strike_price | float | Lower strike price (Inline only) |
| inline_price_status | string | Inline status: WITH_IN/OUTSIDE/N/A |
