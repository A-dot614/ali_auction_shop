# Assumptions, Decisions and Open Questions

This register prevents unspecified business rules from being silently invented during implementation.

## 1. Source-derived facts

The business plan explicitly states:
- AliAuction Inc.; Toronto, Ontario, Canada.
- C2C and B2C auction marketplace.
- Canadian-first positioning and CAD-native transactions.
- Timed auctions and proxy bidding in MVP.
- Stripe Connect and PayPal Business as planned payment processors.
- Canada Post, Purolator and UPS Canada as planned carrier partners.
- The business plan's proposed stack (React/Next.js, Node.js/Express, GraphQL, Socket.io, React Native, Elasticsearch, Kubernetes, Datadog) is superseded where it conflicts with the confirmed stack in `17-technology-stack.md`. AWS Canada, Docker and GitHub Actions remain as planned.
- Basic, Professional and Enterprise seller subscription tiers.
- 8% transaction commission in the revenue model.
- First five listings per month free; CAD $1.99 for additional listings.
- Featured listings CAD $4.99–$14.99.
- Professional subscription CAD $29.99/month and Enterprise CAD $99.99/month.
- High-value escrow concept for items CAD $500+.
- Phase 1 Months 1–4, Phase 2 Months 5–8, Phase 3 Months 9–12.

## 2. Assumptions used in this implementation pack

These are proposed implementation defaults, not confirmed requirements:
- One listing has one timed auction in the MVP.
- A user may be both a buyer and a seller.
- Seller cannot bid on their own listing.
- PostgreSQL is the durable source of truth.
- Financial values use integer minor units.
- Auction time decisions use server UTC time.
- Public users see accepted bid history but not proxy maximums.
- MVP uses a modular monolith in Laravel rather than many microservices.
- Search starts with PostgreSQL full-text search and may move to Meilisearch or Elasticsearch (through Laravel Scout) when justified.
- Manual shipping configuration may be used if carrier APIs are not ready.
- High-risk admin actions require audit logs and may require dual approval.

## 3. Decisions required before auction implementation

1. Are auctions English ascending auctions only?
2. Does the bid field represent a next bid or a maximum bid?
3. What are the bid increments by price band/category?
4. How are equal proxy maximums resolved?
5. What is the snipe-protection trigger window and extension duration?
6. Is there a maximum number of extensions?
7. Are reserve prices enabled in MVP?
8. Is reserve status visible to bidders?
9. Can sellers cancel an auction after the first bid?
10. What happens if the winner does not pay?
11. How long does the winner have to pay?
12. Are bidders required to add a verified payment method before bidding?
13. Are deposits or pre-authorizations required for high-value bids?
14. What happens if a user is suspended while bidding?

### Slice 4a placeholder decisions (recorded 2026-11-16)

Slice 4a implements auction creation and manual bidding behind `FEATURE_LIVE_BIDDING_ENABLED`, which remains **OFF by default** and is asserted off at API startup. Live bidding is code-complete but **enabled for no user**. The items below remain **OPEN**; the values in Slice 4a are placeholders, marked `NOT FOR PRODUCTION` in code and not approved rules.

- **§3 #1 Auction format:** the Slice 4a data model assumes **English ascending auctions only**. Not approved; no other format is implemented.
- **§3 #2 Bid input semantics:** placeholder = the submitted `amountMinor` is the **next bid amount** (not a maximum/proxy bid). Proxy/maximum input is rejected via `setProxyBid` → `NOT_IMPLEMENTED_PENDING_DECISION`.
- **§3 #3 Bid increments:** placeholder = **one configured increment**, `AUCTION_BID_INCREMENT_MINOR`, default `100` minor units (CAD $1.00). No price-band/category schedule; not hardcoded in resolvers.
- **§3 #4 Proxy tie-breaking:** **not implemented**. The `proxy_bids` table is created but unused; any proxy path throws `NOT_IMPLEMENTED_PENDING_DECISION`.
- **§3 #5 Snipe-protection window/extension:** **disabled**. No trigger window, no extension logic, and no `extended` auction status.
- **§3 #6 Maximum number of extensions:** **not implemented** (snipe protection disabled, so no extension cap applies yet).
- **§3 #7 Reserve prices in MVP:** **disabled**. `auctions.reserve_price_minor` exists as a nullable column and is unset/unused in Slice 4a.
- **§3 #8 Reserve status visible to bidders:** **not implemented** (reserve disabled).
- **§3 #9 Seller cancellation after the first bid:** **OPEN**. The state machine allows cancellation only from `scheduled` with no bids; cancel-after-bid is not implemented.

## 4. Decisions required for fees and revenue

1. Is the 8% commission charged to buyer, seller or split?
2. Is it calculated on hammer price only?
3. Are shipping, tax and other fees excluded from the commission base?
4. Are fees refundable if an order is cancelled or refunded?
5. Are there category-specific fee rates?
6. Are first five listings free per calendar month or rolling 30 days?
7. Does the listing fee apply when a listing is submitted, approved or published?
8. What exact entitlements apply to subscriptions?
9. Are subscriptions billed monthly only?
10. Are featured placements sold per listing and for what duration?
11. What are advertising pricing, eligibility and disclosure rules?

**Financial reconciliation required:** The business plan's Year 1 commission line, total revenue, GMV and marketing budget require clarification before billing and financial reporting are implemented.

## 5. Decisions required for payments and payouts

1. Which provider is the MVP primary processor?
2. What Stripe Connect account type and charge model will be used?
3. Who is merchant of record?
4. What is the seller onboarding/KYC responsibility split?
5. When is payment captured: at winning bid, checkout or shipment?
6. Is payment authorization required before bidding?
7. When is seller payout released?
8. How are reserves, refunds, chargebacks and negative balances handled?
9. Is PayPal available at launch or later?
10. What exactly is meant by escrow, and is the proposed model legally/provider approved?

## 6. Decisions required for shipping and fulfilment

1. Is shipping mandatory, optional or category-dependent?
2. Can sellers offer both shipping and pickup?
3. Who pays shipping?
4. Which provinces are supported at launch?
5. Are cross-border orders supported?
6. Which carrier API is integrated first?
7. Are sellers allowed to use their own shipping labels?
8. What happens when a package is lost or damaged?
9. How are pickup no-shows and disputes handled?

## 7. Decisions required for identity, privacy and trust

1. What identity checks are required for individual sellers?
2. What business documents are required for professional sellers?
3. Is buyer verification required before bidding?
4. What age restrictions apply?
5. What items are prohibited or restricted?
6. What is the seller appeals process after suspension?
7. What data retention periods apply to identity, orders, bids and disputes?
8. What privacy request workflows are required?
9. Which provinces are in the initial operating scope?
10. Which legal counsel approves terms, auction rules, privacy and consumer disclosures?

## 8. Decisions required for product and operations

1. What are the approved categories and category attributes?
2. Is listing moderation manual, automated or hybrid?
3. What support hours and languages are offered?
4. What is the dispute-resolution policy and evidence window?
5. What are the official SLA targets?
6. What are the production availability, RPO and RTO targets?
7. What is the launch province/city rollout?
8. What are the launch-day user and traffic assumptions?
9. What analytics and attribution tools are approved?
10. Is the companion wireframe/design specification available?

## 9. Source document inconsistencies to resolve

The source business plan contains items that should be clarified:
- Year 1 GMV is CAD $5.25M and the commission is described as 8%, but the commission line shows CAD $252,000. Eight percent of CAD $5.25M is CAD $420,000.
- Year 1 total revenue is CAD $420,000 and take rate is stated as 8%, while revenue streams include multiple other sources. The treatment of commission and other streams should be reconciled.
- Year 1 marketing budget allocation totals CAD $80,000, while the P&L marketing and advertising expense is CAD $30,000.
- Seed funding allocates CAD $50,000 to marketing, while the marketing budget is CAD $80,000.
- The source table of contents refers to page numbers beyond the 25-page supplied PDF.
- The founding leadership names are marked `[To Be Confirmed]`.
- The referenced companion wireframe/design specification was not included.

## 10. Technology stack decisions

Confirmed by the engineering team:

| Item | Status |
|---|---|
| GitHub and GitHub Actions | Confirmed |
| Laravel 13 (PHP) backend | Confirmed |
| React.js with TypeScript web frontend | Confirmed |
| REST APIs with JSON | Confirmed |
| PostgreSQL 16 | Confirmed |
| Redis queues | Confirmed |
| Docker | Confirmed |
| Flutter mobile apps | Confirmed |

Open stack decisions (detail in `17-technology-stack.md`, section 6):

| Decision | Owner | Status |
|---|---|---|
| Public SEO rendering approach for React | Engineering + product | Open |
| Monorepo versus separate repositories | Engineering lead | Recommended: monorepo, open |
| AWS hosting model (ECS on Fargate or EC2 with Compose) | DevOps | Open |
| Kubernetes adoption timing | DevOps | Deferred |
| Search platform timing | Engineering + product | Recommended: PostgreSQL FTS for MVP |
| Monitoring platform (Datadog or Laravel-native plus Sentry) | DevOps | Open |
| Styling approach (Tailwind CSS or other) and PWA | Design + engineering | Open |
| Flutter state management and navigation | Mobile lead | Open, needed before Phase 2 |
| Email and SMS providers, Canadian residency | Engineering + privacy | Open |
| Reverb scaling model | Engineering | Recommended: single instance for MVP, Redis pub/sub later |
| Laravel and PostgreSQL upgrade policy | Engineering | Open |

Items to verify before use:
- Confirm the exact Laravel 13 and PHP version requirements against the official release notes before setting the Docker base image.
- Confirm the Reverb, Horizon, Sanctum and Cashier major versions compatible with Laravel 13.

## 10. Approval log template

| Decision | Owner | Status | Date | Notes |
|---|---|---|---|---|
| MVP scope | Product owner | Open | — | — |
| Auction rules | Business + product | Open | — | — |
| Fee model | Finance + business | Open | — | — |
| Payment architecture | Engineering + finance + legal | Open | — | — |
| Shipping approach | Operations | Open | — | — |
| Privacy/legal review | Legal/privacy lead | Open | — | — |
| Cloud architecture | Engineering/DevOps | Open | — | — |
| UI/UX design approval | Product/design | Open | — | — |
