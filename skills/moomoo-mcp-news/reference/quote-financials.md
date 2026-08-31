# Fundamentals & Research Tool Reference

## quote_financials_statements — Financial Statements

Get income statement / balance sheet / cash flow statement / key indicators.

**Supported markets:** HK, US, SH, SZ, BJ, SG, JP, AU, CA (only corporate securities with public financial reports; indices, plates, ETFs, funds, warrants, options, futures, forex, and cryptocurrencies return no_data)

### Request Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| symbol | string | Yes | — | Stock symbol, e.g. `HK.00700`, `US.AAPL` |
| statement_type | int | No | 1 | Statement type (see enum below) |
| financial_type | int | No | 10 | Fiscal period selector (see enum below) |
| limit | int | No | 10 | Records per page, range 1~50 |
| next_key | string | No | — | Pagination cursor, leave empty for first request, pass back previous `pagination.next_key` for subsequent requests |
| currency_code | string | No | — | ISO 4217 currency code (omit to use the statement's native currency), e.g. `USD`/`HKD`/`CNY` |

### statement_type Enum

| Value | Meaning |
|-------|---------|
| 1 | Income Statement |
| 2 | Balance Sheet |
| 3 | Cash Flow Statement |
| 4 | Key Financial Indicators |

### financial_type Enum

This enum is used for both request parameters and response fields. Some values serve only as multi-period selectors for requests.

**Specific periods (valid for both request and response):**

| Value | Meaning | Description |
|-------|---------|-------------|
| 1 | Q1 | First quarter single quarter |
| 2 | Q2 | Second quarter single quarter |
| 3 | Q3 | Third quarter single quarter |
| 4 | Q4 | Fourth quarter single quarter |
| 5 | Q6 cumulative | Interim report (first half cumulative) |
| 6 | Q9 cumulative | First three quarters cumulative |
| 7 | Annual | Full year (Annual/FY) |
| 70 | Malaysia single quarter | period_text suffix SQ |
| 71 | Malaysia cumulative quarter | period_text suffix CQ |

**Multi-period selectors (request only):**

| Value | Meaning | Applicable API |
|-------|---------|----------------|
| 0 | Auto-match | `revenue_breakdown` (auto-matches period type by date) |
| 8 | Aggregated quarterly | `revenue_breakdown` (US only, to disambiguate Q4 and FY) |
| 9 | All single quarters | `statements` (returns Q1~Q4) |
| 10 | Single quarters + annual | `statements` (default, returns Q1~Q4 and annual) |
| 11 | All cumulative | `statements` (returns Q1, Q6, Q9, annual) |
| 102 | All cumulative | `operational_efficiency` (returns Q1, Q6, Q9, annual) |

### Response Structure

**Top-level `data`:**

| Field | Type | Description |
|-------|------|-------------|
| report_list[] | array | Report list |
| pagination.has_more | bool | Whether there is more data |
| pagination.next_key | string | Next page cursor (stop paging when `has_more=false`) |

**report_list[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| date_time | int64 | Reporting period end timestamp (**milliseconds**) |
| fiscal_year | int | Fiscal year, e.g. 2026 |
| financial_type | int | Specific period type of this report (values same as request parameter enum) |
| structure | int | Report structure number (market x industry), see enum below |
| structure_name | string | Structure name, e.g. `NORMAL_HK` |
| period_text | string | Period text, e.g. `"2024/FY"`, `"2024/Q1"` |
| currency_code | string | Currency code, e.g. `CNY` |
| accounting_standards | string | Accounting standards, e.g. `IAS` (IFRS), `US_GAAP` |
| auditor_report | string | Auditor's opinion (may be null) |
| item_list[] | array | Statement line item list |

**item_list[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| field_id | int | Line item ID (item sets differ across structures) |
| display_name | string | Line item English name, e.g. `Total Revenue`, `Net Income` |
| value_type | string | Value type: `amount`=monetary amount, `percent`=percentage |
| data | number | Line item value (monetary amounts are in the smallest unit of the native currency) |
| yoy | number | Year-over-year growth rate (%) |
| qoq | number | Quarter-over-quarter growth rate (%) |

### structure (Report Structure) Enum

| Value | structure_name | Applicable Scenario |
|-------|----------------|---------------------|
| 1 | NORMAL_KCB | STAR Market - General |
| 2 | BANK_KCB | STAR Market - Financial |
| 3 | NORMAL_A | A-shares - General |
| 4 | BANK_A | A-shares - Financial |
| 5 | NORMAL_HK | HK - General |
| 6 | BANK_HK | HK - Banking |
| 7 | INSURANCE_HK | HK - Insurance |
| 8 | NORMAL_MSTAR | US/SG/CA/AU - General |
| 9 | BANK_MSTAR | US/SG/CA/AU - Banking |
| 10 | INSURANCE_MSTAR | US/SG/CA/AU - Insurance |
| 11 | NONNORMAL_MSTAR | US/SG/CA/AU - General (Non-standard) |
| 12 | NONBANK_MSTAR | US/SG/CA/AU - Banking (Non-standard) |
| 13 | NONINSURANCE_MSTAR | US/SG/CA/AU - Insurance (Non-standard) |
| 14 | NORMAL_MAIN_INDEX_US | US Key Indicators - General |
| 15 | BANK_MAIN_INDEX_US | US Key Indicators - Banking |
| 16 | INSURANCE_MAIN_INDEX_US | US Key Indicators - Insurance |
| 17 | NORMAL_MAIN_INDEX_MSTAR | SG/CA/AU Key Indicators - General |
| 18 | BANK_MAIN_INDEX_MSTAR | SG/CA/AU Key Indicators - Banking |
| 19 | INSURANCE_MAIN_INDEX_MSTAR | SG/CA/AU Key Indicators - Insurance |

### Error Codes

| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|---------------------|
| -3 | Invalid parameter (missing symbol, statement_type not in [1,2,3,4], limit>50) | Fix parameters and retry |
| -7 | Symbol cannot be resolved | Confirm symbol via search API |
| -10 | Security is valid but no data for this statement/period | Normal empty result, no retry needed |
| -4/-6 | Gateway internal error | Retry |

---

## quote_financials_revenue_breakdown — Revenue Breakdown

Get the company's revenue breakdown by product / industry / region / business. Only dimensions with data are returned.

**Supported markets:** HK, US, SH, SZ, BJ, SG, CA, AU (equity securities only)

### Request Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| symbol | string | Yes | — | Stock symbol, e.g. `HK.00700` |
| date | int | No | 0 | Fiscal period end timestamp (**seconds**), 0=latest period. Can be selected from the returned `screen_date_list` |
| financial_type | int | No | 0 | Fiscal period type: 0=auto-match by backend (use 0 for non-US markets); US only: 7=Annual, 8=Aggregated quarterly (to disambiguate Q4 and FY at the same timestamp) |
| currency_code | string | No | — | ISO 4217 currency code (omit to use the statement's native currency) |

### Response Structure

**Top-level `data`:**

| Field | Type | Description |
|-------|------|-------------|
| period | string | Current period text, e.g. `"2025/FY"`, `"2025/H1"` |
| currency_code | string | Currency code |
| breakdown_list[] | array | Revenue breakdown by dimension (only dimensions with data are returned) |
| screen_date_list[] | array | Available reporting period dropdown list |

**breakdown_list[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| type | int | Breakdown dimension (see enum below) |
| item_list[] | array | Line items under this dimension |

### Breakdown Dimension type Enum

| Value | Meaning | Typical Coverage Markets |
|-------|---------|--------------------------|
| 1 | By Product | A-shares, HK |
| 2 | By Industry | A-shares |
| 4 | By Region | A-shares, HK, US/SG/CA/AU |
| 8 | By Business | HK, US/SG/CA/AU |

**item_list[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| name | string | Item name |
| main_oper_income | number | Operating revenue (native currency unit) |
| ratio | number | Revenue share (%) |

**screen_date_list[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| date | int | Fiscal period end timestamp (seconds) |
| period_text | string | Period text, e.g. `"2025/FY"` |
| financial_type | int | Period type |

### Error Codes

| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|---------------------|
| -3 | Invalid symbol format or illegal financial_type | Fix parameters and retry |
| -7 | Symbol cannot be resolved | Confirm symbol via search API |
| -10 | Security is valid but no revenue breakdown data | Confirm it is an equity security in a supported market |
| -5 | Gateway/backend error | Retry |

---

## quote_financials_earnings_price_history — Earnings Day Price Performance

Get the stock price sequence around each historical earnings disclosure day (disclosure day +/-15 days, totaling 30 data points), including option implied volatility expectations and IV Crush.

**Supported markets:** HK, US, SH, SZ (common stocks only; ETFs, indices, warrants, options return no_data)

### Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| symbol | string | Yes | Stock symbol, e.g. `HK.00700`, `US.AAPL` |

### Response Structure

Returns `data.records[]`, with **30 records** generated per earnings period (`schedule_delta` from -15 to +14), providing a complete price window centered on the disclosure day.

**records[] elements — Earnings period identification:**

| Field | Type | Description |
|-------|------|-------------|
| fiscal_year | int | Fiscal year |
| financial_type | int | Earnings period type |
| period_text | string | Period text, e.g. `"2025/Q4"` |
| is_current | bool | Whether this is the most recent period |
| pub_trading_day | int64 | Disclosure corresponding trading day timestamp (**milliseconds**) |
| pub_trading_day_str | string | Disclosure trading day (yyyy-MM-dd) |
| pub_time | int64 | Disclosure timestamp (**seconds**) |
| pub_time_str | string | Disclosure time (yyyy-MM-dd HH:mm:ss) |
| pub_type | int | Disclosure timing type (pre-market/after-hours, etc.) |

**records[] elements — Option expected volatility and IV Crush:**

| Field | Type | Description |
|-------|------|-------------|
| predict_vola_ratio_newest | float | Latest expected volatility (%) |
| predict_vola_ratio_highest | float | Highest expected volatility (%) |
| predict_vola_val_newest | float | Latest expected volatility price magnitude |
| predict_vola_val_highest | float | Highest expected volatility price magnitude |
| option_iv_crush | float | Post-earnings IV Crush (percentage points) |
| option_strike_date_iv_crush | float | Earnings expiry date IV Crush (percentage points) |

**records[] elements — Earnings corresponding trading day quotes (OHLCV):**

| Field | Type | Description |
|-------|------|-------------|
| trading_day | int64 | Earnings corresponding trading day timestamp (**milliseconds**) |
| trading_day_str | string | Trading day (yyyy-MM-dd) |
| open_price | float | Open price |
| close_price | float | Close price |
| highest_price | float | High price |
| lowest_price | float | Low price |
| last_close_price | float | Previous close price |
| volume | int | Volume (shares) |

**records[] elements — Offset day close price series:**

| Field | Type | Description |
|-------|------|-------------|
| schedule_delta | int | Day offset relative to disclosure day (-15 to +14, 0=disclosure day) |
| schedule_close_price | float | Close price on that offset day |

### Error Codes

| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|---------------------|
| -3 | Invalid symbol format | Fix parameters and retry |
| -7 | Symbol cannot be resolved | Confirm symbol via search API |
| -8 | Market not in HK/US/SH/SZ | Only common stocks from these four markets are supported |
| -10 | Security is valid but no earnings day price data | Normal empty result, no retry needed |
| -6 | Gateway internal error | Retry |

---

## quote_financials_earnings_price_move — Earnings Day Quote Series

Get daily quote series centered on the disclosure day across multiple earnings periods (including OHLCV and option IV/HV), with a summary of the average price change on earnings days over the most recent N periods.

**Supported markets:** HK, US, SH, SZ, CA, AU (common stocks/ADRs only; options, futures, forex, indices return unsupported)

### Request Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| symbol | string | Yes | — | Stock symbol, e.g. `US.AAPL` |
| count | int | No | 10 | Return the most recent N earnings periods, range 1~50 |
| overview_count | int | No | 8 | Number of periods used for average calculation, range 1~50; actual value is min(overview_count, count) |

### Response Structure

**Top-level `data`:**

| Field | Type | Description |
|-------|------|-------------|
| records[] | array | Daily quote records for each earnings period (flat list, ordered by period + offset day) |
| overview_recent_period_count | int | Actual number of periods used in the overview statistics N |
| overview_avg_earnings_day_change_pct | float | Average earnings day price change over the most recent N periods (%), e.g. 1.15 means +1.15% |

**records[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| fiscal_year | int | Fiscal year |
| financial_type | int | Earnings period type |
| period_text | string | Period text, e.g. `"2026/Q1"` |
| pub_trading_day | int | Disclosure day corresponding trading day timestamp (**seconds**) |
| pub_trading_day_str | string | Disclosure trading day (yyyy-MM-dd) |
| pub_type | int | Disclosure timing type (pre-market/after-hours, etc.) |
| price_info_index | int | Index position of the disclosure day (day_offset=0) in this period's sub-sequence |
| day_offset | int | Day offset relative to disclosure day (negative=before, 0=disclosure day, positive=after) |
| trading_day | int | Current row trading day timestamp (**seconds**) |
| trading_day_str | string | Current row trading day (yyyy-MM-dd) |
| open_price | float | Open price |
| close_price | float | Close price |
| highest_price | float | High price |
| lowest_price | float | Low price |
| last_close_price | float | Previous close price |
| option_iv | float | Option implied volatility (%), 0 when no option data |
| option_hv | float | Option historical volatility (%), 0 when no option data |
| volume | int | Volume (shares) |
| volume_precision | int | Volume precision n, actual volume = volume / 10^n (usually 0) |

### Error Codes

| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|---------------------|
| 0 | Success (records is empty for valid symbol with no data) | — |
| -3 | Invalid parameter (count>50, etc.) | Fix parameters and retry |
| -7 | Symbol cannot be resolved | Confirm symbol via search API |
| -8 | Non-equity security | Only common stocks are supported |
| -4/-6 | Gateway internal error | Retry |

---
