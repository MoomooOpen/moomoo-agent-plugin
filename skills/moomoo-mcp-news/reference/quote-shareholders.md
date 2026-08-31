# Shareholders & Corporate Actions Tool Reference

## quote_shareholders_overview — Shareholder Overview

Get the company's shareholding overview, including top 5 shareholders, holder type distribution, and available reporting period list.

**Supported markets:** HK / US / SG / CA / AU / JP — common stocks only. A-shares (SH/SZ/BJ) have no data; ETFs/warrants/options/indices return an empty array.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| symbol | string | Yes | — | Stock symbol (path parameter), e.g. `HK.00700` |
| period_id | int | No | 0 | Reporting period ID; the backend currently ignores this parameter and always returns the latest period, only 0 is allowed |

**Response `data`:**

| Field | Type | Description |
|-------|------|-------------|
| main_holder[] | array | Top 5 shareholders + "Others" summary row |
| holder_type[] | array | Distribution by holder category |
| holding_period[] | array | Available reporting period list |

**main_holder[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| name | string | Shareholder name |
| holder_pct | double | Shareholding percentage (%) |
| holder_id | int/null | Shareholder ID; null for "Others" row |
| static_date | int | Data timestamp (seconds) |
| static_date_str | string | Statistics date (yyyy-MM-dd) |

**holder_type[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| name | string | Holder category name (e.g. "Traditional Investment Manager", "Venture Capital / Private Equity", "Individual", "Other") |
| holder_pct | double | Percentage (%) |
| holder_id | null | Always null |

**holding_period[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| period_id | int | Reporting period ID (used for querying holder_detail and other interfaces) |
| period_text | string | Period text, format `YYYY/QN` (e.g. `2026/Q2`) |

**Error codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|---------------------|
| 0 | Success (including empty array) | — |
| -3 | Invalid period_id | Fix parameters and retry |
| -7 | Symbol cannot be resolved to a valid security | Confirm symbol via search API |
| -2/-4/-6 | Gateway internal error | Retry |

---

## quote_shareholders_holder_detail — Shareholder Holding Details

Get the shareholder holding detail list for a specified stock, supporting filtering by holder type, reporting period, and shareholder ID, with sorting.

**Supported markets:** HK / US / SG / JP / CA / AU — common stocks only. A-shares (SH/SZ/BJ), ETFs, indices, warrants, options, and futures are not supported.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| symbol | string | Yes | — | Stock symbol (path parameter), e.g. `HK.00700` |
| request_type | int | No | 1000 | Holder type filter (see enum below) |
| period_id | int | No | 0 | Reporting period ID (obtained from overview's holding_period), 0=latest |
| holder_id | int | No | 0 | Specified shareholder ID; when non-zero, returns only that shareholder's cross-period holdings, sort_column is ignored, results ordered by period_id ascending |
| sort_column | int | No | 61 | Sort field: 61=Holding quantity, 62=Holding change count |
| sort_type | int | No | 0 | 0=Descending, 1=Ascending |
| limit | int | No | 10 | Records per page, max 50 |
| next_key | string | No | — | Pagination cursor; leave empty for first request, pass back `pagination.next_key` for subsequent requests |

**request_type enum (OwnershipType):**
| Value | Meaning | Value | Meaning |
|-------|---------|-------|---------|
| 1000 | All (default) | 7 | Insurance Company |
| 1 | Other Institution | 8 | Bank / Investment Bank |
| 2 | Traditional Investment Manager | 9 | Family Office / Trust |
| 3 | Hedge Fund Manager | 10 | Sovereign Wealth Fund |
| 4 | VC / PE Firm | 11 | REIT |
| 5 | Corporate Pension Plan | 12 | Structured Finance Pool Manager |
| 6 | Foundation Sponsor | 13 | Union Pension Plan |
| 14 | Government Pension Plan | 100 | Individual |
| 15 | Endowment Fund | 200 | ADR |
| 300 | Public Corporation | 400 | Private Corporation |
| 500 | State-Owned Shares | | |

**Response `data`:**

| Field | Type | Description |
|-------|------|-------------|
| holders[] | array | Shareholder list |
| pagination.has_more | bool | Whether there are more records |
| pagination.next_key | string | Next page cursor |

**holders[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| holder_id | int | Shareholder ID |
| name | string | Shareholder name |
| period_text | string | Reporting period (YYYY/QN format) |
| holder_quantity | int | Number of shares held |
| holder_quantity_change | int | Holding change count (positive=increase, negative=decrease) |
| holder_pct | float | Holding percentage (%) |
| holder_pct_change | float | Holding percentage change (%) |
| holding_date | int64 | Holding date (millisecond timestamp) |
| holding_date_str | string | Holding date (yyyy-MM-dd) |
| close_price | float | Period close price |
| price_change_pct | float | Price change percentage (%) |
| source_group_name | string | Data source (e.g. "Annual Report") |
| update_time | int64 | Data update time (millisecond timestamp) |
| update_time_str | string | Data update time (formatted string) |

**Special behavior:**
- When `holder_id` is non-zero, returns that shareholder's holding records across all reporting periods; `sort_column` is ignored, results ordered by period_id ascending
- `period_id=0` means the latest reporting period; historical periods can be obtained from `shareholders_overview`'s `holding_period`

**Error codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|---------------------|
| 0 | Success | — |
| -3 | Parameter type error / invalid enum value / out of range | Fix parameters and retry |
| -7 | Symbol cannot be resolved to a valid security | Confirm symbol via search API |
| -10 | Valid request but no shareholder data | Confirm security and filter conditions are within supported range |
| -2/-4/-6 | Gateway internal error | Retry |
| portfolio_ratio | double | Percentage of this stock in the portfolio |

---

## quote_shareholders_holding_changes — Holding Changes

Get shareholder holding increase/decrease change records, supporting multi-dimensional sorting and direction filtering.

**Supported markets:** HK / US / JP / SG / CA / AU — common stocks only. A-shares have no data.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| symbol | string | Yes | — | Stock symbol (path parameter), e.g. `HK.00700` |
| holder_category | string | No | INSTITUTIONS | Holder scope (see enum below) |
| filter_type | int | No | 0 | Direction filter (see enum below) |
| sort_column | int | No | 1 | Sort field (see enum below) |
| sort_type | int | No | 0 | 0=Descending, 1=Ascending |
| limit | int | No | 30 | Records per page, max 50 |
| next_key | string | No | — | Pagination cursor; leave empty for first request, pass back `pagination.next_key` for subsequent requests |

**holder_category enum:**
| Value | Meaning |
|-------|---------|
| INSTITUTIONS | Institutions (default, OwnershipType 1-15) |
| INDIVIDUALS | Individuals |
| CORPORATIONS | Corporate entities |
| ALL | All |

**filter_type enum:**
| Value | Meaning |
|-------|---------|
| 0 | No filter |
| 1 | Increase |
| 2 | Decrease |
| 3 | New position |
| 4 | Close position |

**sort_column enum:**
| Value | Meaning |
|-------|---------|
| 1 | Change quantity |
| 2 | Holding date |
| 3 | Change ratio |
| 4 | Change amount |
| 5 | Shareholding percentage |

**Response `data`:**

| Field | Type | Description |
|-------|------|-------------|
| changes[] | array | Change record list |
| pagination.has_more | bool | Whether there are more records |
| pagination.next_key | string | Next page cursor |

**changes[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| name | string | Shareholder name |
| holder_id | int | Shareholder ID |
| holder_type | string | Shareholder type (English text) |
| holder_type_id | int | Shareholder type ID |
| period_text | string | Reporting period (YYYY/QN format) |
| holding_date | int64 | Holding date (millisecond timestamp) |
| holding_date_str | string | Holding date (yyyy-MM-dd) |
| share_change_num | int | Share change count (positive=increase, negative=decrease) |
| share_num | int | Current number of shares held |
| share_ratio | float | Holding percentage (%) |
| share_ratio_change | float | Holding percentage change (%) |
| shares_change_price | int | Change reference amount |

**Error codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|---------------------|
| 0 | Success | — |
| -3 | Invalid symbol format or parameters out of range | Fix parameters and retry |
| -7 | Symbol format is valid but security does not exist | Confirm symbol via search API |
| -10 | Valid security but no holding change data | Normal empty result, no retry needed |

---

## quote_shareholders_institutional — Institutional Holding Statistics

Get institutional holding statistics aggregated by reporting period, including institution count, total shares held, period-over-period changes, and period prices.

**Supported markets:** HK / US / CA / AU / SG / JP — common stocks only (some DRs/REITs/ETFs/funds/warrants may have coverage). A-shares, indices, futures, options, and bonds have no data.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| symbol | string | Yes | — | Stock symbol (path parameter), e.g. `HK.00700` |
| limit | int | No | 10 | Reporting periods per page, max 50 |
| next_key | string | No | — | Pagination cursor; leave empty for first request, pass back `pagination.next_key` for subsequent requests; `"-1"` means no more |

**Response `data`:**

| Field | Type | Description |
|-------|------|-------------|
| holders[] | array | Summary list by period |
| pagination.has_more | bool | Whether there are more records |
| pagination.next_key | string | Next page cursor |

**holders[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| period_text | string | Reporting period text (YYYY/QN format, e.g. `2026/Q2`) |
| institution_quantity | int | Number of institutions |
| institution_quantity_change | int | Institution count period-over-period change |
| holder_quantity | int | Total institutional shares held |
| holder_quantity_change | int | Total shares held period-over-period change |
| holder_pct | float | Institutional holding percentage (%) |
| holder_pct_change | float | Holding percentage period-over-period change (%) |
| close_price | float | Period close price |
| open_price | float | Period open price |
| last_close_price | float | Period previous close price |
| update_time | int64 | Data update time (millisecond timestamp) |
| update_time_str | string | Data update time (formatted string) |

**Special behavior:**
- Period text is calculated by advancing the quarter-end timestamp by approximately 45 days (falling into the next quarter)
- Valid security but no institutional data returns ret_code=-10

**Error codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|---------------------|
| 0 | Success | — |
| -3 | Invalid symbol format or limit out of range | Fix parameters and retry |
| -7 | Symbol format is valid but security does not exist | Confirm symbol via search API |
| -10 | Valid security but no institutional holding data | Normal empty result, no retry needed |

---

## quote_insider_holder_list — Insider Holdings

Get the company insider (executive/director) holding list and summary statistics.

**Supported markets:** Primarily covers US; JP/AU/CA common stocks also have data. Markets without insider disclosure requirements such as HK/A-shares/ETFs return an empty list.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| symbol | string | Yes | — | Stock symbol (path parameter), e.g. `US.AAPL` |
| limit | int | No | 10 | Records per page, max 30 |
| next_key | string | No | — | Pagination cursor; leave empty for first request, pass back `pagination.next_key` for subsequent requests |

**Response `data`:**

| Field | Type | Description |
|-------|------|-------------|
| insiders[] | array | Insider list |
| pagination.total | int | Total number of insiders |
| pagination.has_more | bool | Whether there are more records |
| pagination.next_key | string | Next page cursor; `"-1"` means no more |

**insiders[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| holder_id | int | Insider ID (used for holder_id filtering in insider_trade_list) |
| name | string | Name |
| title | string | Position/title |
| holder_quantity | int | Number of shares held |
| holder_pct | float | Holding percentage (%) |
| insider_total_count | int | Total number of company insiders |
| insider_bought_count | int | Number of insiders with buy records |
| insider_sold_count | int | Number of insiders with sell records |

**Special behavior:**
- ret_code=0 with an empty `insiders` array is normal (the security has no insider data)
- HK/A-shares have no insider disclosure requirements and always return an empty list

**Error codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|---------------------|
| 0 | Success (including empty list) | — |
| -3 | limit exceeds 30 or invalid symbol format | Fix parameters and retry |
| -7 | Symbol format is valid but security does not exist | Confirm symbol via search API |

---

## quote_insider_trade_list — Insider Trades

Get company insider (director/executive/5%+ shareholder) trading records, sourced from US SEC insider filings (Form 3/4/144).

**Supported markets:** Primarily covers US-listed companies and their overseas dual-listed/ADR counterparts. A-shares, most HK stocks, ETFs, etc. have no such disclosure data and return an empty array.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| symbol | string | Yes | — | Stock symbol (path parameter), e.g. `US.AAPL` |
| holder_id | int | No | 0 | Filter by specific insider (obtained from insider_holder_list), 0=no filter |
| limit | int | No | 10 | Records per page, max 50 |
| next_key | string | No | — | Pagination cursor; leave empty for first request, pass back `pagination.next_key` for subsequent requests |

**Response `data`:**

| Field | Type | Description |
|-------|------|-------------|
| trades[] | array | Trade record list |
| pagination.total | int | Total number of trade records |
| pagination.has_more | bool | Whether there are more records |
| pagination.next_key | string | Next page cursor; `"-1"` means no more |

**trades[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| holder_id | int | Insider ID |
| name | string | Name |
| title | string | Position/title |
| trade_shares | int | Number of shares traded (positive=buy, negative=sell) |
| min_trade_date | int64 | Earliest trade date in the range (millisecond timestamp) |
| max_trade_date | int64 | Latest trade date in the range (millisecond timestamp) |
| min_trade_date_str | string | Earliest trade date (yyyy-MM-dd) |
| max_trade_date_str | string | Latest trade date (yyyy-MM-dd) |
| min_price | float | Lowest trade price in the range |
| max_price | float | Highest trade price in the range |
| security_holder_quantity | int | Current number of shares held |
| transaction_type | string | Transaction type (see enum below) |
| source_group_name | string | Filing type source (see enum below) |
| is_proposed_sale_of_securities | bool | Whether this is a proposed sale of securities (Form 144) |
| security_description | string | Security description (e.g. "Common Stock") |

**transaction_type enum:**
| Value | Meaning |
|-------|---------|
| Buy | Buy |
| Sell | Sell |
| Exercise and Sell | Exercise and Sell |
| Other Acquisition | Other Acquisition |

**source_group_name enum:**
| Value | Meaning |
|-------|---------|
| Form 3 | Initial ownership filing |
| Form 4 | Change in ownership filing |
| Form 144 | Proposed sale of securities filing |

**Special behavior:**
- Securities with no insider trade data return an empty `trades` array, ret_code is still 0
- When `holder_id` is non-zero, only that insider's trade records are returned

**Error codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|---------------------|
| 0 | Success (including empty array) | — |
| -3 | limit exceeds 50 or invalid symbol format | Fix parameters and retry |
| -7 | Symbol format is valid but security does not exist | Confirm symbol via search API |
| -2/-5/-6 | Gateway/backend internal error | Retry |

---
