---
name: Create and publish a WeTravel trip
description: Build a draft trip in the WeTravel Trip Builder, enrich it with a package and details, then publish it.
api: openapi/wetravel-partner-openapi.json
operations: [issueAccessToken, createTrip, getTripById, createPackage, updateTrip, publishTrip]
---

# Create and publish a WeTravel trip

Use the WeTravel Partner API (Trip Builder) to programmatically create a bookable trip page.

## Auth (required first)
1. Your Partner API key (from the WeTravel profile page) is a **refresh token** — it cannot call the API directly.
2. Call `issueAccessToken` (`POST /v2/auth/tokens/access`) with the API key in the `Authorization` header to get a JWT **access token** (valid 1 hour).
3. Send the access token as `Authorization: Bearer <access_token>` on every subsequent call (or send the key as `X-Api-Key`).

## Steps
1. `createTrip` (`POST /draft_trips`) — creates a **draft** trip object. Capture the returned `trip_uuid`.
2. `createPackage` (`POST /packages`) — add at least one priced package to the trip. All currency amounts are in **minor units**.
3. Optionally enrich: add-ons (`createOption`), discounts (`createDiscount`), images (`createImage`), itinerary (`createItinerary`), included/not-included items, participant questions (`createSurvey`).
4. `updateTrip` (`PATCH /draft_trips/{trip_uuid}`) — set trip-level details.
5. `getTripById` (`GET /draft_trips/{trip_uuid}`) — verify the assembled trip.
6. `publishTrip` (`POST /draft_trips/{trip_uuid}/publish`) — make the trip live and bookable. A `trip.published` webhook fires.

## Rules
- Access tokens expire after 1 hour — re-issue on `401 Unauthorized`.
- Respect the global rate limit: 200 requests/minute; on `429` honor the `Retry-After` header.
- `publishTrip` can return `500` — retry with backoff.
- On v3 (current release), drafts are removed and trips publish immediately on create/update; this skill targets the v2 draft workflow captured in the spec.
