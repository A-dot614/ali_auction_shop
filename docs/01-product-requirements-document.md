# Product Requirements Document (PRD) — AliAuction.ca

**Version:** 0.1  
**Status:** Draft for stakeholder approval  
**Product type:** Canadian C2C and B2C online auction marketplace  
**Primary market:** Canada  
**Currency baseline:** CAD  
**Source:** AliAuction.ca Business Plan 2026–2029

## 1. Product overview

AliAuction.ca is a proposed Canadian auction marketplace connecting individual sellers, small businesses, collectors and institutional sellers with buyers. The platform should let sellers publish auction listings and manage sales, while buyers discover items, place bids, follow auctions, pay for won items and track fulfilment.

The product differentiator described in the business plan is a Canadian-first auction experience: CAD-native pricing, Canadian shipping options, buyer/seller trust features, transparent fees and community-oriented marketplace features.

## 2. Product vision

Build a transparent, trusted and technology-forward auction platform that makes online auctions accessible to Canadians and aims to become a leading homegrown Canadian auction marketplace by 2029.

## 3. Goals and success measures

### Business goals from the source plan

| Goal | Source target |
|---|---:|
| Year 1 revenue | CAD $420,000 |
| Year 3 revenue | CAD $2.8 million |
| Year 1 GMV | CAD $5.25 million |
| Year 3 GMV | CAD $28 million |
| Year 1 monthly active users | 25,000 |
| Year 1 active buyers | 15,000 |
| Year 1 active sellers | 1,200 |
| Annual seller retention | 70% |
| Buyer CAC | CAD $20 |
| Seller CAC | CAD $150 |
| LTV:CAC | Greater than 3:1 |
| Breakeven target | Month 16 |

These are business-plan projections, not guaranteed outcomes. The financial model has internal figures that require reconciliation; see `16-assumptions-decisions-and-open-questions.md`.

### Product success measures to instrument

- Visitor-to-registration conversion.
- Registration-to-first-bid conversion.
- Seller onboarding completion.
- Listing creation completion and listing approval time.
- Number of active auctions and unique bidders per auction.
- Bid acceptance/error rate and real-time event delivery latency.
- Auction sell-through rate.
- Checkout completion and payment failure rate.
- Dispute rate, refund rate and chargeback rate.
- On-time shipment rate and tracking coverage.
- Repeat buyer and seller retention.
- Support first-response and dispute-resolution time.

Targets for these product metrics are **to be agreed**; do not invent baselines.

## 4. Users and personas

### Buyer
A Canadian user who searches for products, watches auctions, bids, pays for won items and follows shipping or pickup.

### Individual seller
A person listing personal property, collectibles or household items and managing auctions, orders and payouts.

### Professional seller
A retailer, liquidator or artisan who needs higher listing limits, analytics, store branding and potentially subscription features.

### Institutional seller
A public-sector or organizational seller disposing of surplus equipment, vehicles, furniture or IT assets. This segment may require onboarding and approval workflows.

### Marketplace administrator
An internal operator who manages users, listings, disputes, reports, categories, fees and marketplace safety.

### Customer support / trust and safety agent
An internal user who reviews reports, assists users and manages disputes within assigned permissions.

## 5. Product scope

### MVP — launch foundation

The source plan places the MVP in Months 1–4. The following scope translates its listed deliverables into functional requirements.

1. Account registration, login, password reset and user profiles.
2. Seller profile and basic verification status.
3. Listing creation, editing, submission and publication.
4. Categories, item attributes, image uploads and listing search/browse.
5. Timed auctions with defined start/end times.
6. Standard bidding and proxy bidding, subject to approved bidding rules.
7. Auction status, current high bid and bid history.
8. Winner determination at auction close.
9. Checkout and payment-provider integration.
10. Basic order record and seller/buyer order views.
11. Responsive web application / PWA.
12. Basic admin tools for users, listings and auction moderation.
13. Transactional email notifications for essential account, bid, auction and order events.
14. Audit logging for sensitive actions.

**MVP release gate:** No live public bidding or real-money transaction launch until auction settlement, payment capture, refunds, identity checks, fraud controls, privacy notices, terms and support escalation are reviewed and tested.

### Phase 2 — enhancement

The source plan lists Months 5–8:
- Flutter iOS and Android apps.
- Seller dashboard with analytics and reporting.
- Automated bidding improvements, snipe protection and bid increments.
- Watchlist and notification system.
- Seller reputation and badge system.
- Integrated shipping label generation.

### Phase 3 — advanced capabilities

The source plan lists Months 9–12:
- Live auction streaming with real-time bidding.
- AI-powered recommendations.
- Escrow services for high-value items (CAD $500+), subject to legal/payment design.
- API marketplace for third-party integrations.
- Image recognition for auto-categorization.
- Advanced fraud detection.

## 6. Functional requirements

Priority: **P0** = launch-critical; **P1** = next release; **P2** = advanced/later.

### Identity and accounts

- FR-001 (P0): A visitor can register using an email address and password.
- FR-002 (P0): A user can verify their email before using protected marketplace functions.
- FR-003 (P0): A user can log in, log out and request a password reset.
- FR-004 (P0): A user can manage profile and contact information.
- FR-005 (P0): The system stores account status and verification status separately.
- FR-006 (P0): The system enforces authorization on the server, not only in the UI.
- FR-007 (P1): Seller identity/business verification can be submitted and reviewed.
- FR-008 (P1): Two-factor authentication is supported; the source plan says it is mandatory for sellers.

### Listings

- FR-010 (P0): An eligible seller can create a listing with title, description, category, condition, images, location, auction start/end and starting price.
- FR-011 (P0): The system validates required listing fields and auction timing.
- FR-012 (P0): A seller can save a draft and edit an unpublished listing.
- FR-013 (P0): The platform can approve, reject, pause or remove a listing through authorized moderation.
- FR-014 (P0): Published listings appear in category browsing and search.
- FR-015 (P1): Sellers can purchase featured placement.
- FR-016 (P1): Listing limits depend on the seller's plan.
- FR-017 (P2): Image recognition can suggest categories and attributes, with human correction.

### Auctions and bidding

- FR-020 (P0): The system creates a timed auction with a start time and end time.
- FR-021 (P0): An eligible buyer can place a bid while the auction is open.
- FR-022 (P0): The system validates bid amount, user eligibility and auction state atomically.
- FR-023 (P0): The system records an immutable bid history.
- FR-024 (P0): The system updates the current high bid and winning bidder consistently.
- FR-025 (P0): The system closes an auction and determines its outcome.
- FR-026 (P1): Proxy bidding automatically places incremental bids up to a bidder's confidential maximum.
- FR-027 (P1): Bid increments can be configured by the platform.
- FR-028 (P1): Snipe protection can extend an auction when a qualifying bid arrives near closing time; exact window and extension rules require approval.
- FR-029 (P1): Buyers can add auctions to a watchlist.
- FR-030 (P1): Buyers receive bid/outbid/ending/win/loss notifications.
- FR-031 (P2): Live auction sessions support real-time bidding and host controls.

### Orders and payments

- FR-040 (P0): The system creates an order for a winning auction after the close process.
- FR-041 (P0): The buyer can view amount due, fees, shipping/pickup selection and payment status.
- FR-042 (P0): The platform integrates a payment provider using hosted/tokenized payment methods.
- FR-043 (P0): Payment status is updated from verified provider webhooks, not browser redirects alone.
- FR-044 (P0): The platform records refunds, payment failures and chargeback events.
- FR-045 (P0): The platform calculates and displays all fees before payment.
- FR-046 (P1): Seller payouts are tracked and released according to an approved payout policy.
- FR-047 (P2): High-value escrow-like handling is implemented only after legal and provider approval.

### Shipping and fulfilment

- FR-050 (P0): A seller can specify shipping, local pickup or both, according to listing rules.
- FR-051 (P0): A buyer can see fulfilment options and estimated charges before payment.
- FR-052 (P1): The platform integrates carrier rates and label creation with supported carriers.
- FR-053 (P1): Tracking numbers and shipment status are stored and shown to buyers.
- FR-054 (P1): Pickup windows and pickup confirmation can be recorded.

### Seller tools and subscriptions

- FR-060 (P0): Sellers can view their listings and auction outcomes.
- FR-061 (P1): Seller dashboards show sales, listing and performance metrics.
- FR-062 (P1): The platform supports Basic, Professional and Enterprise plan entitlements.
- FR-063 (P1): Subscription billing status controls listing limits and premium features.
- FR-064 (P1): Sellers can purchase featured listing placement.

### Trust, support and administration

- FR-070 (P0): Users can report a listing or another user.
- FR-071 (P0): Admins can suspend users and hide listings with a recorded reason.
- FR-072 (P0): Users can open a support request or dispute tied to an order.
- FR-073 (P0): Authorized staff can review and resolve disputes with an audit trail.
- FR-074 (P0): Admin actions and financial state changes are auditable.
- FR-075 (P1): Seller badges and buyer reputation indicators can be displayed using approved criteria.
- FR-076 (P1): AI-assisted fraud flags are advisory and can be reviewed by staff.

## 7. Non-functional requirements

- Performance: common page loads and API requests should meet agreed response-time budgets; define budgets during architecture validation.
- Real-time: bid updates should be delivered quickly and consistently; define latency and reconnect targets before launch.
- Availability: define an MVP service-level objective and maintenance policy before public launch.
- Scalability: support growth in concurrent auctions, bidders and notifications; load-test auction close and bid contention.
- Security: use TLS, secure password hashing, least privilege, secrets management, audit logs and secure payment-provider integrations.
- Privacy: collect only necessary personal data, publish clear privacy notices, define retention/deletion workflows and validate Canadian privacy obligations.
- Accessibility: target WCAG 2.2 AA for core buyer and seller journeys.
- Responsive design: support mobile, tablet and desktop breakpoints.
- Observability: structured logs, metrics, traces, alerting and incident response.
- Recovery: backups and restore drills; define RPO/RTO with business stakeholders.
- Maintainability: automated tests, code review, migrations and documented REST API contracts (OpenAPI).
- Technology constraints: the confirmed stack in `17-technology-stack.md` (Laravel 13, React + TypeScript, REST, PostgreSQL 16, Redis queues, Docker, GitHub, Flutter) is used unless an approved architecture decision changes it.

## 8. Out of scope for initial MVP

Unless explicitly approved, defer:
- Flutter mobile apps (iOS and Android) before the Phase 2 release.
- Live streaming auctions.
- AI recommendations and image classification.
- Public third-party API marketplace.
- Automated escrow service.
- Advanced seller analytics and enterprise account management.
- Complex cross-border multi-currency checkout.

## 9. Acceptance and launch criteria

MVP is not launch-ready until:
1. Critical buyer and seller journeys pass end-to-end tests.
2. Auction concurrency and close-time edge cases pass automated tests.
3. Payment webhook verification, idempotency, refund and failure paths pass.
4. Authorization tests show users cannot access another user's private records.
5. Admin actions are audited.
6. Privacy policy, terms, auction rules, fee disclosure and refund/dispute policy are approved.
7. Security review and vulnerability remediation are complete.
8. Backups and restore process are verified.
9. Monitoring and support escalation are operational.
10. Product owner signs off all P0 requirements.
