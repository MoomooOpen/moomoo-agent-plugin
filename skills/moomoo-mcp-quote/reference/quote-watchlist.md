# Watchlist Tool Reference

## quote_user_security_group — Watchlist Group List

Retrieve the user's watchlist group list, including system preset groups and user-defined custom groups.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| group_type | string | No | ALL | Group type filter (case-sensitive): `ALL`=all, `CUSTOM`=user-defined, `SYSTEM`=system preset |

**Response `data.group_list[]`:**

| Field | Type | Description |
|-------|------|-------------|
| group_name | string | Group name; system groups are in English (e.g. All/Favorites/HK/US/CN/Options, etc.), custom groups use the user's original naming |
| group_type | string | Type: `SYSTEM` / `CUSTOM` |

**System group names (up to 19):** All, Favorites, HK, US, CN, HK Options, US Options, Options, moomoores, Index, Bonds, Notes, Crypto, SG, JP, MY, AU, CA

**Special behavior:**
- Does not include holdings/FX/fund groups
- Hidden groups are not returned
- The `group_type` parameter is case-sensitive and must be uppercase

**Error codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|-------------------|
| 0 | Success | — |
| -3 | group_type value not in [ALL, CUSTOM, SYSTEM] | Fix parameters and retry |
| -9 | User identity missing or invalid | Verify user identity information |
| -5 | Backend watchlist service call failure | Retry |

---

## quote_user_security — Watchlist Securities

Retrieve the securities in a specified watchlist group, including code, name, lot size, security type, and derivative information.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| group_name | string | Yes | Group name (URL-encoded, max 100 characters); when duplicate names exist, the first match is used; can be obtained via `user_security_group` |

**Response `data.security_list[]`:**

| Field | Type | Description |
|-------|------|-------------|
| code | string | Stock code, e.g. `US.AAPL` |
| name | string | English name |
| sc_name | string | Simplified Chinese name |
| tc_name | string | Traditional Chinese name |
| lot_size | int | Shares per lot (options = contract shares, futures = contract multiplier) |
| stock_type | string | Security type: `STOCK`/`ETF`/`WARRANT`/`IDX`/`DRVT`/`FUTURE`/`FOREX`/`CRYPTO`/`BOND`, etc. |
| stock_child_type | int | Security sub-type numeric value |
| stock_owner | string | Underlying stock code (for derivatives); empty string for non-derivatives |
| option_type | string | Option direction: `ALL` (non-option) / `CALL` / `PUT` |
| strike_time | string | Option strike date (yyyy-MM-dd); empty string for non-options |
| strike_price | double | Option strike price; 0 for non-options |
| listing_date | string | Listing date (yyyy-MM-dd); empty string when no data |
| stock_id | int64 | Internal numeric identifier |
| main_contract | bool | Whether it is a futures main continuous contract; always false for non-futures |
| last_trade_time | string | Last trading date for options/futures (yyyy-MM-dd); empty string for other types |

**Special behavior:**
- Empty groups return an empty `security_list` array; ret_code remains 0
- Suspended/delisted securities are not returned
- `group_name` must be URL-encoded

**Error codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|-------------------|
| 0 | Success (including empty array) | — |
| -3 | group_name missing, exceeds 100 characters, or group does not exist | Verify group name via user_security_group |
| -9 | User identity missing or invalid | Verify user identity information |
| -5 | Gateway/backend internal error | Retry |

---

## quote_modify_user_security — Modify Watchlist

Add, delete, or move out watchlist securities. Only custom groups are supported; system/virtual groups are not supported.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| op | string | Yes | Operation type: `ADD`=add to specified group, `DEL`=delete from all groups, `MOVE_OUT`=move out of specified group |
| code_list | string[] | Yes | Stock code list, 1–200 items, e.g. `["HK.00700","US.AAPL"]` |
| group_name | string | Conditionally required | Custom group name (max 100 characters); required for `ADD` and `MOVE_OUT`, optional for `DEL` (ignored); when duplicate names exist, the first match is used |

**Response `data`:**

| Field | Type | Description |
|-------|------|-------------|
| result_code | int | Operation result code; 0 on success |

**Special behavior:**
- `ADD`/`MOVE_OUT` only supports custom groups; operating on system groups returns -8
- `DEL` is group-independent and removes the security from all groups
- All markets and security types can be operated on

**Error codes:**
| ret_code | Trigger Condition | Recommended Action |
|----------|-------------------|-------------------|
| 0 | Success | — |
| -3 | op missing/invalid, code_list empty or exceeds 200, group_name exceeds 100 characters, ADD/MOVE_OUT missing group_name, group does not exist, code format error | Fix parameters and retry |
| -7 | Code format is valid in code_list but security does not exist | Verify code via search API |
| -8 | Add/delete operation on system/virtual group | Use custom groups only |
| -9 | User identity missing or invalid | Verify user identity information |
| -5 | Gateway/backend internal error | Retry |
