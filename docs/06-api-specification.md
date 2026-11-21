# API Specification — AliAuction.ca

**API style:** REST over HTTPS with JSON, implemented in Laravel 13
**Contract:** OpenAPI 3.1 document generated from the Laravel codebase and checked in CI
**Consumers:** React.js web application (TypeScript), Flutter mobile application (Dart), provider webhooks, internal jobs

## 1. API conventions

- Base path: `/api/v1` for all client-facing endpoints.
- Provider webhooks live under `/api/webhooks/...` and are excluded from user authentication. They are authenticated by signature.
- HTTPS only. HSTS enabled.
- Authentication:
  - React SPA on the same domain: Laravel Sanctum cookie-based authentication with CSRF protection.
  - Flutter and any third-party client: Sanctum personal access tokens, with rotation and revocation.
- Resource naming: plural nouns, kebab-case paths, nested routes only one level deep (for example `/auctions/{auction}/bids`).
- Responses use Laravel API Resources, so the shape of each resource is defined in one place.
- Pagination: cursor pagination (`cursorPaginate`) for listings, bids, orders and admin queues. Responses include `links.next` and `meta.next_cursor`.
- Filtering and sorting use explicit query parameters, for example `filter[category]=`, `sort=-ends_at`. Only whitelisted fields are accepted.
- Money is always returned as an object: `{ "amount_minor": 12500, "currency": "CAD" }`.
- Timestamps are ISO 8601 in UTC, for example `2026-11-21T18:00:00Z`.
- Idempotency: `POST` requests that create bids, checkouts, refunds, subscriptions or payments require an `Idempotency-Key` header. Replays with the same key return the original response. A reused key with a different body returns `409 IDEMPOTENCY_CONFLICT`.
- Use server time for all auction rules.
- Never expose proxy maximum bids, private addresses, verification documents or provider secrets.

## 2. Error model

Validation errors use HTTP 422 with Laravel's standard `errors` object. Other errors use RFC 9457 problem details:

```json
{
  "type": "https://api.aliauction.ca/errors/bid-too-low",
  "title": "Bid too low",
  "status": 422,
  "code": "BID_TOO_LOW",
  "detail": "The bid must be at least CAD 25.00.",
  "correlation_id": "7c1f2a9e-4d3b-4f6a-9b1e-2a8f0c5d7e11"
}
```

Stable error codes:
`UNAUTHENTICATED`, `FORBIDDEN`, `VALIDATION_ERROR`, `NOT_FOUND`, `LISTING_NOT_PUBLISHED`, `AUCTION_NOT_OPEN`, `BID_TOO_LOW`, `SELLER_CANNOT_BID`, `BIDDER_NOT_ELIGIBLE`, `IDEMPOTENCY_CONFLICT`, `PAYMENT_REQUIRES_ACTION`, `PAYMENT_FAILED`, `RATE_LIMITED`, `NOT_IMPLEMENTED_PENDING_DECISION`, `INTERNAL_ERROR`.

Stack traces and provider secrets are never returned to clients. Every error includes a `correlation_id` that matches the server log entry.

## 3. Endpoint inventory

Endpoints are grouped by domain. Names are proposals for the first OpenAPI draft.

### Identity and account

| Method | Path | Purpose | Auth |
|---|---|---|---|
| POST | `/api/v1/auth/register` | Create an account (unverified) | Guest |
| POST | `/api/v1/auth/login` | Create a session or token | Guest |
| POST | `/api/v1/auth/logout` | End the session or revoke the token | User |
| POST | `/api/v1/auth/email/verify` | Verify email with a single-use token | User |
| POST | `/api/v1/auth/password/forgot` | Request a reset link | Guest |
| POST | `/api/v1/auth/password/reset` | Set a new password with a single-use token | Guest |
| POST | `/api/v1/auth/mfa/challenge` | Complete two-factor sign-in | Guest |
| GET | `/api/v1/me` | Current user and roles | User |
| PATCH | `/api/v1/me` | Update profile | User |

### Seller

| Method | Path | Purpose | Auth |
|---|---|---|---|
| POST | `/api/v1/seller-profile` | Create seller profile | User |
| GET | `/api/v1/seller-profile` | Own seller profile and verification status | Seller |
| POST | `/api/v1/seller-profile/verification` | Submit verification documents | Seller |
| GET | `/api/v1/seller/dashboard` | Sales and listing metrics | Seller |
| GET | `/api/v1/seller/orders` | Own orders | Seller |
| GET | `/api/v1/seller/payouts` | Own payout history | Seller |
| POST | `/api/v1/seller/subscription` | Start or change a plan | Seller |

### Catalogue and listings

| Method | Path | Purpose | Auth |
|---|---|---|---|
| GET | `/api/v1/categories` | Category tree | Public |
| GET | `/api/v1/listings` | Browse and search published listings | Public |
| GET | `/api/v1/listings/{slug}` | Public listing detail | Public |
| POST | `/api/v1/listings` | Create draft | Seller |
| GET | `/api/v1/listings/mine` | Own listings and drafts | Seller |
| PATCH | `/api/v1/listings/{listing}` | Update draft or allowed fields | Seller (owner) |
| POST | `/api/v1/listings/{listing}/media` | Upload image (multipart) | Seller (owner) |
| POST | `/api/v1/listings/{listing}/submit` | Submit for moderation | Seller (owner) |
| POST | `/api/v1/listings/{listing}/reports` | Report a listing | User |

### Auctions and bidding

| Method | Path | Purpose | Auth |
|---|---|---|---|
| GET | `/api/v1/auctions/{auction}` | Auction snapshot (current price, bid count, end time) | Public |
| GET | `/api/v1/auctions/{auction}/bids` | Public bid history (no proxy maximums) | Public |
| POST | `/api/v1/auctions/{auction}/bids` | Place a manual bid (`Idempotency-Key` required) | Eligible buyer |
| PUT | `/api/v1/auctions/{auction}/proxy-bid` | Set a proxy maximum. Returns `NOT_IMPLEMENTED_PENDING_DECISION` until approved. | Eligible buyer |
| GET | `/api/v1/watchlist` | Own watchlist | User |
| POST | `/api/v1/watchlist` | Add an auction | User |
| DELETE | `/api/v1/watchlist/{auction}` | Remove an auction | User |

### Orders, checkout and fulfilment

| Method | Path | Purpose | Auth |
|---|---|---|---|
| GET | `/api/v1/orders` | Own orders (buyer or seller view) | Participant |
| GET | `/api/v1/orders/{order}` | Order detail | Participant |
| POST | `/api/v1/orders/{order}/checkout` | Create or retry payment (`Idempotency-Key` required) | Buyer (winner) |
| POST | `/api/v1/orders/{order}/fulfilment` | Choose shipping or pickup | Buyer (winner) |
| GET | `/api/v1/orders/{order}/shipment` | Tracking status | Participant |
| POST | `/api/v1/orders/{order}/disputes` | Open a dispute | Participant |
| GET | `/api/v1/disputes/{dispute}` | Dispute detail and messages | Participant or assigned staff |

### Notifications

| Method | Path | Purpose | Auth |
|---|---|---|---|
| GET | `/api/v1/notifications` | Own notifications (cursor paginated) | User |
| PATCH | `/api/v1/notifications/{notification}/read` | Mark as read | User |
| GET | `/api/v1/notification-preferences` | Channel preferences | User |
| PUT | `/api/v1/notification-preferences` | Update preferences | User |

### Administration (staff only, separate admin route group and MFA)

| Method | Path | Purpose | Permission |
|---|---|---|---|
| GET | `/api/v1/admin/listings` | Moderation queue | `listing.moderate` |
| POST | `/api/v1/admin/listings/{listing}/moderate` | Approve, reject, pause or remove (reason required) | `listing.moderate` |
| GET | `/api/v1/admin/users` | User search | `user.view` |
| POST | `/api/v1/admin/users/{user}/suspend` | Suspend with reason | `user.suspend` |
| GET | `/api/v1/admin/disputes` | Dispute queue | `dispute.view` |
| POST | `/api/v1/admin/disputes/{dispute}/resolve` | Resolve a dispute | `dispute.resolve` |
| POST | `/api/v1/admin/payments/{payment}/refunds` | Issue a refund (`Idempotency-Key` required) | `payment.refund` |
| GET | `/api/v1/admin/fee-rules` | Fee configuration | `fee_rule.view` |
| PUT | `/api/v1/admin/fee-rules/{rule}` | Update fee rule | `fee_rule.update` |
| GET | `/api/v1/admin/audit-logs` | Audit log | `audit.read` |

### Webhooks and operations

| Method | Path | Purpose | Auth |
|---|---|---|---|
| POST | `/api/webhooks/stripe` | Stripe events (Connect, payments, subscriptions) | Stripe signature |
| POST | `/api/webhooks/paypal` | PayPal events | PayPal verification |
| POST | `/api/webhooks/carriers/{carrier}` | Carrier tracking updates | Carrier signature or shared secret |
| GET | `/api/health` | Liveness and readiness (database, Redis) | Internal network only |

## 4. Example: place a bid

Request:

```http
POST /api/v1/auctions/9b1d3c2e-7f4a-4e8b-a2c1-5d6e7f8a9b0c/bids
Authorization: Bearer <token>            (Flutter)  or  session cookie + X-XSRF-TOKEN (React)
Idempotency-Key: 4f3e2d1c-0b9a-4a8b-8c7d-6e5f4a3b2c1d
Content-Type: application/json

{ "amount_minor": 12500, "currency": "CAD" }
```

Accepted response (`201 Created`):

```json
{
  "data": {
    "accepted": true,
    "bid": {
      "id": "c2a1f0e9-3b4d-4c5e-8f6a-7b8c9d0e1f2a",
      "amount": { "amount_minor": 12500, "currency": "CAD" },
      "accepted_at": "2026-11-21T18:14:03Z"
    },
    "auction": {
      "id": "9b1d3c2e-7f4a-4e8b-a2c1-5d6e7f8a9b0c",
      "status": "active",
      "current_price": { "amount_minor": 12500, "currency": "CAD" },
      "ends_at": "2026-11-21T20:00:00Z"
    }
  }
}
```

Rejected response (`422 Unprocessable Content`):

```json
{
  "type": "https://api.aliauction.ca/errors/bid-too-low",
  "title": "Bid too low",
  "status": 422,
  "code": "BID_TOO_LOW",
  "detail": "The bid must be at least CAD 13.00.",
  "correlation_id": "7c1f2a9e-4d3b-4f6a-9b1e-2a8f0c5d7e11"
}
```

The server validates the auction state and bid rules inside a locked transaction. Clients never decide whether a bid is accepted. A client shows "submitted" until the response confirms "accepted".

## 5. Real-time events (Laravel Reverb)

Channels:
- `private-auction.{auctionId}`: authorised participants and watchers. Requires a channel authorisation endpoint that checks the Policy for the auction.
- `private-user.{userId}`: the user's own notifications and order updates.

Event names (proposed):
- `auction.bid.accepted`
- `auction.price.updated`
- `auction.extended` (only when snipe protection is approved)
- `auction.closed`
- `auction.cancelled`
- `notification.created`

Rules:
- Authenticate every WebSocket connection. Authorise every channel subscription on the server.
- Broadcast only public auction information.
- Never broadcast proxy maximums, personal data or internal risk flags.
- Each event carries an event ID, auction ID, event type and server timestamp.
- Clients recover from missed events by refetching the auction snapshot.
- Events are broadcast only after the database commit (`ShouldBroadcast` with `afterCommit`).

## 6. Webhooks

Provider webhooks must:
1. Verify the provider signature against the raw request body before parsing JSON.
2. Persist the event ID and the minimum required payload in `webhook_events`.
3. Deduplicate by `(provider, provider_event_id)` with a unique index.
4. Return `2xx` promptly after durable receipt.
5. Dispatch a queued job to process the event.
6. Apply state transitions idempotently.
7. Record failures and retry safely.
8. Never treat a browser redirect as proof of payment.

## 7. OpenAPI and client generation

- The Laravel application generates an OpenAPI 3.1 document. CI fails if the generated document differs from the committed one.
- The React app generates TypeScript types and a typed fetch client from the OpenAPI document.
- The Flutter app uses a generated Dart client from the same document.
- Breaking changes require a new API version (`/api/v2`). Additive changes are allowed within `v1`.

## 8. Security and performance

- Rate-limit registration, login, password reset, bidding, reports and checkout using Laravel's `RateLimiter` backed by Redis.
- Validate every input with Form Request classes.
- Use Policies on every route that touches an object. Never rely on hidden buttons or client-side checks.
- Eager-load relationships to avoid N+1 queries. Enforce this in tests with query-count assertions.
- Log correlation IDs and outcomes, not secrets or full payment details.
- Set response-time budgets for bidding, listing search and checkout before launch (see document 14).
