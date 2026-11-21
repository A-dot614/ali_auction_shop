# Testing and Quality Plan

## 1. Tooling by layer

| Layer | Backend (Laravel 13) | Web (React + TypeScript) | Mobile (Flutter) |
|---|---|---|---|
| Unit | Pest (PHPUnit) | Vitest | `flutter test` |
| Component or widget | n/a | React Testing Library | Widget tests |
| Integration | Pest feature tests against PostgreSQL 16 and Redis | MSW for API mocking | `integration_test` |
| End-to-end | Via browser tests | Playwright | `integration_test` on device or emulator against staging |
| Contract | OpenAPI check in CI | Generated types must compile | Generated Dart client must compile |
| Static analysis | Larastan, Pint | ESLint, `tsc --noEmit` | `flutter analyze` |
| Accessibility | n/a | axe in Playwright | Semantics tests |
| Load and soak | k6 against staging | n/a | n/a |

**Rule:** backend integration and concurrency tests run against real PostgreSQL 16 and Redis, never SQLite. SQLite does not reproduce row-level locking behaviour, so it cannot prove the bidding rules.

## 2. Test layers

### Unit tests
- Fee calculations.
- Bid validation and increment logic.
- Proxy-bidding algorithm (once approved).
- Auction state transitions.
- Reserve and extension rules (once approved).
- Policy classes (permission checks).
- Subscription entitlement logic.

### Feature and integration tests
- PostgreSQL transactions and `lockForUpdate()` behaviour under concurrency.
- Redis locks, rate limiters and queue dispatch.
- Policy enforcement on every API route.
- Form Request validation rules.
- Stripe and PayPal sandbox flows, using recorded or fake provider responses for CI.
- Signed webhook processing, including duplicates and out-of-order delivery.
- Carrier adapters (fake adapters in CI, sandbox in staging).
- Search indexing and query behaviour.
- Notification dispatch and channel selection.
- Reverb channel authorisation.

### End-to-end tests (Playwright and Flutter integration)
- Register and verify account.
- Seller creates and publishes listing.
- Buyer places bid.
- Auction closes.
- Winner completes checkout.
- Seller fulfils order.
- Buyer opens dispute.
- Staff resolves case and issues approved refund.

### Non-functional tests
- Load and stress tests for concurrent bidding (k6).
- Soak tests for long-running WebSocket connections on Reverb.
- Security tests and penetration testing.
- Accessibility tests.
- Responsive browser and device testing.
- Backup restoration and disaster recovery drill.

## 3. Critical auction test cases

- Simultaneous bids on the same auction (run with parallel requests against PostgreSQL 16).
- Bid received exactly at the end-time boundary.
- Bid received after end time.
- Proxy bids compete and tie.
- Snipe protection extends the auction correctly (once approved).
- Close job runs multiple times.
- Worker restarts during close.
- Database transaction rolls back and no broadcast is sent.
- Socket event is delayed or duplicated.
- Client disconnects and reconnects.
- Suspended bidder tries to bid.
- Seller tries to bid on own item.
- Auction is cancelled with bids present.

## 4. Critical payment test cases

- Successful payment.
- Payment requires additional action.
- Declined payment.
- Buyer closes browser before redirect.
- Duplicate webhook.
- Out-of-order webhook.
- Invalid webhook signature.
- Refund full and partial.
- Refund exceeds captured amount.
- Chargeback or dispute event.
- Provider outage.
- Payment succeeded but local processing failed and must reconcile.

## 5. Authorization test cases

- Buyer cannot view another buyer's address.
- Seller cannot view another seller's orders.
- User cannot modify another user's listing.
- Public API cannot retrieve proxy maximum.
- Support agent cannot issue refund without permission.
- Moderator cannot alter fee configuration.
- Finance operator cannot assign roles.
- Suspended user cannot bid or list.
- Admin routes require MFA.
- Channel subscription to another user's private channel is rejected.

## 6. Quality gates in CI

- Pest and Vitest suites pass with coverage thresholds agreed by the team. Coverage is a signal, not a goal. Critical domain code (auction, payments, policies) is held to a higher bar.
- Query-count assertions on key endpoints to prevent N+1 regressions.
- No `dd()`, `dump()`, `console.log` or `.only` left in committed tests or code.
- Larastan and TypeScript checks pass with no new errors.

## 7. Test data

- Synthetic users, listings, auctions and provider sandbox accounts only.
- Factories and seeders for local development and tests. Seeders never contain real personal or payment data.
- Include Canadian postal-code formats and provinces in fixtures.
- Stripe test cards are used only in sandbox environments.

## 8. Release gates

- All P0 acceptance criteria pass.
- No open critical or high security findings.
- Auction concurrency tests pass against PostgreSQL 16.
- Payment reconciliation and webhook tests pass.
- Accessibility review completed for critical journeys.
- Load test meets the response-time budgets agreed in the architecture review.
- Monitoring and alerts are enabled.
- Backup restoration is verified.
- Legal and privacy approvals are recorded.
- Product owner and engineering lead approve the release.
