---
name: Search a market and place an order
description: Find a prediction market across exchanges by River ID, then place and monitor an order on a subaccount.
api: openapi/river-markets-openapi-original.json
operations:
- search_markets_v1_markets_search_get
- get_orderbook_v1_orderbooks__river_id__get
- create_order_v1_orders_post
- get_order_v1_orders__order_id__get
- cancel_order_v1_orders__order_id__delete
---

# Search a market and place an order

River Markets is a unified prime brokerage for prediction markets. Every contract
across Kalshi, Polymarket, and Polymarket US is addressed by a single **River ID**.

## Auth
Every REST request is Ed25519-signed with three headers: `X-River-Key-Id`,
`X-River-Timestamp` (unix seconds, within 30s of server time), and
`X-River-Signature` (base64 signature over `METHOD\nPATH\nSORTED_QUERY\nTIMESTAMP\nSHA256(body)hex`).
The official `rivermarkets` Python SDK signs transparently. Base URL: `https://api.rivermarkets.com/v1`.

## Steps
1. **Find the market.** Call `search_markets_v1_markets_search_get` with `q` (and optional
   `exchange_name`, `status=active`, `expiration_date_start`). Read the `river_id` from a result.
2. **Check the book.** Call `get_orderbook_v1_orderbooks__river_id__get` with that `river_id` to
   see `best_bid_price` / `best_ask_price` before pricing your order.
3. **Place the order.** Call `create_order_v1_orders_post` with `river_id`, `subaccount_id`,
   `buy_flag`, `qty`, `price`, `order_type`, and `time_in_force`. The response returns a
   `river_order_id`.
4. **Monitor.** Poll `get_order_v1_orders__order_id__get` with the `river_order_id` to read status
   and fills, or subscribe to the `orders`/`fills` WebSocket streams for push updates.
5. **Cancel if needed.** Call `cancel_order_v1_orders__order_id__delete` with the `river_order_id`.

## Rules
- There is **no idempotency key** — do not blind-retry `create_order`; on a timeout, reconcile with
  `list_orders_v1_orders_get` before retrying to avoid duplicate orders.
- Validation failures return HTTP 422 with a `HTTPValidationError` `detail[]` of `{loc,msg,type}`.
- Standard rate limit is 10 req/s.
- Always scope writes to a `subaccount_id`; row-level security prevents cross-account access.
