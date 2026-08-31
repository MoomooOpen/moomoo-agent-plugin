# Capital Flow Tool Reference

## quote_capital_flow — Intraday Capital Flow

Get minute-level net capital inflow time series data for the current day.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| symbol | string | Yes | — | Stock symbol, e.g. `HK.00700` |
| section | string | No | NORMAL | Trading session: NORMAL (HK auto-includes dark pool) / FULL / PREMARKET / AFTERHOURS |

**Response `data`:**

| Field | Type | Description |
|-------|------|-------------|
| flow_list[] | array | Data point list (ascending by time), empty array when no data |
| last_valid_time | int64 | Last valid data timestamp (milliseconds), null when no data |

**flow_list[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| capital_flow_item_time | int64 | Data point timestamp (milliseconds) |
| in_flow | double | Overall net inflow (positive=net inflow, negative=net outflow, local currency) |
| super_in_flow | double | Super-large order net inflow |
| big_in_flow | double | Large order net inflow |
| mid_in_flow | double | Medium order net inflow |
| sml_in_flow | double | Small order net inflow |

**Relationship:** `in_flow = super_in_flow + big_in_flow + mid_in_flow + sml_in_flow`

Intraday data only; returns empty list outside trading hours. The intraday interface does not return main force net inflow or price change percentage fields.

---

## quote_capital_flow_history — Historical Capital Flow

Get historical net capital inflow data at daily/weekly/monthly granularity.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| symbol | string | Yes | — | Stock symbol |
| period_type | string | No | DAY | Period: DAY / WEEK / MONTH |
| start | string | No | — | Start date yyyy-MM-dd |
| end | string | No | Today | End date yyyy-MM-dd |
| count | int | No | 365 | Maximum number of records, 1~1000 |

**Response `data`:**

| Field | Type | Description |
|-------|------|-------------|
| flow_list[] | array | Data point list (ascending by time) |

**Pagination:**

| Field | Type | Description |
|-------|------|-------------|
| pagination.has_more | bool | Whether there is earlier data available for further pagination |

**flow_list[] elements:**

| Field | Type | Description |
|-------|------|-------------|
| capital_flow_item_time | int64 | Timestamp (milliseconds, aligned to day/week/month per period_type) |
| in_flow | double | Overall net inflow (positive=net inflow, negative=net outflow) |
| main_in_flow | double | Main force net inflow (super-large + large orders) |
| super_in_flow | double | Super-large order net inflow |
| big_in_flow | double | Large order net inflow |
| mid_in_flow | double | Medium order net inflow |
| sml_in_flow | double | Small order net inflow |
| main_deal_ratio | double | Main force turnover ratio (e.g. 0.123 means 12.3%), omitted when not provided by backend |
| acc_main_in_flow | double | Accumulated main force net inflow, omitted when not provided by backend |

---

## quote_capital_distribution — Capital Distribution

Get the cumulative capital inflow/outflow summary snapshot for the current day by order size (large/medium/small).

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| symbol | string | Yes | Stock symbol |

**Response `data`:**

| Field | Type | Description |
|-------|------|-------------|
| capital_in_super | double | Super-large order cumulative inflow |
| capital_in_big | double | Large order cumulative inflow |
| capital_in_mid | double | Medium order cumulative inflow |
| capital_in_small | double | Small order cumulative inflow |
| capital_out_super | double | Super-large order cumulative outflow |
| capital_out_big | double | Large order cumulative outflow |
| capital_out_mid | double | Medium order cumulative outflow |
| capital_out_small | double | Small order cumulative outflow |
| update_time | int64 | Data update timestamp (milliseconds) |

**Net inflow calculation:** Net inflow per tier = `capital_in_*` - `capital_out_*`

Intraday snapshot only; all values are 0 outside trading hours.
