# AliAuction.ca — Implementation Documentation Index

**Product:** AliAuction.ca  
**Company:** AliAuction Inc.  
**Source business plan:** AliAuction.ca Business Plan 2026–2029, prepared April 2026  
**Document set:** Software implementation planning pack  
**Status:** Draft for product, engineering, design and business review (stack updated to the confirmed engineering stack)  
**Classification:** Confidential — For Authorized Recipients Only

## Purpose

This documentation pack translates the supplied business plan into a practical software-development blueprint for a working web application. It is intended to guide product discovery, UX/UI design, backend and frontend implementation, database design, integrations, QA and deployment.

The business plan is the source for the business concept, proposed technology stack, revenue model, planned capabilities and high-level delivery phases. Where the business plan does not define implementation-level behaviour, this pack labels the item as an **assumption**, **proposal** or **decision required** rather than presenting it as an approved business rule.

## Documents

| File | Purpose |
|---|---|
| `01-product-requirements-document.md` | Product vision, goals, scope, functional and non-functional requirements, acceptance criteria |
| `02-epics-user-stories.md` | Epics, user stories and testable acceptance criteria |
| `03-system-architecture.md` | Proposed application architecture, services, deployment topology and data flows |
| `04-database-design.md` | PostgreSQL entities, relationships, constraints, indexes and data lifecycle |
| `05-postgresql-starter-schema.sql.md` | Starter SQL DDL for core entities; review and split into migrations before production |
| `06-api-specification.md` | REST API conventions, endpoint inventory, sample requests, errors, webhooks and real-time events |
| `07-auction-engine-and-business-rules.md` | Auction lifecycle, bidding, proxy bidding, increments, reserves, extensions and settlement rules |
| `08-user-flows-and-screen-inventory.md` | Key journeys and screen inventory for web application UX |
| `09-roles-permissions-and-admin.md` | Roles, permission matrix and administrative controls |
| `10-security-privacy-and-compliance.md` | Security requirements, privacy controls, payment boundaries and compliance workstream |
| `11-third-party-integrations.md` | Payments, shipping, email/SMS, search, analytics and AI integration plans |
| `12-frontend-implementation-guide.md` | React + TypeScript application structure, state, forms, Flutter mobile plan, accessibility and responsive UI |
| `13-devops-and-deployment-plan.md` | Environments, GitHub, Docker, GitHub Actions CI/CD, AWS Canada, Kubernetes (deferred), observability and backups |
| `14-testing-and-quality-plan.md` | Test strategy, critical scenarios, performance, security and release gates |
| `15-delivery-roadmap.md` | Phased implementation plan, milestones, dependencies and definition of done |
| `16-assumptions-decisions-and-open-questions.md` | Items that must be decided before or during implementation |
| `17-technology-stack.md` | Confirmed technology stack, supporting components, stack rules and open stack decisions |

## How to use this pack

1. Review the PRD and the assumptions/open-questions register with the business owner.
2. Confirm MVP scope and explicitly defer later-phase features.
3. Approve the auction and payment rules before implementing bid settlement or charging users.
4. Review the data model and SQL with the backend engineer before creating production migrations.
5. Convert user stories into tickets with owners, estimates and sprint milestones.
6. Use the testing and release documents as mandatory acceptance gates.

## Confirmed technology stack

The engineering team has confirmed the following stack. Full details are in `17-technology-stack.md`.
- GitHub (source control, pull requests, GitHub Actions CI/CD).
- Laravel 13 (PHP) REST API backend.
- React.js with TypeScript for the web application.
- REST APIs over HTTPS with JSON, documented with OpenAPI.
- PostgreSQL 16 as the primary database.
- Redis for queues, cache, locks and rate limits.
- Docker for local development and production images.
- Flutter for iOS and Android apps (Phase 2).

## Source-derived scope

The supplied business plan also contains product scope and some technology proposals. Where the business plan and the confirmed stack differ, the confirmed stack applies:
- Node.js/Express, GraphQL, Socket.io and React Native are replaced by Laravel 13, REST, Laravel Reverb and Flutter.
- Next.js and Tailwind CSS are replaced by a React + TypeScript SPA. SEO rendering and styling are open decisions.
- Elasticsearch is deferred. PostgreSQL full-text search is the MVP baseline.
- Kubernetes and Datadog remain open decisions.
- AWS Canada (`ca-central-1`), Stripe Connect and PayPal Business, carrier integrations and the product scope (timed auctions, proxy bidding, seller dashboards, watchlists, notifications, shipping labels, live auctions, recommendations, escrow for high-value items, image categorisation and fraud detection) are unchanged.

These are planned choices, not proof that services, contracts, compliance controls or integrations are already active.

## Important implementation warning

This is a planning specification, not a legal opinion, final security certification, production-ready payment/escrow design or complete executable application. Payment flows, escrow terminology, privacy obligations, taxes, consumer protection, marketplace liability, auction law, refunds and seller verification require qualified Canadian legal, tax, payment and compliance review before launch.
