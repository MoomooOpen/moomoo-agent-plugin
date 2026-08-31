# Real Trading — Place/Modify/Cancel/Confirm Orders

## Place Order

### trading_order_place — Real Order Placement

Submit a securities trading order, supporting various order types and trading sessions.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| acc_id | string | Yes | Account ID (path parameter) |
| code | string | Yes | Code format `Exchange.Code` (e.g., `US.AAPL`, `SEHK.00700`, `US.AAPL250926C235000`); not required for multi-leg orders (order_class=MLEG) |
| side | string | Yes | Side: `BUY`/`SELL`/`SELL_SHORT`/`BUY_BACK` |
| order_type | string | Yes | Order type: `LIMIT`/`MARKET`/`AUCTION`/`AUCTION_LIMIT`/`STOP`/`STOP_LIMIT`/`MARKET_IF_TOUCHED`/`LIMIT_IF_TOUCHED` |
| qty | string | Yes | Quantity (unit is "contracts" for options/futures) |
| price | string | No | Price for limit orders; precision up to 4 decimal places for securities accounts, 9 for futures, excess is rounded |
| aux_price | string | No | Trigger price; required for STOP/STOP_LIMIT/MARKET_IF_TOUCHED/LIMIT_IF_TOUCHED; 3 decimal places for securities, 9 for futures |
| time_in_force | string | Yes | Time in force: `DAY` (good for day) / `GTC` (good till cancelled) |
| session | string | No | Trading session (US stocks only): `RTH`/`RTH+Pre/Post-Mkt`/`OVERNIGHT`/`ALL_DAY`; market orders only support RTH |
| lot_type | string | No | Round lot/odd lot (HK stocks only): `ROUND`/`ODD` |
| remark | string | No | Remark, max 64 bytes in UTF-8 encoding |
| order_class | string | No | Pass `MLEG` for multi-leg orders |
| multi_leg_info | object | No | Multi-leg order info (required when order_class=MLEG) |

**Exchange codes:** US=US stocks, SEHK=HKEX, SGX=Singapore, SSE=SSE/Shanghai Stock Connect, SZSE=SZSE/Shenzhen Stock Connect, JP=Japan, KR=South Korea, CA=Canada, CME/CBOT/NYMEX/COMEX=US futures, CBOE=US options, HKFE=HK futures

**Success response:**

| Field | Type | Description |
|-------|------|-------------|
| s | string | `"ok"` |
| d.order_id | string | New order ID |

**Error response:**

| Field | Type | Description |
|-------|------|-------------|
| s | string | `"error"` |
| errcode | int | Error code |
| errmsg | string | Error message |
| jump_url | string | Redirect URL |
| need_order_confirm | bool | Whether secondary confirmation is required (call trading_order_confirm when true) |
| confirm_id | string | Confirmation ID (used when secondary confirmation is needed) |

**Special behavior:**
- When `need_order_confirm=true`, the order has not taken effect; use `confirm_id` to call the confirmation API to complete secondary confirmation
- Market orders only support the `RTH` session

**Error codes:**
| errcode | Trigger condition | Recommended action |
|---------|-------------------|-------------------|
| -1200 | General error | Handle based on errmsg content |

---

## Modify Order

### trading_order_replace — Real Order Modification

Modify the price, quantity, or conditional parameters of a submitted order. A-share order modification is not supported.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| acc_id | string | Yes | Account ID (path parameter) |
| order_id | string | Yes | Order ID (path parameter) |
| exchange | string | Yes | Exchange identifier: US/SEHK/SGX/SSE/SZSE/JP/KR/CA/CME/CBOT/NYMEX/COMEX/CBOE/HKFE |
| qty | string | Yes | New quantity (unit is "contracts" for options/futures) |
| price | string | Yes | New price; up to 9 decimal places for futures, 4 for other securities, excess is truncated |
| aux_price | string | No | New trigger price; required for STOP/STOP_LIMIT/MARKET_IF_TOUCHED/LIMIT_IF_TOUCHED; 3 decimal places for securities, 9 for futures |

**Success response:**

| Field | Type | Description |
|-------|------|-------------|
| s | string | `"ok"` |

**Error response:**

| Field | Type | Description |
|-------|------|-------------|
| s | string | `"error"` |
| errcode | int | Error code (e.g., `-1200`) |
| errmsg | string | Error message |
| jump_url | string | Redirect URL |
| need_order_confirm | bool | Whether secondary confirmation is required (call trading_order_confirm when true) |
| confirm_id | string | Confirmation ID (used when secondary confirmation is needed) |

**Special behavior:**
- A-share order modification is not supported
- When `need_order_confirm=true`, the modification has not taken effect; use `confirm_id` to call `trading_order_confirm` to complete secondary confirmation
- `price` is truncated (not rounded) when exceeding the precision limit

**Error codes:**
| errcode | Trigger condition | Recommended action |
|---------|-------------------|-------------------|
| -1200 | General error | Handle based on errmsg content |

---

## Cancel Order

### trading_order_cancel — Real Order Cancellation

Cancel the specified unfilled or partially filled orders.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| acc_id | string | Yes | Account ID (path parameter) |
| order_id | string | Yes | Order ID (path parameter) |
| exchange | string | Yes | Exchange identifier (query parameter): US/SEHK/SGX/SSE/SZSE/JP/KR/CA/CME/CBOT/NYMEX/COMEX/CBOE/HKFE |

**Success response:**

| Field | Type | Description |
|-------|------|-------------|
| s | string | `"ok"` |

**Error response:**

| Field | Type | Description |
|-------|------|-------------|
| s | string | `"error"` |
| errcode | int | Error code (e.g., `-1200`) |
| errmsg | string | Error message |

**Special behavior:**
- The request has no body; parameters are passed via path and query string
- Only unfilled or partially filled orders can be cancelled

**Error codes:**
| errcode | Trigger condition | Recommended action |
|---------|-------------------|-------------------|
| -1200 | General error | Handle based on errmsg content |

---

## Secondary Confirmation

### trading_order_confirm — Order Risk Control Confirmation

Must be called when order placement (trading_order_place) or order modification (trading_order_replace) returns `need_order_confirm=true`; the order only takes effect after secondary confirmation is completed.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| acc_id | string | Yes | Account ID (path parameter) |
| confirm_id | string | Yes | Confirmation ID returned by the place/modify order API |
| exchange | string | No | Exchange identifier |
| confirm_operate | int | No | Confirmation operation type |
| batch_confirm_type | int[] | No | Batch confirmation type list |
| operate_req | string | No | Operation request data (base64 encoded) |
| security_type | int | No | Security type |

**Success response:**

| Field | Type | Description |
|-------|------|-------------|
| s | string | `"ok"` |
| d.order_id | string | Order ID after confirmation |

**Error response:**

| Field | Type | Description |
|-------|------|-------------|
| s | string | `"error"` |
| errcode | int | Error code (e.g., `-1200`) |
| errmsg | string | Error message |

**Special behavior:**
- `confirm_id` originates only from the immediately preceding place/modify order response and is valid for one-time use only
- The order does not take effect until the confirmation API is called

**Error codes:**
| errcode | Trigger condition | Recommended action |
|---------|-------------------|-------------------|
| -1200 | General error | Handle based on errmsg content |
