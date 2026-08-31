# Valuation / Ratings / Company Research
## quote_valuation_detail — Valuation Analysis

Get PE/PB/PS valuation trends and historical percentiles.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| symbol | string | Yes | — | Stock or index symbol |
| valuation_type | int | No | 1 | 1=PE, 2=PB, 3=PS |
| interval_type | int | No | 3 | Time span: 1=3 months, 2=6 months, 3=1 year, 4=3 years, 5=Since May 2019, 6=5 years, 7=10 years, 8=2 years, 9=20 years, 10=30 years |

**Response `data`:**

| Field | Type | Description |
|-------|------|-------------|
| valuation_type | int | Valuation type |
| last_update_time | int64 | Last update timestamp (seconds) |
| last_update_time_str | string | Update time string |
| trend | object | Trend data |
| market_distribution | object | Market distribution |
| plate_distribution | object | Plate distribution (stocks only) |
| profit_growth_rate | object | Profit growth rate (stocks only) |

**trend sub-object:**

| Field | Type | Description |
|-------|------|-------------|
| current_value | double | Current valuation |
| average_value | double | Average |
| avg_plus_std | double | Average + 1 standard deviation |
| avg_minus_std | double | Average - 1 standard deviation |
| forward_value | double | Forward valuation (stocks only) |
| valuation_percentile | double | Percentile |
| historical_items[] | array | Historical data points |

**historical_items[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| time | int64 | Timestamp |
| time_str | string | Date string |
| value | double | Valuation |
| plate_value | double | Plate average (stocks only) |

**market_distribution sub-object:**

| Field | Type | Description |
|-------|------|-------------|
| sections[] | array | Distribution intervals |
| total | int | Total stock count |
| ranking | int | Ranking |
| average_value | double | Market average |
| median_value | double | Market median |

**plate_distribution sub-object (stocks only):**

| Field | Type | Description |
|-------|------|-------------|
| plate_symbol | string | Plate symbol |
| plate_name | string | Plate name |
| plate_average | double | Plate average |
| ranking | int | Ranking within plate |
| total | int | Number of constituent stocks in plate |
| stock_items[] | array | Valuation of each stock in the plate |

**profit_growth_rate sub-object (stocks only):**

| Field | Type | Description |
|-------|------|-------------|
| financial_ttm_multiple | double | TTM financial multiple |
| market_cap_multiple | double | Market cap multiple |
| year_count | int | Number of years in statistics |
| conclusion_detailed | string | Conclusion text |
| profit_data[] | array | Annual profit data |

---

## quote_research_analyst_consensus — Analyst Consensus

Get consensus rating and target price.

**Supported markets:** HK, US, CN (SH/SZ/BJ), SG, CA, AU, JP, MY (only common stocks with analyst coverage; returns empty `data: {}` when no coverage)

### Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| symbol | string | Yes | Stock symbol, e.g. `HK.00700`, `US.AAPL` |

### Response Structure

| Field | Type | Description |
|-------|------|-------------|
| rating | int | Consensus rating: 1=Sell, 2=Underperform, 3=Hold, 4=Buy, 5=Strong Buy |
| total | int | Total number of covering analysts |
| strong_buy | float | Strong Buy proportion (%) |
| buy | float | Buy proportion (%) **only returned for HK/CN/SG/MY/AU/JP** |
| hold | float | Hold proportion (%) |
| underperform | float | Underperform proportion (%) **only returned for HK/CN/SG/MY/AU/JP** |
| sell | float | Sell proportion (%) |
| average | float | Average target price |
| highest | float | Highest target price |
| lowest | float | Lowest target price |
| num_of_target_analysts | int | Number of analysts providing target prices |
| update_time | int | Update timestamp (seconds) |
| update_time_str | string | Update date (yyyy-MM-dd) |

> **Rating tier differences:** HK/CN/SG/MY/AU/JP return 5 tiers (strong_buy/buy/hold/underperform/sell); US/CA return only 3 tiers (strong_buy/hold/sell).

### Error Codes

| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|---------------------|
| -3 | Invalid symbol format | Fix parameters and retry |
| -7 | Symbol cannot be resolved | Confirm symbol via search API |
| -2/-4/-6 | Gateway internal error | Retry |

---

## quote_research_rating_summary — Rating Details

Rating summary by institution or analyst dimension (including target price and recommendation date).

**Supported markets:** US, CA only (other markets return an empty list)

### Request Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| symbol | string | Yes | — | Stock symbol, e.g. `US.AAPL` |
| rating_dimension_type | int | No | 1 | 1=By institution, 2=By analyst |
| limit | int | No | 10 | Records per page, max 20 |
| next_key | string | No | — | Pagination cursor, leave empty for first request; `"-1"` means no more data |

### Response Structure

**Pagination:**

| Field | Type | Description |
|-------|------|-------------|
| pagination.has_more | bool | Whether there is a next page |
| pagination.next_key | string | Next page cursor |
| pagination.total | int | Total number of ratings in this dimension |

**Institution dimension (`rating_dimension_type=1`) -> `data.inst_rating_summary_list[]`:**

| Field | Type | Description |
|-------|------|-------------|
| institution_info.institution_uid | string | Institution unique ID |
| institution_info.institution_name | string | Institution name |
| institution_info.institution_en_name | string | Institution English name |
| institution_info.institution_picture_url | string | Institution logo URL |
| institution_info.update_time | int64 | Institution info update time (milliseconds) |
| rating_item_list[] | array | Rating record list for this institution |

**Analyst dimension (`rating_dimension_type=2`) -> `data.analyst_rating_summary_list[]`:**

| Field | Type | Description |
|-------|------|-------------|
| analyst_info.analyst_uid | string | Analyst unique ID |
| analyst_info.analyst_name | string | Analyst name |
| analyst_info.num_of_stars | int | Star rating (max 5) |
| analyst_info.success_rate | float | Success rate (%) |
| analyst_info.excess_return | float | Excess return (%) |
| rating_item_list[] | array | Rating record list for this analyst |

**rating_item_list[] elements (common to both dimensions):**

| Field | Type | Description |
|-------|------|-------------|
| rating | int | Rating: 1=Sell, 2=Hold, 3=Buy |
| target_price | float | Target price |
| recommendation_date | int64 | Recommendation date timestamp (milliseconds) |
| recommendation_date_str | string | Recommendation date (ISO format) |
| update_time | int64 | Update timestamp (milliseconds) |
| update_time_str | string | Update time (ISO format) |

### Error Codes

| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|---------------------|
| -3 | Invalid symbol format, rating_dimension_type not in [1,2], limit>20 | Fix parameters and retry |
| -7 | Symbol cannot be resolved | Confirm symbol via search API |
| -2/-4/-6 | Gateway internal error | Retry |

---

## quote_research_morningstar_report — Morningstar Report

Get the Morningstar comprehensive rating report (star rating, fair value, economic moat, uncertainty, capital allocation, financial health, and analyst commentary).

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| symbol | string | Yes | Stock symbol, e.g. `HK.00700`, `US.AAPL` |

**Response `data`:**

| Field | Type | Description |
|-------|------|-------------|
| rating_type | int | Rating type: 1=Quantitative rating, 2=Qualitative rating (analyst covered) |
| star_rating | int | Star rating (1~5) |
| star_update_time | int | Star rating update time (second timestamp) |
| star_update_time_str | string | Star rating update date (yyyy-MM-dd) |
| fair_value | number | Fair value (reporting currency) |
| economic_moat_label | string | Economic moat label: Wide / Narrow / None |
| economic_moat_type | int | Economic moat enum: 1=Wide, 2=Narrow, 3=None |
| uncertainty_label | string | Uncertainty label: Low / Medium / High / Very High / Extreme |
| uncertainty_type | int | Uncertainty enum: 1=Low, 2=Medium, 3=High, 4=Very High, 5=Extreme |
| capital_allocation_label | string | Capital allocation label: Exemplary / Standard / Poor / Not Rated |
| capital_allocation_type | int | Capital allocation enum: 1=Exemplary, 2=Standard, 3=Poor, 4=Not Rated |
| financial_health_label | string | Financial health label: Strong / Moderate / Weak |
| financial_health_type | int | Financial health enum: 1=Strong, 2=Moderate, 3=Weak |
| analyst_report_by_line | array<string> | Analyst byline |
| analyst_report_update_time | int | Analyst report update time (second timestamp) |
| analyst_report_update_time_str | string | Analyst report update date |
| fair_value_content | object | Fair value analysis text, containing context / update_time / update_time_str |
| economic_moat_content | object | Economic moat analysis text (same structure as above) |
| uncertainty_content | object | Uncertainty analysis text (same structure as above) |
| capital_allocation_content | object | Capital allocation analysis text (same structure as above) |
| financial_health_content | object | Financial health analysis text (same structure as above) |
| bull_say | array<object> | Bull case list, each item contains context / update_time / update_time_str |
| bear_say | array<object> | Bear case list (same structure as bull_say) |
| ai_analysis | object | Morningstar AI analysis, containing summary / analysis |
| analyst_note_title | object | Analyst note title, containing context / update_time / update_time_str |

**Coverage:** Only individual stocks have Morningstar ratings (ETFs, indices, options, futures, forex have no data). Observed covered markets: HK, US, SH, SZ, AU, CA. Uncovered stocks return ret_code=-10 (no_data).

**Error codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|---------------------|
| -3 | Missing or invalid symbol format | Fix symbol format and retry |
| -7 | Format is valid but security cannot be found | Confirm symbol exists |
| -10 | Security is valid but no Morningstar coverage | No report available, do not retry |
| -2/-4/-6 | Gateway internal error | Retry |

---

## quote_company_profile — Company Profile

Get company detail tags (overview, listing info, key metrics, etc.), returned as name/value tag pairs. ETFs/REITs are automatically routed to the fund data source.

**Supported markets:** HK, US, SH, SZ, BJ, AU, CA, JP, SG (equities, ETFs, REITs). Futures/options return unsupported; plates/OTC bonds/OTC funds return invalid_symbol.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| symbol | string | Yes | Stock symbol, e.g. `HK.00700`, `US.AAPL` |

**Response `data.items[]`:**

| Field | Type | Description |
|-------|------|-------------|
| name | string | Localized display label (e.g. "Symbol", "Company Name", "Website") |
| value | string | Tag value |
| field_type | int | Layout type: 0=Text, 1=Link, 2=Standalone title (long text block) |
| attribute_type | int | Semantic type (stable machine-readable key). Not returned for ETF/REIT path |

**attribute_type enum:**

| Value | Meaning | Value | Meaning |
|-------|---------|-------|---------|
| 1 | A-share Short Name | 28 | Email |
| 2 | A-share Code | 29 | Company Description |
| 3 | B-share Short Name | 30 | CEO |
| 4 | B-share Code | 31 | Security Type |
| 5 | H-share Short Name | 32 | ADS Conversion Ratio |
| 6 | H-share Code | 33 | Province (A-shares) |
| 7 | Symbol | 34 | State (US stocks) |
| 8 | ISIN | 35 | Main Business |
| 9 | Company Name | 36 | Business Scope |
| 10 | Listing Date | 37 | Chairman |
| 11 | Issue Price | 38 | Legal Representative |
| 12 | Issue Size | 39 | Board Secretary |
| 13 | Incorporation Date | 40 | Business License Number |
| 14 | Listing Exchange | 41 | Accounting Firm |
| 15 | Market | 42 | Securities Affairs Representative |
| 16 | Phone | 43 | Legal Counsel |
| 17 | Fiscal Year End Month | 44 | Company Category |
| 18 | Number of Employees | 45 | Auditor |
| 19 | Company Address | 46 | Audit Institution |
| 20 | Office Address | 47 | Registered Office |
| 21 | Office Address Postal Code | 48 | Headquarters & Principal Place of Business |
| 22 | Registered Address | 49 | Investor Relations Link |
| 23 | Company Website | 50 | Industry |
| 24 | City | 51 | Region |
| 25 | Country | 52 | General Manager |
| 26 | Postal Code | 53 | Share Registrar (currently AU only) |
| 27 | Fax | | |

**Error codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|---------------------|
| -3 | Missing or invalid symbol format | Check symbol spelling and format |
| -7 | Cannot be resolved to a valid security | Confirm it is an equity/ETF symbol in a supported market |
| -8 | Unsupported category (futures/options) | Query the corresponding underlying stock instead |
| -10 | Security is valid but no company data | No retry needed |

---

## quote_company_executives — Management List

Get the company's executive/director list, including name, position, gender, age, tenure start date, education, shareholdings, and annual salary.

**Supported markets:** HK, US, SH, SZ, BJ, CA, AU, JP, SG (equities, funds). Indices, forex, etc. return no_data.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| symbol | string | Yes | Stock symbol, e.g. `HK.00700`, `US.AAPL` |

**Response `data.executives[]`:**

| Field | Type | Description |
|-------|------|-------------|
| leader_name | string | Executive name (HK usually in Chinese e.g. `马化腾`; US includes honorific e.g. `Mr. Timothy D. Cook`). Must be an exact match of this field when querying `executive_background` |
| display_leader_name | string | Client display name |
| position_name | string | Position (English), e.g. `Chairman of the Board, Chief Executive Officer` |
| leader_gender | string | Gender: `male` / `female` |
| leader_age | string | Age |
| highest_education | string | Highest education (English), e.g. `Bachelor`, `Master`, `PhD` |
| begin_date | int64 | Tenure start timestamp (**milliseconds**) |
| begin_date_str | string | Tenure start date (yyyy-MM-dd) |
| issue_date | int64 | Information publish timestamp (**milliseconds**) |
| issue_date_str | string | Information publish date (yyyy-MM-dd) |
| shares | string | Number of shares held |
| annual_salary | string | Annual salary amount |
| annual_salary_currency | string | Annual salary currency (ISO 4217) |

**Error codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|---------------------|
| -3 | Missing or invalid symbol format | Check symbol format |
| -7 | Security symbol does not exist | Confirm symbol is correct |
| -10 | Security is valid but no executive data | Normal empty result, no retry needed |

---

## quote_company_executive_background — Executive Background

Get the detailed background biography of a single executive. `leader_name` must first be obtained from the `company_executives` API and matched exactly.

**Supported markets:** HK, US (other markets not yet supported). Equity common stocks only.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| symbol | string | Yes | Stock symbol, e.g. `HK.00700`, `US.AAPL` |
| leader_name | string | Yes | Executive name, must exactly match the `leader_name` field returned by `company_executives` (HK usually in Chinese e.g. `马化腾`; US includes honorific e.g. `Mr. Timothy D. Cook`). Do not pass `display_leader_name` |

**Response `data`:**

| Field | Type | Description |
|-------|------|-------------|
| brief_background | string | Executive background biography long text |

**Error codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|---------------------|
| -3 | Missing leader_name parameter | Provide the required parameter |
| -7 | Symbol cannot be resolved | Confirm symbol via search API |
| -10 | leader_name does not match or no background data | First call `company_executives` to get the exact `leader_name` |
| -4/-6 | Gateway internal error | Retry |

---

## quote_company_operational_efficiency — Operational Efficiency

Get the company's historical operational efficiency metrics (employee count, revenue/operating profit/net profit per capita and YoY growth), returned by fiscal period.

**Supported markets:** HK, US, SH, SZ, BJ, SG, JP, AU, CA (only equity securities with public financial reports). Non-corporate securities return unsupported (-8).

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| symbol | string | Yes | — | Stock symbol, e.g. `HK.00700` |
| financial_type | int | No | 7 | 7=Annual; 102=All cumulative quarterly (returns Q1/Q6/Q9/FY) |
| limit | int | No | 10 | Number of records, max 100 |
| next_key | string | No | — | Pagination cursor, leave empty for first request; pass back `pagination.next_key` for subsequent requests, stop when `has_more=false` |
| currency_code | string | No | — | ISO 4217 currency code (omit to use the latest financial report's currency) |

**Response `data`:**

| Field | Type | Description |
|-------|------|-------------|
| currency_code | string | Currency unit for monetary metrics |
| item_list[] | array | Operational efficiency data by period |
| pagination.has_more | bool | Whether there are more records |
| pagination.next_key | string | Next page cursor |

**item_list[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| fiscal_year | int | Fiscal year |
| financial_type | int | Fiscal period type |
| period_text | string | Period text, e.g. `"2025/FY"` |
| end_date | int64 | Fiscal period end timestamp (**milliseconds**) |
| end_date_str | string | End date (yyyy-MM-dd) |
| employee_num | int | Number of employees |
| employee_num_yoy | double | Employee count YoY growth (%) |
| income_per_capita | double | Revenue per capita (reporting currency) |
| income_per_capita_yoy | double | Revenue per capita YoY growth (%) |
| profit_per_capita | double | Operating profit per capita |
| profit_per_capita_yoy | double | Operating profit per capita YoY growth (%) |
| net_profit_per_capita | double | Net profit per capita |
| net_profit_per_capita_yoy | double | Net profit per capita YoY growth (%) |

**Error codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|---------------------|
| -3 | limit>100 / financial_type not in {7,102} / invalid symbol format | Fix parameters and retry |
| -7 | Symbol cannot be resolved | Confirm symbol via search API |
| -8 | Security category not in supported range | Only call for corporate equity securities |
| -10 | Security is valid but no operational efficiency data | Normal empty result, no retry needed |
