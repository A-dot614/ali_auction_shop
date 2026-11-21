# Delivery Roadmap — AliAuction.ca

The business plan describes three delivery phases: MVP in Months 1–4, enhancements in Months 5–8, and advanced features in Months 9–12. The roadmap translates those phases into workstreams on the confirmed stack. Delivery dates depend on team size, requirements decisions, vendor onboarding and legal review.

## Phase 0 — Discovery, foundations and stack setup

**Suggested duration:** before implementation, plus sprint zero

Deliverables:
- Stakeholder sign-off on the PRD and MVP scope.
- Confirm auction rules and fee model.
- Obtain companion wireframes and design specifications.
- Confirm Canadian legal, privacy and payment review plan.
- Architecture decision record covering the open items in document 17.
- GitHub repository (monorepo), branch protection, CODEOWNERS and Dependabot.
- Laravel 13 application scaffold with Sanctum, Horizon, Reverb, Pest and Larastan configured.
- React + TypeScript (Vite) scaffold with ESLint, Vitest and Playwright configured.
- Docker Compose with `api`, `web`, `worker`, `scheduler`, `reverb`, PostgreSQL 16, Redis and Mailpit.
- OpenAPI generation and TypeScript client generation wired into CI.
- Database ERD and migration plan, reviewed against PostgreSQL 16.
- UX flows and design system.
- GitHub Actions pipeline running on pull requests.
- Threat model and data classification.

Exit criteria:
- Open decisions that block core development are resolved.
- The MVP backlog is estimated and prioritised.
- A "hello world" flow runs end to end: React calls a Laravel REST endpoint, which reads from PostgreSQL 16 and uses Redis, in Docker, deployed to a test environment through GitHub Actions.

## Phase 1 — MVP (business-plan Months 1–4)

Workstreams:
1. Identity and account management (Sanctum, email verification, password reset, MFA for sellers and staff).
2. Seller profile and basic verification workflow.
3. Categories and listing management.
4. Timed auction engine (Laravel Actions, locks, scheduler and queued close jobs).
5. Manual bidding and approved proxy-bidding baseline.
6. Real-time auction updates through Reverb and Laravel Echo.
7. Search and browsing (PostgreSQL full-text search).
8. Checkout and the selected payment provider (Stripe Connect and Cashier for subscriptions).
9. Orders and basic fulfilment.
10. Admin moderation and audit trail.
11. Notifications (mail, database and broadcast channels).
12. Responsive React web application.
13. Security, privacy, support and observability.

Exit criteria:
- P0 PRD requirements accepted.
- Critical end-to-end journeys pass in Playwright.
- Auction and payment tests pass against PostgreSQL 16.
- Policies and legal review complete.
- Production support and monitoring ready.

## Phase 2 — Enhancements (business-plan Months 5–8)

Deliverables:
- Flutter mobile apps for iOS and Android.
- Seller analytics and reporting.
- Advanced automated bidding.
- Snipe protection and configurable bid increments.
- Watchlist and notification preferences.
- Seller reputation and badges.
- Carrier rate and shipping-label integration (Canada Post, Purolator, UPS Canada).

Exit criteria:
- Flutter apps pass App Store and Google Play release requirements.
- Carrier flows are reconciled and tested.
- Analytics definitions are documented.
- Reputation and badge criteria are transparent.

## Phase 3 — Advanced features (business-plan Months 9–12)

Deliverables:
- Live auction streaming and real-time bidding (scaling Reverb and load testing it).
- AI recommendations.
- High-value escrow-style workflow, after legal and provider approval.
- Third-party API marketplace (versioned REST API with API keys and quotas).
- Image recognition and auto-categorisation.
- Advanced fraud detection.

Exit criteria:
- Human review and appeals exist for AI-assisted risk decisions.
- Streaming and bidding are load-tested.
- Financial and legal controls for the escrow-style service are approved.
- API security, quotas and developer terms are defined.

## Cross-phase workstreams

- Security and privacy.
- Compliance and legal.
- Product analytics.
- Customer support.
- Seller acquisition and onboarding.
- Incident response and reliability.
- Documentation (OpenAPI, runbooks, developer onboarding).

## Team skills and roles

The stack needs skills in PHP and Laravel, TypeScript and React, Flutter and Dart, PostgreSQL, Redis, Docker and GitHub Actions, and cloud operations. The business plan projects:
- Year 1: 8 FTEs.
- Year 2: 15 FTEs.
- Year 3: 25 FTEs.

Its Year 1 split is 2 executive, 3 engineering, 1 marketing, 1 operations and 1 customer support. Three engineers covering Laravel, React, Flutter, DevOps and payments is very thin. Plan for at least:
- Two backend engineers with Laravel and PostgreSQL experience.
- One frontend engineer with React and TypeScript.
- One mobile engineer with Flutter (Phase 2, can be contracted earlier).
- DevOps and security responsibility, shared or contracted.

## Planning caution

The 12-month feature roadmap and the Month 16 breakeven target are projections. Treat delivery dates as planning assumptions until technical discovery, staffing, vendor onboarding and legal dependencies are validated.
