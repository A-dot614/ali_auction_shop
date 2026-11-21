# Database Design — AliAuction.ca

**Primary database proposed in source:** PostgreSQL  
**Design status:** Conceptual logical model; validate before migrations  
**Currency baseline:** CAD  
**Time storage:** Store timestamps in UTC (`timestamptz`); render in user/local timezone.

## 1. Design principles

- Use UUID primary keys for externally exposed domain records. In Laravel, use the `HasUuids` trait with `uuid` primary key columns.
- Use `timestamptz` for event timestamps.
- Store money as integer minor units (e.g. cents) or `numeric(12,2)` consistently; do not use floating point.
- Store currency code on monetary records, even if CAD is the default.
- Keep provider identifiers separate from internal IDs.
- Use immutable financial and bidding ledgers where practical.
- Add `created_at` and `updated_at` to mutable entities.
- Soft-delete (Laravel `SoftDeletes`) or archive user-facing content where business/legal retention requires it; do not hard-delete financial/audit records without an approved retention policy.
- Use database constraints for uniqueness and valid ranges, plus service-layer validation.

## 2. Core entity groups

### Identity and access
- `users`: account identity and status.
- `user_profiles`: public/display profile.
- `user_addresses`: shipping/billing addresses; restricted access.
- `roles`, `permissions`, `user_roles`, `role_permissions`: access control.
- `sessions` / token records: session management, if not delegated to a secure identity provider.
- `seller_profiles`: seller-specific settings and onboarding.
- `seller_verifications`: verification submissions and review status.

### Catalogue
- `categories`: hierarchical categories.
- `listings`: item/auction listing details and seller ownership.
- `listing_media`: image/document metadata and storage keys.
- `listing_attributes`: flexible category-specific attributes, with typed validation strategy.
- `listing_moderation_events`: review and moderation history.

### Auctions
- `auctions`: timing, starting price, reserve, current price and lifecycle status.
- `bids`: accepted bid ledger.
- `proxy_bids`: confidential bidder maximums.
- `bid_increment_rules`: configured increments by price band/category.
- `auction_events`: state transition history.
- `watchlist_items`: buyer watchlist.

### Commerce and money
- `orders`: buyer, seller, winning listing/auction and order lifecycle.
- `order_items`: item and price snapshot.
- `fee_rules`: effective-dated fee configuration.
- `order_fee_lines`: immutable fee snapshot at checkout.
- `payment_transactions`: provider payment references and states.
- `refunds`: refund records.
- `seller_payouts`: payout records and provider references.
- `webhook_events`: provider event deduplication and processing status.

### Fulfilment
- `shipping_options`: seller/listing fulfilment configuration.
- `shipments`: carrier, label and tracking data.
- `pickup_appointments`: local pickup scheduling and confirmation.

### Subscription and promotion
- `subscription_plans`: Basic, Professional and Enterprise plan definitions.
- `seller_subscriptions`: provider-linked subscription status.
- `listing_promotions`: featured placement and promotion periods.
- `advertising_campaigns`: optional later-phase sponsored placements.

### Trust and operations
- `reports`: user/listing reports.
- `disputes`: order dispute cases.
- `dispute_messages` / `dispute_evidence`: restricted case data.
- `notifications`: notification records.
- `notification_preferences`: user channel preferences.
- `audit_logs`: privileged and sensitive actions.
- `outbox_events`: reliable event publication.
- `admin_settings`: controlled platform configuration.

## 3. Relationships

```text
users 1---1 user_profiles
users 1---0..1 seller_profiles
users 1---many user_addresses
users many---many roles (through user_roles)

users 1---many listings
categories 1---many listings
listings 1---0..1 auctions
listings 1---many listing_media
auctions 1---many bids
auctions 1---many proxy_bids
users 1---many bids
users 1---many watchlist_items
auctions 1---many watchlist_items

auctions 1---0..1 orders (winning result)
orders 1---many order_items
orders 1---many order_fee_lines
orders 1---many payment_transactions
payment_transactions 1---many refunds
orders 1---many shipments
orders 1---many disputes

seller_profiles 1---many seller_subscriptions
subscription_plans 1---many seller_subscriptions
listings 1---many listing_promotions

users 1---many notifications
users 1---many reports
users 1---many audit_logs (as actor)
```

## 4. Important constraints

- One listing should have at most one auction record for the MVP auction format.
- One accepted bid record must reference one auction and one bidder.
- Bid amount must be positive and currency must match auction currency.
- A bid cannot be accepted after the server-authoritative close time unless an approved extension rule extends the auction.
- A user cannot bid on their own listing unless the business explicitly permits it (recommended default: prohibit).
- Proxy maximums must never be exposed through public queries.
- One auction must not create more than one winning order.
- Provider webhook event IDs must be unique per provider.
- Payment provider transaction IDs must be unique where supplied.
- Refund total must not exceed captured amount, considering previous refunds.
- Seller subscription entitlements must derive from confirmed subscription state.
- User role assignment and privileged actions must be auditable.

## 5. Index recommendations

- `users(email)` unique, normalized.
- `listings(status, category_id, published_at)`.
- `listings(seller_id, status, created_at)`.
- `auctions(status, ends_at)`.
- `bids(auction_id, created_at DESC)`.
- `bids(auction_id, amount_minor DESC)`.
- `watchlist_items(user_id, auction_id)` unique.
- `orders(buyer_id, created_at DESC)`.
- `orders(seller_id, created_at DESC)`.
- `payment_transactions(provider, provider_payment_id)` unique where not null.
- `webhook_events(provider, provider_event_id)` unique.
- `shipments(order_id, created_at DESC)`.
- `notifications(user_id, read_at, created_at DESC)`.
- `audit_logs(actor_user_id, created_at DESC)` and target lookup indexes.

For large bid/event tables, evaluate partitioning and retention/archival only after measuring actual volume.

## 6. Data classification

| Classification | Examples | Access |
|---|---|---|
| Public | Published listing title, description, public seller display name | Public |
| Personal | Email, phone, profile details | Account owner and authorized staff |
| Sensitive personal | Shipping address, verification documents | Restricted service and assigned staff |
| Financial metadata | Provider IDs, payment status, payout status | Finance/admin services |
| Highly restricted | Secrets, tokens, raw identity documents | Never in normal logs; tightly limited access |

## 7. Migration strategy

- Create versioned migrations.
- Apply migrations through CI/CD with a rollback or forward-fix plan.
- Back up before risky production migrations.
- Prefer additive schema changes followed by backfill and later cleanup.
- Avoid long table locks during peak bidding.
- Test migration on production-like data volume.
- Seed only non-production demo data; never put real personal or payment data in seed fixtures.

## 8. PostgreSQL 16 and Laravel implementation notes

- Target PostgreSQL 16 for all environments, including local Docker, CI service containers and production managed databases.
- `gen_random_uuid()` is a core PostgreSQL function from version 13 onward, so the `pgcrypto` extension is not needed for UUIDs.
- Implement the schema as ordered Laravel migrations. Use `$table->uuid('id')->primary()` with the `HasUuids` trait on models.
- Store enum-like values as `string` columns with CHECK constraints rather than native PostgreSQL enum types. Adding a value to a native enum needs a special migration; a CHECK constraint is simpler to evolve.
- Use Laravel's `timestampTz()` column type for all event timestamps so they map to PostgreSQL `timestamptz`. Keep `created_at` and `updated_at` managed by Eloquent.
- Use `jsonb` columns for flexible data (`notifications.data`, `audit_logs.metadata`, `webhook_events.payload`) and index JSON keys only when queries require it.
- Concurrency-sensitive writes (bids, closing, payments) use `SELECT ... FOR UPDATE` through Eloquent's `lockForUpdate()` inside a database transaction.
- Full-text search (MVP) uses a generated `tsvector` column on `listings` with a GIN index.
- Use `CREATE INDEX CONCURRENTLY` for indexes added to large live tables.
