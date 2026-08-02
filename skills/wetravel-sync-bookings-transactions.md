---
name: Sync WeTravel bookings and transactions
description: Pull trips, orders and transactions from WeTravel to sync bookings and payments into an external system.
api: openapi/wetravel-partner-openapi.json
operations: [issueAccessToken, getTrips, getOrders, getOrdersByBuyer, getOrderById, getTransactions, getTransaction, getLeads]
---

# Sync WeTravel bookings and transactions

Use the Bookings, Transactions and Leads APIs to mirror WeTravel activity into a CRM/accounting system.

## Auth
1. `issueAccessToken` (`POST /v2/auth/tokens/access`) using your Partner API key → JWT access token (1 hour).
2. Send `Authorization: Bearer <access_token>` or `X-Api-Key`.

## Steps
1. `getTrips` (`GET /draft_trips`) — enumerate trips (paginate with `page` / `per_page`).
2. For each trip, `getOrders` (`GET /bookings/trips/{trip_uuid}/orders`) — list orders/bookings; use `getOrdersByBuyer` or `getOrderById` for detail.
3. `getTransactions` (`GET /transactions`) / `getTransaction` — pull the payment/refund/payout ledger.
4. `getLeads` (`GET /leads`) — pull brochure downloads and inquiries.
5. Keep in sync in real time by subscribing to `booking.*`, `payment.*`, `transaction.*`, `trip.published`, and `lead.*` webhooks (see `asyncapi/wetravel-webhooks.yml`).

## Rules
- Paginate list endpoints; v3 default `per_page` is 25 (v2 was 1000).
- Currency amounts are in **minor units**.
- Global rate limit 200/min; honor `Retry-After` on `429`; re-issue the access token on `401`.
