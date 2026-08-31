# News, Calendar & Other Tool Reference

## quote_news_search — News Search

Search articles / announcements / research reports.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| symbol | string | Yes | — | Search keyword (required) |
| news_type | int | No | — | 1=Article, 2=Announcement, 3=Research Report; omit to return all |
| sort_type | int | No | — | 0=Default, 1=By views, 2=By time |
| size | int | No | 10 | Number of results, 1~50 |
| lang | string | No | — | zh-CN/zh-HK/en/ja |

**Response `data.news_list[]`:**

| Field | Type | Description |
|-------|------|-------------|
| news_id | string | News ID |
| news_type | string | Type: POST (Article) / NOTICE (Announcement) / REPORT (Research Report) |
| title | string | Title |
| publish_time | int64 | Publish time (second timestamp) |
| url | string | Original article link |
| img_url | string | Image link (optional) |

---

## quote_community_search — Community Search

Search community posts / topics / live streams.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| symbol | string | Yes | — | Search keyword |
| community_type | int | No | — | 1=Post, 2=Topic, 3=Live |
| sort_type | int | No | — | 0=Default, 1=By popularity, 2=By time |
| size | int | No | 10 | Number of results, 1~50 |
| lang | string | No | — | zh-CN/zh-HK/en/ja |

**Response `data.community_list[]`:**

| Field | Type | Description |
|-------|------|-------------|
| id | string | Content ID |
| community_type | string | Type: FEED / TOPIC / LIVE |
| title | string | Title |
| publish_time | int64 | Publish time (second timestamp) |
| url | string | Link |
| img_url | string | Image link (optional) |

---

## quote_stock_feed — Stock Feed

Get community posts related to a specific stock.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| symbol | string | Yes | — | Stock name (e.g. "Tencent", "AAPL") |
| size | int | No | 10 | Number of results, 1~50 |

**Response `data.feed_list[]`:**

| Field | Type | Description |
|-------|------|-------------|
| id | string | Post ID |
| title | string | Title |
| publish_time | int64 | Publish time (second timestamp) |
| desc | string | Summary description |

---

## quote_economic_calendar_hot — Hot Economic Data

Get the hot/recommended economic data list for a specified date, sorted by importance. Returns major economic indicators for the day (such as CPI, GDP, Non-Farm Payrolls, central bank rate decisions, etc.), including previous value, forecast, actual value, and star-rated importance.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| date | string | No | Today | Query date, format `yyyyMMdd`, e.g. `20260618` |
| limit | int | No | 10 | Records per page, range 1~20 |
| next_key | string | No | — | Pagination cursor, leave empty for first request; pass back `pagination.next_key` for subsequent requests; value `"-1"` means no more data |
| timezone | string | No | Asia/Shanghai | Timezone, e.g. `America/New_York` |

**Response:**

Top-level `data` is an array (not an object), `pagination` is at the same level as `data`.

| Field | Type | Description |
|-------|------|-------------|
| data[] | array | Economic data event list |
| pagination.has_more | bool | Whether there are more records |
| pagination.next_key | string | Next page cursor (`"-1"` means no more) |

**data[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| event_text | string | Event content, e.g. `"US Fed Interest Rate Decision (Upper Bound) as of June 17"` |
| previous | string | Previous value, e.g. `"3.75"` |
| predictive | string | Forecast/consensus value, e.g. `"3.75"` |
| announce | string | Announced/actual value; empty string when not yet published |
| star | int | Importance star rating, range 1~5 (5=most important, e.g. central bank rate decisions) |
| event_time | int64 | Event publish time (second timestamp) |
| country | string | Country/region, e.g. `"United States"`, `"Japan"` |
| currency | string | Related currency, may be null |
| unit | string | Data unit, e.g. `"%"`, `"10K persons"`, may be null |
| unique_id | string | Unique identifier, e.g. `"calendar_economic:103384299"`, may be null |
| detail_url | string | Detail page redirect link |

**Error codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|---------------------|
| 0 | Success (including empty list) | Check data array length to determine if there is data |
| -3 | limit out of 1~20 range | Fix parameters and retry |
| -4 | Backend business error | Retry |

---

## quote_economic_calendar_search — Economic Calendar Search

Search economic calendar data by keyword, supporting four types: economic data (CPI/GDP/Non-Farm Payrolls, etc.), market closures, economic events (central bank speeches/meetings), and rights & interests (dividends/stock splits). Returns a paginated, time-sortable event list.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| keyword | string | Yes | — | Search keyword, e.g. `CPI`, `Fed`, `GDP` |
| search_type | int | Yes | — | Search type: 1=Economic Data, 2=Market Closure, 3=Economic Event, 4=Rights & Interests (dividends/stock splits) |
| limit | int | No | 20 | Records per page, range 1~30 |
| next_key | string | No | — | Pagination cursor, leave empty for first request; pass back `pagination.next_key` for subsequent requests |
| time_order_type | int | No | 0 | Sort order: 0=Ascending (ASC), 1=Descending (DESC); **only effective for search_type=2 and 3** |

**Response:**

Top-level `data` is an array (not an object), `pagination` is at the same level as `data`.

| Field | Type | Description |
|-------|------|-------------|
| data[] | array | Matched event list |
| pagination.has_more | bool | Whether there are more records |
| pagination.next_key | string | Next page cursor |

**data[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| event_text | string | Event content; matched keywords are highlighted with `<em>` tags, e.g. `Canada May <em>CPI</em> MoM` |
| previous | string | Previous value; null for search_type=3 |
| predictive | string | Forecast/consensus value; null for search_type=3 |
| announce | string | Announced/actual value; null when not yet published; null for search_type=3 |
| star | int | Importance star rating, range 1~5 |
| event_time | int64 | Event publish time (second timestamp) |
| country | string | Country/region |
| currency | string | Related currency, may be null |
| unit | string | Data unit, may be null |
| unique_id | string | Unique identifier, e.g. `"calendar_economic:155416319"`, `"calendar_event:83109"` |
| detail_url | string | Detail link; may be null for search_type=3 |

**Special notes by search_type:**
- **search_type=1 (Economic Data):** `previous`/`predictive` usually have values; `announce` is null before official publication
- **search_type=3 (Economic Event):** `previous`, `predictive`, `announce`, `detail_url` are all null
- **Keyword highlighting:** Each matching character/word segment in `event_text` is wrapped with `<em>` tags (CJK characters use character-level highlighting)

**Error codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|---------------------|
| 0 | Success (including empty list) | Check data array length to determine if there are results |
| -3 | Missing keyword or search_type; search_type not in 1~4; limit out of range | Fix parameters and retry |
| -4 | Backend business error | Retry |

---

## quote_ipo_list_hk / _us / _cn / _sg / _my — IPO Lists

Five separate market-specific interfaces returning recent subscribable and upcoming IPOs (excluding historical IPOs). Data comes from the current user's primary broker; if the user has no broker for the corresponding market, an empty list may be returned.

### Request Parameters

**HK / US / Singapore / Malaysia:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| request_type | int | No | 11 | 9=Subscribing, 10=Awaiting Listing, 11=Upcoming (includes Subscribing + Awaiting Listing) |

**A-shares:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| request_type | int | No | 4 | 1=IPO Notice, 2=Applying, 3=Winning Results Published, 4=Awaiting Listing, 5=All |

### Response `data.list[]`

#### HK

| Field | Type | Description |
|-------|------|-------------|
| code | string | Symbol with market prefix, e.g. `HK.12233` |
| name | string | English name |
| sc_name | string | Simplified Chinese name |
| tc_name | string | Traditional Chinese name |
| list_time | string | Listing date yyyy-MM-dd; `"--"` when not confirmed |
| list_timestamp | int64 | Listing timestamp (milliseconds); 0 when not confirmed |
| ipo_price_min | float | Offer price lower bound (HKD) |
| ipo_price_max | float | Offer price upper bound (HKD) |
| list_price | float | Final offer price (HKD) |
| lot_size | int | Board lot size |
| entrance_price | float | Entry fee (HKD) |
| apply_start_timestamp | int64 | Subscription start timestamp (milliseconds) |
| apply_end_time | string | Subscription deadline yyyy-MM-dd |
| apply_end_timestamp | int64 | Subscription deadline timestamp (milliseconds) |
| apply_countdown_secs | int | Subscription deadline countdown (seconds) |
| is_apply_started | bool | Whether subscription has started |
| is_support_apply | bool | Whether subscription is supported |
| is_subscribe_status | bool | Whether currently in subscribable status |
| apply_multiple | string | Subscription multiple (oversubscription ratio) |
| lucky_ratio | string | One-lot winning rate |
| win_ratio_msg | string | Winning probability description |
| lucky_time | string | Results announcement date yyyy-MM-dd |
| lucky_timestamp | int64 | Results announcement timestamp (milliseconds) |
| margin_fee_ratio | float | Margin subscription interest rate (%) |
| margin_fee_ratio_min | float | Minimum margin interest rate (%); 0=not provided |
| margin_fee_ratio_max | float | Maximum margin interest rate (%); 0=not provided |
| margin_lever_ratio | float | Margin leverage multiple |
| real_margin_lever_ratio | float | Actual available margin leverage; 0=not provided |
| margin_rate | float | Margin ratio (4 decimal places) |
| dark_trade_date | string | Grey market trading date; empty string=none |
| dark_trade_timestamp | int64 | Grey market date timestamp (milliseconds); 0=none |
| dark_trade_period | string | Grey market trading session, e.g. `16:15~18:30` |
| dark_trade_start_timestamp | int64 | Grey market start timestamp (milliseconds); 0=none |
| dark_trade_end_timestamp | int64 | Grey market end timestamp (milliseconds); 0=none |
| is_support_dark_trade | bool | Whether grey market trading is supported |
| is_support_intl_placing | bool | Whether international placing is supported |
| show_intl_placing_info | bool | Whether to show international placing info to the current user |
| intl_placing_apply_start_timestamp | int64 | International placing subscription start (milliseconds); 0=none |
| intl_placing_apply_end_timestamp | int64 | International placing subscription deadline (milliseconds); 0=none |
| intl_placing_apply_limit | float | International placing minimum subscription amount (HKD) |
| intl_placing_apply_limit_str | string | International placing minimum amount (with thousand separators) |
| placing_countdown_secs | int | International placing deadline countdown (seconds); 0=none |
| offer_type | string | Offer type (see enum) |
| security_type | string | Security type (see enum) |
| apply_status | string | Public subscription status (see enum) |
| placing_apply_status | string | International placing subscription status (same enum as apply_status) |

#### US

| Field | Type | Description |
|-------|------|-------------|
| code | string | Symbol with market prefix, e.g. `US.AAPL` |
| name | string | English name |
| sc_name | string | Simplified Chinese name |
| tc_name | string | Traditional Chinese name |
| list_time | string | Expected listing date yyyy-MM-dd |
| list_timestamp | int64 | Listing timestamp (milliseconds) |
| ipo_price_min | float | Offer price lower bound (USD) |
| ipo_price_max | float | Offer price upper bound (USD) |
| issue_size | int | Number of shares offered |
| apply_limit | float | Minimum subscription amount (USD) |
| apply_start_timestamp | int64 | Subscription start timestamp (milliseconds) |
| apply_end_timestamp | int64 | Subscription deadline timestamp (milliseconds) |
| lucky_timestamp | int64 | Expected results announcement timestamp (milliseconds) |
| stock_status | string | IPO status: `PENDING` / `APPLYING` / `CLOSED` |
| is_user_applied | bool | Whether the current user has subscribed |
| offer_type | string | Offer type (same as HK enum) |

#### A-shares (CN)

| Field | Type | Description |
|-------|------|-------------|
| code | string | Symbol with market prefix, e.g. `SZ.300728` |
| name | string | English name |
| sc_name | string | Simplified Chinese name |
| tc_name | string | Traditional Chinese name |
| list_time | string | Listing date; `"--"` if not yet listed |
| list_timestamp | int64 | Listing timestamp (milliseconds); 0 if not yet listed |
| apply_code | string | Subscription code |
| apply_upper_limit | int | Subscription upper limit (shares) |
| ipo_price | float | Offer price (CNY) |
| winning_ratio | float | Winning rate (%) |
| issue_pe_rate | float | Issue P/E ratio |
| apply_timestamp | int64 | Subscription timestamp (milliseconds) |
| winning_time | string | Winning results announcement date yyyy-MM-dd |
| winning_timestamp | int64 | Winning results announcement timestamp (milliseconds) |
| is_has_won | bool | Whether winning results have been announced |

#### Singapore (SG)

| Field | Type | Description |
|-------|------|-------------|
| code | string | Symbol with market prefix, e.g. `SG.1V2` |
| name | string | English name |
| sc_name | string | Simplified Chinese name |
| tc_name | string | Traditional Chinese name |
| list_time | string | Listing date yyyy-MM-dd |
| list_timestamp | int64 | Listing timestamp (milliseconds) |
| ipo_price_min | float | Offer price lower bound (SGD) |
| ipo_price_max | float | Offer price upper bound (SGD) |
| offer_price_display | string | Offer price display text |
| issue_size_display | string | Issue size display text |
| apply_limit | float | Minimum subscription amount (SGD) |
| apply_limit_display | string | Minimum subscription amount display text |
| market_cap_display | string | Market cap display text |
| fund_amount_display | string | Fund raised amount display text |
| industry_display | string | Industry information |
| managers_display | string | Underwriter information |
| ipo_book_link | string | Prospectus link |
| security_type | string | Security type (same as HK enum) |
| apply_start_timestamp | int64 | Subscription start timestamp (milliseconds) |
| apply_end_timestamp | int64 | Subscription deadline timestamp (milliseconds) |
| is_subscribe_status | bool | Whether currently in subscribable status |
| is_user_applied | bool | Whether the current user has subscribed |

#### Malaysia (MY)

| Field | Type | Description |
|-------|------|-------------|
| code | string | Symbol with market prefix, e.g. `BMS.MYIPO007` |
| name | string | English name |
| sc_name | string | Simplified Chinese name |
| tc_name | string | Traditional Chinese name |
| list_time | string | Listing date yyyy-MM-dd |
| list_timestamp | int64 | Listing timestamp (milliseconds) |
| ipo_price | float | Offer price (MYR) |
| issue_size | int | Number of shares offered |
| fund_amount | float | Total funds raised (MYR) |
| market_cap | int | Market capitalization (MYR) |
| currency | string | Currency code, e.g. `MYR` |
| manager | string | Underwriter information |
| industry_plate | string | Industry plate name |
| business | string | Company description |
| ipo_book_link | string | Prospectus link |
| security_type | string | Security type (same as HK enum) |
| apply_start_timestamp | int64 | Subscription start timestamp (milliseconds) |
| apply_end_timestamp | int64 | Subscription deadline timestamp (milliseconds) |
| draw_timestamp | int64 | Retail ballot draw timestamp (milliseconds) |
| lucky_timestamp | int64 | Results announcement timestamp (milliseconds) |
| apply_limit | float | Minimum subscription amount (MYR) |
| is_subscribe_status | bool | Whether currently in subscribable status |
| is_user_applied | bool | Whether the current user has subscribed |
| support_leverage | bool | Whether leveraged subscription is supported |

### Enum Values

**offer_type (Offer Type):**
| Value | Meaning |
|-------|---------|
| UNSUPPORTED | Not supported |
| PUBLIC_OFFER_ONLY | Public offer only |
| INTERNATIONAL_PLACING_ONLY | International placing only |
| PUBLIC_AND_INTER_PLACING | Public offer + International placing |

**security_type (Security Type):**
| Value | Meaning |
|-------|---------|
| NORMAL | Common stock |
| SPAC | Special Purpose Acquisition Company |
| HK_IBOND | Hong Kong iBond |
| HK_CLIMATE_BOND | Hong Kong Green Bond |
| HK_SILVER_BOND | Hong Kong Silver Bond |
| HK_ETF | Hong Kong ETF |
| SG_ETF | Singapore ETF |

**apply_status (Subscription Status):**
| Value | Meaning |
|-------|---------|
| NOT_APPLIED | Not applied |
| APPLY_PENDING | Application pending |
| APPLY_PROCESSING | Application processing |
| APPLY_WON | Won allocation |
| APPLY_LOST | Did not win allocation |

**stock_status (US IPO Status):**
| Value | Meaning |
|-------|---------|
| PENDING | Awaiting subscription opening |
| APPLYING | Subscription in progress |
| CLOSED | Subscription closed |

### Error Codes

| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|---------------------|
| 0 | Success | — |
| -3 | Invalid request_type value | Fix per market-specific allowed values and retry |
| -5 | Backend call failed/timeout | Retry later |

---
