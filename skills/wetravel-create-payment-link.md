---
name: Create and publish a WeTravel payment link
description: Create a hosted WeTravel payment link, publish it, and retrieve it for collecting a payment.
api: openapi/wetravel-partner-openapi.json
operations: [issueAccessToken, createPaymentLink, getPaymentLinkById, updatePaymentLink, publishPaymentLink, getPaymentLinks]
---

# Create and publish a WeTravel payment link

Use the WeTravel Payments API to spin up a hosted payment link outside of a full trip page.

## Auth
1. Exchange your Partner API key for an access token via `issueAccessToken` (`POST /v2/auth/tokens/access`).
2. Send `Authorization: Bearer <access_token>` (token valid 1 hour) or `X-Api-Key`.

## Steps
1. `createPaymentLink` (`POST /payment_links`) — create the link. Amounts are in **minor units**; the Payment Links API supports full-cents precision. Capture the returned `uuid`.
2. `updatePaymentLink` (`PUT /payment_links/{uuid}`, v2 / `PATCH` on v3) — adjust pricing or details if needed.
3. `getPaymentLinkById` (`GET /payment_links/{uuid}`) — verify.
4. `publishPaymentLink` (`POST /payment_links/{uuid}/publish`) — make the link usable (on v3, links are usable immediately and publish is removed).
5. `getPaymentLinks` (`GET /payment_links`) — list/reconcile your links.

## Rules
- Re-issue the access token on `401`; validate the body on `400`; the `uuid` must exist for `404`.
- Global rate limit 200/min; honor `Retry-After` on `429`.
- Subscribe to `payment.created` / `payment.updated` / `transaction.created` webhooks to track collection.
