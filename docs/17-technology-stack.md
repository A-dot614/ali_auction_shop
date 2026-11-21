# Technology Stack — AliAuction.ca

**Status:** Confirmed by the engineering team (supersedes the proposed stack in the business plan where they differ).
**Purpose:** Single reference for every technology used in the platform, the role each one plays, and the decisions still open around it.

## 1. Confirmed stack

| Area | Technology | Role in AliAuction |
|---|---|---|
| Source control and collaboration | GitHub | Repositories, pull requests, code review, branch protection, issue tracking |
| CI/CD | GitHub Actions | Build, test, scan, image publishing and deployment pipelines |
| Backend framework | Laravel 13 (PHP) | REST API, domain logic, authorization, queues, scheduling, notifications, broadcasting |
| Web frontend | React.js with TypeScript | Buyer, seller and admin web application (SPA) |
| API style | REST over HTTPS with JSON | Contract between web, mobile and backend. Documented with OpenAPI. |
| Primary database | PostgreSQL 16 | Source of truth for all durable business data |
| Cache, queues, locks, rate limits | Redis | Queue backend, cache, short-lived locks and rate-limit counters |
| Containers | Docker | Local development (Docker Compose) and production images |
| Mobile application | Flutter (Dart) | iOS and Android buyer and seller apps (Phase 2) |
| Language (web) | TypeScript | Type-safe React code and generated API types |

## 2. Supporting components (Laravel ecosystem)

These are first-party or widely used Laravel components that fit the confirmed stack. They are recommended defaults and should be confirmed in the first sprint.

| Concern | Component | Notes |
|---|---|---|
| API authentication | Laravel Sanctum | Cookie-based SPA authentication for React on the same domain; personal access tokens for Flutter |
| Authorization | Laravel Policies and Gates | Object-level checks on every request, supporting the RBAC model in document 09 |
| Input validation | Form Request classes | Server-side validation. React validates for usability only. |
| Queue monitoring | Laravel Horizon | Dashboard for Redis queues, failed jobs and throughput |
| Scheduling | Laravel Scheduler | Runs the auction-close sweep and reconciliation commands |
| Real-time events | Laravel Reverb | First-party WebSocket server for live bid updates. Replaces the Socket.io design in the business plan. |
| Client-side real-time | Laravel Echo (React) and a Pusher-protocol client (Flutter) | Subscribe to authorized private auction channels |
| Payments (subscriptions) | Laravel Cashier (Stripe) | Seller subscription lifecycle (Basic, Professional, Enterprise) |
| Payments (marketplace) | stripe-php SDK | Stripe Connect accounts, payment intents, transfers and refunds |
| Search | Laravel Scout (later) | Optional adapter when search outgrows PostgreSQL full-text search |
| API contract | OpenAPI generated from the Laravel API (for example Scramble or an equivalent package) | Source for TypeScript types (React) and a generated Dart client (Flutter) |
| Testing | Pest (on PHPUnit) for PHP; Vitest and React Testing Library for React; Playwright for end-to-end; `flutter test` and `integration_test` for Flutter | See document 14 |
| Code quality | Laravel Pint (formatting), Larastan/PHPStan (static analysis), ESLint and `tsc --noEmit` (TypeScript) | Run in CI on every pull request |
| Dependency security | `composer audit`, `npm audit`, Dependabot on GitHub | See document 10 |
| Error tracking | Sentry or equivalent | Recommended. Decision required (see section 6). |

## 3. How the components fit together

```text
Flutter app ──┐                      ┌── PostgreSQL 16 (durable data)
              │  REST (JSON)         │
React SPA ────┼──────────────► Laravel 13 API ──┼── Redis (cache, locks, rate limits, queues)
              │                      │
              │  WebSocket           ├── Reverb (WebSocket server, private channels)
              └──────────────────────┘
                                     ├── Queue workers (Horizon) ──► mail, webhooks, carriers, search
                                     ├── Scheduler (auction close, reconciliation)
                                     └── Object storage (listing images, documents)
```

## 4. Stack rules

- **PostgreSQL is the source of truth.** Redis may be flushed without losing business state. Every bid, order and payment must be recoverable from PostgreSQL.
- **Business rules live in Laravel, not React or Flutter.** Clients display state and send intent. The server decides whether a bid, checkout or refund is valid.
- **The OpenAPI document is the contract.** Changes to REST endpoints must update the OpenAPI spec, the generated TypeScript types and the generated Dart client in the same pull request.
- **Use real PostgreSQL 16 in tests.** Do not use SQLite for concurrency, locking or migration tests, because its locking behaviour differs from PostgreSQL.
- **Docker parity.** Development, CI and production use the same PHP, Node and PostgreSQL versions, pinned in the Dockerfiles and the lockfiles.
- **Feature flags for risky capabilities.** Live bidding, payments and the mobile release are gated by flags until their tests and approvals pass.

## 5. Changes from the business plan

| Business-plan item | Confirmed stack | Reason |
|---|---|---|
| Node.js with Express | Laravel 13 (PHP) | Team standard |
| GraphQL | REST with JSON and OpenAPI | Team standard. Simpler caching, authorization and webhook handling. |
| Next.js | React.js (TypeScript) SPA | Team standard. SEO for public pages is a decision (section 6). |
| Socket.io | Laravel Reverb | Native fit with Laravel broadcasting and authorization |
| React Native | Flutter | Team standard |
| Tailwind CSS | Not in the confirmed list | Styling approach to be confirmed (section 6) |
| Progressive Web App | Not in the confirmed list | Decision required (section 6) |

## 6. Stack decisions still open

1. **Public SEO rendering.** Options: (a) prerender public listing and seller pages, (b) Inertia.js with server-side rendering, (c) a small server-rendered Laravel Blade layer for public pages only. Recommendation: option (c) for public pages, with React for the authenticated application.
2. **Repository structure.** Monorepo (API, web and shared contracts in one repository) or separate repositories. Recommendation: monorepo for the MVP.
3. **Hosting target on AWS Canada.** ECS on Fargate or EC2 with Docker Compose for the MVP. Kubernetes is deferred. Confirm region `ca-central-1` for every service.
4. **Search.** PostgreSQL full-text search for the MVP. Meilisearch or Elasticsearch via Scout only when catalogue volume justifies it.
5. **Observability.** Datadog (from the business plan) or a Laravel-native option such as Telescope in non-production, Pulse, and Sentry for errors. Decision required.
6. **Styling and PWA.** Confirm whether Tailwind CSS is used and whether the web app is a PWA.
7. **Flutter state management and navigation.** Choose one standard (for example Riverpod or Bloc) before the mobile MVP starts.
8. **Email and SMS providers.** Choose a provider that supports Canadian data residency (for example Amazon SES in `ca-central-1`) and confirm the SMS provider.
9. **Payment flow.** Stripe Connect account and charge model (see document 11).
10. **Laravel and PostgreSQL version policy.** Laravel 13 and PostgreSQL 16 are the baseline. Confirm the upgrade and security-patch policy.
