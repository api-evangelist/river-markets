---
name: Set up a subaccount and track positions
description: Create an isolated trading subaccount, then read its balances, positions, and fills.
api: openapi/river-markets-openapi-original.json
operations:
- create_subaccount_v1_subaccounts_post
- list_subaccounts_v1_subaccounts_get
- get_exchange_balance_v1_balance_exchange_get
- get_positions_v1_positions_get
- get_fills_v1_fills_get
---

# Set up a subaccount and track positions

Subaccounts are isolated trading containers — each holds its own exchange credentials,
orders, positions, and balances, gated by row-level security.

## Auth
Ed25519 request signing (see the search-and-trade skill). Base URL `https://api.rivermarkets.com/v1`.

## Steps
1. **Create the subaccount.** Call `create_subaccount_v1_subaccounts_post`. Add Kalshi/Polymarket
   credentials to it out-of-band (they are encrypted at rest and never returned by any endpoint).
2. **List subaccounts.** Call `list_subaccounts_v1_subaccounts_get` to confirm and get the
   `subaccount_id`.
3. **Read balances.** Call `get_exchange_balance_v1_balance_exchange_get` for live per-exchange
   balances on the subaccount.
4. **Read positions.** Call `get_positions_v1_positions_get` (scoped to the subaccount) for net
   positions by `river_id`.
5. **Read fills.** Call `get_fills_v1_fills_get` for execution history.

## Rules
- Every read/write is scoped to a `subaccount_id`; RLS prevents seeing another user's data.
- Deleting a subaccount (`delete_subaccount_v1_subaccounts__subaccount_id__delete`) cascades and
  deletes its exchange credentials — no soft delete.
- Validation errors return HTTP 422 (`HTTPValidationError`). Rate limit 10 req/s.
