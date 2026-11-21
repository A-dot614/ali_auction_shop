# Third-Party Integrations Plan

The business plan names Stripe Connect, PayPal Business, Canada Post, Purolator, UPS Canada, AWS, Elasticsearch, email and support tooling, and Datadog. This document maps each integration to the confirmed Laravel 13 stack. Exact products, contracts and API capabilities must be verified during procurement.

## 1. Payments — Stripe Connect (marketplace payments)

**Laravel integration:** `stripe/stripe-php` SDK, called from Payment domain actions. Webhooks handled at `/api/webhooks/stripe` and processed by queued jobs.

Purpose:
- Marketplace payment collection.
- Seller connected accounts.
- Platform fees and split payments, where supported.
- Refund and payout status.

Implementation requirements:
- Confirm the supported Canadian Connect account model and onboarding requirements.
- Determine the charge model (destination charges, separate charges and transfers, or another supported flow).
- Define the merchant of record and who bears chargeback and refund liability.
- Use Stripe-hosted or tokenised payment collection. Store provider IDs and status, never card data.
- Send an `Idempotency-Key` on every Stripe write, derived from the internal `payment_transactions.idempotency_key`.
- Verify webhook signatures with the raw request body (`Stripe-Signature` header and the endpoint secret).
- Reconcile internal order, payment and payout ledgers with a scheduled command.

**Decision required:** Stripe Connect is named in the plan, but the payment and payout flow is not yet specified.

## 2. Subscriptions — Laravel Cashier (Stripe)

Seller plans (Basic, Professional, Enterprise) are billed through Cashier:
- Billing customer and subscription lifecycle.
- Trial, plan change, cancellation and failed-renewal handling.
- Entitlements change only after Cashier or webhook confirmation of the subscription state. Never on the browser's redirect.

Plan prices are configured in Stripe and mirrored in `subscription_plans`. The mapping is checked in tests.

## 3. Payments — PayPal Business

The plan lists PayPal Business as an additional processor. Decide:
- Whether PayPal is available at MVP or later.
- Whether it supports the required marketplace split and payout model in the intended account configuration.
- How refunds, disputes and reconciliation work across both providers.
- Whether buyers choose a provider or routing is controlled by the platform.

Integration: the official PayPal REST SDK or a maintained PHP wrapper, behind a `PaymentProvider` interface so that Stripe and PayPal share one domain contract. Webhooks at `/api/webhooks/paypal` with PayPal's verification API.

Do not assume PayPal Business alone provides all marketplace split-payment functionality.

## 4. Shipping carriers

Carriers named in the plan: Canada Post, Purolator and UPS Canada.

Integration: one adapter class per carrier implementing a shared `CarrierAdapter` interface (rate, create shipment, label, track, void). Carrier calls run inside queued jobs with timeouts and retries. Tracking updates arrive via webhook where supported, otherwise via a scheduled polling job.

Required capabilities:
- Rate estimates.
- Shipment creation, label generation and tracking number retrieval.
- Tracking updates by webhook or polling.
- Label cancellation (void).
- Pickup or drop-off options where available.

Data required:
- Origin and destination postal codes.
- Package dimensions and weight.
- Service level.
- Declared value and insurance requirements.
- Customs data for cross-border shipments, if enabled.

MVP can begin with seller-entered shipping cost and manual tracking while carrier contracts and APIs are validated. This is a proposed scope option, not a business-plan commitment.

## 5. Cloud and storage — AWS Canada (`ca-central-1`)

The plan identifies `ca-central-1` and EC2, S3, CloudFront and RDS. The proposed use:

| Need | Service (proposed) | Laravel integration |
|---|---|---|
| Container hosting | ECS on Fargate (or EC2 with Docker Compose) | Docker images from GitHub Actions |
| Database | RDS for PostgreSQL 16 | `DB_*` environment variables |
| Queue and cache | ElastiCache for Redis (or a Redis-compatible service) | `REDIS_*` variables, Redis queue driver |
| Object storage | S3 (private bucket) | `league/flysystem-aws-s3-v3` through the `s3` filesystem disk |
| CDN | CloudFront | Public media and React build |
| Email | Amazon SES in `ca-central-1` (proposed) | Laravel mail driver `ses` |
| Secrets | Secrets Manager | Injected as environment variables at deploy time |
| Keys | KMS | Encryption at rest and backups |

Confirm availability, region coverage, pricing and data residency for every chosen service.

## 6. Search

For the MVP, use PostgreSQL full-text search with a `tsvector` column and a GIN index. Keep the query behind a `ListingSearch` service so the implementation can change.

Move to Meilisearch or Elasticsearch through Laravel Scout only when catalogue volume or relevance requirements justify it. PostgreSQL remains the source of truth. Index updates are driven by queued jobs after commit. Provide a reindex command and a failure-recovery runbook.

## 7. Email, SMS and push notifications

The source requires email support and does not name providers. Laravel's notification system dispatches to channels:
- `mail` for transactional email (provider chosen in section 5).
- `database` for in-app notifications (`notifications` table).
- `broadcast` for real-time updates through Reverb.
- SMS and push channels are added when providers are approved. Flutter push uses Firebase Cloud Messaging or Apple Push Notification service through a community or first-party Laravel channel.

Provider selection criteria:
- Canadian deliverability and privacy requirements.
- Transactional templates.
- Bounce and complaint webhooks.
- SMS sender registration and country support.
- Push support for the Flutter apps.
- Data processing terms and retention.
- Cost and observability.

Events include registration verification, password reset, outbid, auction ending, win and loss, payment, shipping, dispute and subscription notices.

## 8. Real-time — Laravel Reverb

Reverb is the WebSocket server for live auction updates. It is part of the Laravel application and is configured in the same repository. It uses Redis pub/sub when more than one instance runs. Clients: Laravel Echo in React and a Pusher-protocol client in Flutter. See document 06 for channels and events.

## 9. Search and analytics

- Product analytics tool: to be approved (see document 16).
- Analytics events must not contain personal data, bid maximums or payment details.

## 10. Observability — Datadog or Laravel-native

The business plan names Datadog. Options:
- Datadog APM and logs with the `dd-trace` PHP extension, with Docker log collection.
- Laravel Pulse for application metrics, plus Sentry for error tracking.

Decision required (see document 17). Either option must cover the metrics listed in document 13.

## 11. AI services (later phase)

Planned uses:
- Recommendations.
- Fraud detection.
- Image recognition and automatic categorisation.
- AI chatbot and knowledge base.

AI output is advisory for high-impact decisions. Do not automatically suspend users or reject valuable listings based only on model output. Define human review, appeal and monitoring. Model calls run from queued jobs.

## 12. Integration governance

For every integration:
- Name a technical owner.
- Store credentials in the secrets manager and never commit `.env` files.
- Keep sandbox and production configuration separate.
- Document webhook authentication and retry policy.
- Add provider outage handling: timeouts, circuit breaking and a manual fallback for critical workflows.
- Track API version and deprecation dates.
- Monitor latency, error rate and quota usage.
