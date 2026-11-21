# System Architecture — AliAuction.ca

**Architecture status:** Baseline aligned to the confirmed stack (see document 17), subject to technical review
**Confirmed stack:** GitHub, Laravel 13, React.js with TypeScript, REST API, PostgreSQL 16, Redis queues, Docker, Flutter

## 1. Architecture principles

1. Keep financial and auction state changes server-authoritative in Laravel.
2. Treat PostgreSQL 16 as the durable source of truth.
3. Use Redis for queues, cache, locks and rate limits. Never as the only record of bids or payments.
4. Make payment and auction-close operations idempotent.
5. Separate public marketplace data from private user, payment and moderation data.
6. Use queued jobs for notifications, image processing, search indexing and carrier updates.
7. Start with a modular monolith in Laravel, organised by domain. Split services only when scale or team boundaries justify it.
8. Keep production data and backups in approved Canadian regions where required by the privacy assessment.

## 2. Logical architecture

```text
Browser (React SPA, TypeScript)        Mobile (Flutter)
   |                                       |
   | HTTPS / REST (JSON)                   | HTTPS / REST (JSON)
   v                                       v
Laravel 13 API (nginx + PHP-FPM)  <-------+
   |---- Authentication (Sanctum) & Authorization (Policies)
   |---- Identity / Seller
   |---- Catalogue (categories, listings, media)
   |---- Auction & Bidding domain
   |---- Checkout / Orders / Payments
   |---- Fulfilment (shipping, pickup)
   |---- Subscriptions (Cashier) / Fees
   |---- Trust & Safety / Disputes / Admin
   |
   |---- PostgreSQL 16 (durable relational data)
   |---- Redis (queues, cache, locks, rate limits)
   |---- Object storage (listing images and documents)
   |---- Reverb (WebSocket server, private auction channels)
            ^
            | broadcast after commit
Queue workers (Horizon) ----> mail, SMS, push, Stripe/PayPal webhooks,
                              carrier APIs, image processing, search indexing
Scheduler (artisan schedule:work) ----> auction close sweep, reconciliation

External providers: Stripe Connect, PayPal, Canada Post, Purolator, UPS Canada
```

## 3. Deployment shape

The first production release runs as a set of Docker containers from one codebase, each with a single responsibility:

| Container | Process | Notes |
|---|---|---|
| `web` | nginx | Serves the Laravel API and the React static build (or the React build is served from a CDN) |
| `api` | PHP-FPM | Laravel HTTP requests |
| `reverb` | `php artisan reverb:start` | WebSocket server |
| `worker` | `php artisan horizon` | Redis queue workers |
| `scheduler` | `php artisan schedule:work` | Runs scheduled commands; a single instance only |
| `postgres` (local) / managed PostgreSQL 16 (cloud) | | Local container for development; managed service in staging and production |
| `redis` (local) / managed Redis-compatible cache (cloud) | | |

Suggested AWS Canada (`ca-central-1`) components: VPC with private subnets for database and Redis, load balancer with HTTPS, container runtime (ECS on Fargate proposed for the MVP), managed PostgreSQL 16, managed Redis, S3 for media and backups, CloudFront for public media and the React build, Secrets Manager and KMS, centralised logs and alerting.

**Region note:** Confirm the exact region and service availability with the cloud architect and privacy counsel.

## 4. Backend module boundaries

Laravel modules are organised as domain folders (for example `app/Domain/Auction`) with their own actions, policies and events. Modules communicate through explicit service classes and domain events, not by reaching into each other's models.

| Module | Owns |
|---|---|
| Identity | Users, sessions, email verification, MFA |
| Seller | Seller profiles, verification, plan entitlements |
| Catalogue | Categories, attributes, listings, media |
| Auction | Auction state, bids, proxy maximums, increments, closing |
| Checkout | Orders, fee snapshots, checkout state |
| Payments | Provider intents, transactions, refunds, payouts, webhooks |
| Fulfilment | Shipping options, labels, tracking, pickup |
| Subscription | Plans, subscriptions, billing events, entitlements (via Cashier) |
| Trust & Safety | Reports, moderation, disputes, risk flags |
| Notifications | Templates, preferences, delivery attempts |
| Admin | Role assignment, configuration, audit views |
| Search | Indexing and query orchestration |

## 5. Real-time bidding architecture

1. The React or Flutter client sends `POST /api/v1/auctions/{auction}/bids` with an `Idempotency-Key` header.
2. Laravel authenticates the request (Sanctum) and checks the bid policy.
3. The Auction action opens a database transaction and locks the auction row with `lockForUpdate()`.
4. The action checks auction status, server time, bidder eligibility and the minimum valid bid.
5. The bid and the auction state are written in the same transaction, together with an outbox row.
6. The `BidAccepted` event is dispatched with `afterCommit()` so that broadcasting happens only if the transaction succeeds.
7. Reverb broadcasts the event to authorised private channels (for example `private-auction.{id}`).
8. Clients reconcile the event with server state. Reconnecting clients call `GET /api/v1/auctions/{auction}` to fetch a fresh snapshot.

Never accept the browser's countdown or client timestamp as authoritative.

## 6. Background jobs and scheduling

Redis-backed queues handle:
- Auction close jobs and delayed close jobs scheduled at `ends_at`.
- Payment webhook processing.
- Email, SMS and push delivery.
- Search indexing.
- Image resizing and scanning.
- Shipping tracking updates.
- Subscription reconciliation.
- Fraud and risk review tasks.

The scheduler runs `auctions:close-due` every minute as a safety net. Each job must be retry-safe and idempotent: it locks the auction, checks status, and does nothing if the auction is already closed. Failed jobs go to Laravel's `failed_jobs` table and appear in Horizon.

## 7. Data consistency

- PostgreSQL is the source of truth for users, listings, auctions, bids, orders and payments.
- Payment status is reconciled from signed webhooks and a periodic reconciliation command.
- The search index is a derived projection and can be rebuilt from the database.
- Redis values may expire and must be reconstructable.
- Notifications and broadcasts are generated from committed domain events, not from uncommitted browser actions.

## 8. Security zones

- **Public:** published listing summaries and public seller storefront details.
- **Authenticated:** buyer watchlists, bid actions and profile.
- **Participant-only:** order details, shipping addresses, dispute evidence.
- **Staff-only:** moderation queue, risk flags, verification records.
- **Restricted financial:** provider references, payout state, reconciliation data.
- **Secrets:** provider credentials, `APP_KEY`, webhook signing secrets and encryption keys. Stored in the secrets manager and never sent to the browser or Flutter app.

## 9. Key architecture decisions to approve

- Modular monolith in Laravel (recommended) versus separate services.
- Repository structure: monorepo versus separate repositories (see document 17).
- Container hosting on AWS: ECS on Fargate versus EC2 with Docker Compose.
- Reverb scaling: single instance for MVP, with Redis pub/sub for multiple instances.
- Search platform timing: PostgreSQL full-text search for the MVP.
- Cloud region, backups, RPO/RTO and data-retention design.
