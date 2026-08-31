# Corporate Actions (Dividends / Buybacks / Stock Splits / Rehab)
## quote_corporate_actions_dividends — Dividend History

Get the stock's dividend history, sorted in reverse chronological order, up to 100 records, no pagination.

**Supported markets:** HK / US / SH / SZ / SG / CA / AU / JP — primarily common stocks. ETFs/bonds/warrants/options/futures/indices return an empty list with no dividend events.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| symbol | string | Yes | Stock symbol (path parameter), e.g. `HK.00700` |

**Response `data`:**

| Field | Type | Description |
|-------|------|-------------|
| total_dividend_count | int | Cumulative historical dividend count |
| total_dividend_money | double | Cumulative historical total dividend amount (reporting currency) |
| dividend_list[] | array | Dividend record list (up to 100 records) |

**dividend_list[] elements:**

| Field | Type | Available Markets | Description |
|-------|------|-------------------|-------------|
| fiscal_year | string | HK / A-shares | Fiscal year |
| statement | string | All | Dividend plan description text (e.g. `"Cash Dividend: 5.30000 HKD Per Share"`) |
| dividend_per_share | double | HK / US | Cash dividend per share (reporting currency) |
| currency | string | HK / US | Dividend currency (e.g. `HKD`, `USD`) |
| payout_ratio | double | A-shares | Payout ratio (%) |
| dividend_type | string | Morningstar markets (SG/CA/AU/JP) / ETF | Dividend type |
| process | string | HK / A-shares | Plan progress status (see enum below) |
| ex_date | string | All | Ex-dividend date (yyyy/MM/dd) |
| record_date | string | All | Record date |
| dividend_payable_date | string | All | Dividend payment date |
| pub_date | string | All | Announcement date |

**process enum:**
| Value | Meaning |
|-------|---------|
| Implementation | Implemented |
| Plan | Proposed |

**Field availability by market:**
| Field | HK | US | A-shares (SH/SZ) | Morningstar markets (SG/CA/AU/JP) |
|-------|----|----|-------------------|-----------------------------------|
| fiscal_year | Yes | No | Yes | No |
| dividend_per_share | Yes | Yes | No | No |
| currency | Yes | Yes | No | No |
| payout_ratio | No | No | Yes | No |
| dividend_type | No | No | No | Yes |
| process | Yes | No | Yes | No |

**Error codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|---------------------|
| 0 | Success (including empty list) | — |
| -3 | Invalid symbol format | Fix parameters and retry |
| -7 | Symbol format is valid but security does not exist | Confirm symbol via search API |
| -2/-4/-6 | Gateway internal error | Retry |

---

## quote_corporate_actions_buybacks — Buyback Records

Get the company's share buyback history. Returns HK or A-share data depending on the symbol's market.

**Supported markets:** HK / SH / SZ — common stocks only. US and other markets return success but with an empty list. ETFs/options/warrants have no buyback data.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| symbol | string | Yes | — | Stock symbol (path parameter), e.g. `HK.00700` |
| limit | int | No | 10 | Records per page, max 50 |
| next_key | string | No | — | Pagination cursor; leave empty for first request, pass back `pagination.next_key` for subsequent requests |

**Response `data`:**

| Field | Type | Description |
|-------|------|-------------|
| hk_buy_back_list[] | array/null | HK buyback record list (populated when symbol is HK, otherwise null) |
| a_buy_back_list[] | array/null | A-share buyback record list (populated when symbol is A-share, otherwise null) |

**Pagination (same level as data):**

| Field | Type | Description |
|-------|------|-------------|
| pagination.has_more | bool | Whether there are more records |
| pagination.next_key | string | Next page cursor |

**hk_buy_back_list[] elements (HK):**

| Field | Type | Description |
|-------|------|-------------|
| publ_date | int64 | Announcement date (millisecond timestamp) |
| publ_date_str | string | Announcement date (yyyy-MM-dd) |
| end_date | int64 | Buyback end date (millisecond timestamp) |
| end_date_str | string | Buyback end date (yyyy-MM-dd) |
| buy_back_money | double | Buyback amount (currency per currency field) |
| currency | string | Currency (e.g. `HKD`) |
| buy_back_sum | int | Number of shares bought back |
| percentage | double | Percentage of total shares (%) |
| high_price | double | Highest buyback price |
| low_price | double | Lowest buyback price |
| cumulative_sum | int | Year-to-date cumulative shares bought back |
| cumulative_percentage | double | Year-to-date cumulative percentage of total shares (%) |
| share_type | string | Share class (e.g. `"Ordinary shares"`) |

**a_buy_back_list[] elements (A-shares):**

| Field | Type | Description |
|-------|------|-------------|
| advance_date | int64 | Proposal announcement date (millisecond timestamp) |
| advance_date_str | string | Proposal announcement date (yyyy-MM-dd) |
| start_date | int64 | Buyback period start date (millisecond timestamp) |
| start_date_str | string | Buyback period start date (yyyy-MM-dd) |
| end_date | int64 | Buyback period end date (millisecond timestamp) |
| end_date_str | string | Buyback period end date (yyyy-MM-dd) |
| event_proce_desc | string | Event progress description |
| buy_back_mode | string | Buyback method |
| buy_back_sum | int | Number of shares bought back in this round |
| buy_back_money | double | Buyback amount (CNY) |
| percentage | double | Percentage of total shares (%) |
| value_floor | double | Planned total buyback fund lower limit |
| value_ceiling | double | Planned total buyback fund upper limit |
| price_floor | double | Planned buyback price lower limit |
| price_ceiling | double | Planned buyback price upper limit |

**Special behavior:**
- The market of the symbol determines whether `hk_buy_back_list` or `a_buy_back_list` is populated; the other will be null
- ret_code=0 does not guarantee data; valid symbols with no buyback history return an empty/null list

**Error codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|---------------------|
| 0 | Success (including empty list) | — |
| -3 | limit exceeds 50 or invalid symbol format | Fix parameters and retry |
| -7 | Symbol format is valid but security does not exist | Confirm symbol via search API |
| -5/-6 | Gateway/backend internal error | Retry |

---

## quote_corporate_actions_stock_splits — Stock Splits

Get detailed stock split/consolidation event records for HK stocks (split / consolidation / consolidation-then-split / split-then-consolidation).

**Supported markets:** HK common stocks only. Other markets return an empty `split_list`.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| symbol | string | Yes | Stock symbol (path parameter), e.g. `HK.00700` |

**Response `data.split_list[]`:**

| Field | Type | Description |
|-------|------|-------------|
| dir_deci_pub_date | int64 | Board resolution announcement date (second timestamp) |
| dir_deci_pub_date_str | string | Board resolution announcement date (yyyy-MM-dd) |
| reform_type | string | Change type (see enum below) |
| rate | string | Split/consolidation ratio, arrow notation (e.g. `1→5` means 1 share becomes 5 shares) |
| ex_date | int64 | Ex-rights effective date (second timestamp) |
| ex_date_str | string | Ex-rights effective date (yyyy-MM-dd) |
| sm_deci_date | int64 | Shareholders' meeting resolution date (second timestamp) |
| sm_deci_date_str | string | Shareholders' meeting resolution date (yyyy-MM-dd) |
| scheme_statement | string | Scheme description text |
| new_par_value | float | New par value (reporting currency) |
| temp_share_code | string | Temporary security code |
| temp_share_abbr_name | string | Temporary security short name |
| new_trade_unit | int | New board lot size |
| shares_after_effect | float | Total shares after effective date |
| event_status | string | Event status (e.g. `"Plan Implementation"`) |

**reform_type enum:**
| Value | Meaning |
|-------|---------|
| SPLIT | Stock split |
| CONSOLIDATION | Share consolidation |
| CONSOLIDATION_THEN_SPLIT | Consolidation then split |
| SPLIT_THEN_CONSOLIDATION | Split then consolidation |

**Special behavior:**
- Timestamps are in seconds (not milliseconds)
- ret_code=0 with no split events returns an empty `split_list` array

**Error codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|---------------------|
| 0 | Success (including empty list) | — |
| -3 | Invalid symbol format | Fix parameters and retry |
| -7 | Symbol format is valid but security does not exist | Confirm symbol via search API |
| -2/-4/-6 | Gateway internal error | Retry |

---

## quote_corporate_actions_rehab — Rehabilitation Factors

Get ex-rights/ex-dividend event details and rehabilitation factors, used for client-side K-line price adjustment, dividend backtracking, and corporate action timeline analysis.

**Supported markets:** HK / US / SH / SZ / CA / AU / JP / SG — common stocks only. Indices/ETFs/warrants/options/futures/bonds typically return an empty array.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| symbol | string | Yes | — | Stock symbol (path parameter), e.g. `HK.00700` |
| divi_mode | string | No | include_divi | Cash dividend handling mode: `exclude_divi`=exclude cash dividends (Yahoo/Bloomberg convention), `include_divi`=include cash dividends (A-share/moomoo convention), `compat`=compatibility mode (JP excludes / non-JP includes, deprecated) |

**Response `data.rehabs[]`:**

| Field | Type | Description |
|-------|------|-------------|
| ex_div_date | string | Ex-dividend date (yyyy-MM-dd) |
| action_types | string[] | Set of event types on that day (see enum below) |
| desc_sc | string | Event description (Simplified Chinese) |
| desc_tc | string | Event description (Traditional Chinese) |
| desc_en | string | Event description (English; may be empty for some events) |
| forward_adj_factorA | double | Forward adjustment factor A for that day |
| forward_adj_factorB | double | Forward adjustment factor B for that day |
| backward_adj_factorA | double | Backward adjustment factor A for that day |
| backward_adj_factorB | double | Backward adjustment factor B for that day |
| cum_forward_adj_factorA | double | Cumulative forward adjustment factor A (from listing to this event) |
| cum_forward_adj_factorB | double | Cumulative forward adjustment factor B |
| cum_backward_adj_factorA | double | Cumulative backward adjustment factor A |
| cum_backward_adj_factorB | double | Cumulative backward adjustment factor B |
| split_ratio | double | Stock split/consolidation ratio (<1=split e.g. 0.25 means 1-for-4 split, >1=consolidation, =1=no change) |
| join_ratio | double | Consolidation ratio |
| bonus_ratio | double | Bonus share ratio (N shares per base shares) |
| bonus_base | double | Bonus share base |
| transfer_ratio | double | Share transfer ratio |
| transfer_base | double | Share transfer base |
| transfer_ert | double | Share transfer event identifier value |
| allotment_ratio | double | Rights issue ratio |
| allotment_price | double | Rights issue price |
| add_base | double | Additional issuance base |
| add_ert | double | Additional issuance event identifier value |
| dividend | double | Cash dividend per share (original currency) |
| special_dividend | double | Special dividend per share (original currency) |
| special_dividend_base | double | Special dividend base |
| spin_off_ratio | double | Spin-off ratio |

**action_types enum:**
| Value | Meaning |
|-------|---------|
| SPLIT | Stock split |
| JOIN | Share consolidation |
| BONUS | Bonus shares |
| TRANSFER | Share transfer |
| ALLOTMENT | Rights issue |
| ADD | Additional issuance |
| DIVIDEND | Cash dividend |
| SPECIAL_DIVIDEND | Special dividend |
| SPIN_OFF | Spin-off listing |

**Special behavior:**
- Returns all ex-rights/ex-dividend events since listing; no time range parameter
- Results are sorted by `ex_div_date` in ascending order
- Adjustment factors are already scaled by 1e-9; callers can use the float values directly without further processing

**Error codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|---------------------|
| 0 | Success (including empty array) | — |
| -3 | divi_mode value not in enum | Fix parameters and retry |
| -7 | Symbol cannot be resolved to a valid security | Confirm symbol via search API |
