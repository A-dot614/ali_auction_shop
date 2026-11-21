# PostgreSQL Starter Schema

This is a starter DDL blueprint for core MVP entities, written for PostgreSQL 16, not a complete production migration. Review naming, constraints, privacy, indexing, retention and provider requirements with the engineering team. Implement it as ordered Laravel migrations (see `04-database-design.md`, section 8) before any production use.

```sql
-- PostgreSQL 16: gen_random_uuid() is built in; no extension required.

CREATE TYPE account_status AS ENUM ('pending_verification', 'active', 'suspended', 'closed');
CREATE TYPE listing_status AS ENUM ('draft', 'pending_review', 'published', 'rejected', 'paused', 'sold', 'expired', 'removed');
CREATE TYPE auction_status AS ENUM ('scheduled', 'active', 'extended', 'closed_sold', 'closed_unsold', 'cancelled');
CREATE TYPE order_status AS ENUM ('awaiting_payment', 'paid', 'processing', 'shipped', 'ready_for_pickup', 'completed', 'cancelled', 'disputed', 'refunded');
CREATE TYPE payment_status AS ENUM ('created', 'requires_action', 'authorized', 'captured', 'failed', 'cancelled', 'partially_refunded', 'refunded', 'disputed');

CREATE TABLE users (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    email text NOT NULL,
    password_hash text NOT NULL,
    status account_status NOT NULL DEFAULT 'pending_verification',
    email_verified_at timestamptz,
    mfa_enabled boolean NOT NULL DEFAULT false,
    created_at timestamptz NOT NULL DEFAULT now(),
    updated_at timestamptz NOT NULL DEFAULT now()
);
CREATE UNIQUE INDEX users_email_lower_uq ON users (lower(email));

CREATE TABLE user_profiles (
    user_id uuid PRIMARY KEY REFERENCES users(id),
    display_name text NOT NULL,
    avatar_key text,
    bio text,
    province_code text,
    created_at timestamptz NOT NULL DEFAULT now(),
    updated_at timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE seller_profiles (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id uuid NOT NULL UNIQUE REFERENCES users(id),
    seller_type text NOT NULL CHECK (seller_type IN ('individual', 'business', 'institutional')),
    legal_name text,
    public_store_name text,
    verification_status text NOT NULL DEFAULT 'not_submitted',
    payout_provider_account_id text,
    created_at timestamptz NOT NULL DEFAULT now(),
    updated_at timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE categories (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    parent_id uuid REFERENCES categories(id),
    name text NOT NULL,
    slug text NOT NULL UNIQUE,
    is_active boolean NOT NULL DEFAULT true,
    sort_order integer NOT NULL DEFAULT 0,
    created_at timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE listings (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    seller_id uuid NOT NULL REFERENCES seller_profiles(id),
    category_id uuid NOT NULL REFERENCES categories(id),
    title text NOT NULL,
    slug text NOT NULL UNIQUE,
    description text NOT NULL,
    condition text,
    status listing_status NOT NULL DEFAULT 'draft',
    currency char(3) NOT NULL DEFAULT 'CAD',
    location_city text,
    location_province text,
    location_postal_prefix text,
    published_at timestamptz,
    created_at timestamptz NOT NULL DEFAULT now(),
    updated_at timestamptz NOT NULL DEFAULT now()
);
CREATE INDEX listings_category_status_idx ON listings(category_id, status, published_at DESC);
CREATE INDEX listings_seller_status_idx ON listings(seller_id, status, created_at DESC);

CREATE TABLE listing_media (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    listing_id uuid NOT NULL REFERENCES listings(id) ON DELETE CASCADE,
    storage_key text NOT NULL,
    media_type text NOT NULL DEFAULT 'image',
    alt_text text,
    sort_order integer NOT NULL DEFAULT 0,
    created_at timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE auctions (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    listing_id uuid NOT NULL UNIQUE REFERENCES listings(id),
    status auction_status NOT NULL DEFAULT 'scheduled',
    currency char(3) NOT NULL DEFAULT 'CAD',
    starting_price_minor bigint NOT NULL CHECK (starting_price_minor > 0),
    reserve_price_minor bigint CHECK (reserve_price_minor IS NULL OR reserve_price_minor > 0),
    current_price_minor bigint NOT NULL CHECK (current_price_minor >= 0),
    starts_at timestamptz NOT NULL,
    ends_at timestamptz NOT NULL,
    extension_count integer NOT NULL DEFAULT 0 CHECK (extension_count >= 0),
    winning_bid_id uuid,
    closed_at timestamptz,
    created_at timestamptz NOT NULL DEFAULT now(),
    updated_at timestamptz NOT NULL DEFAULT now(),
    CHECK (ends_at > starts_at)
);
CREATE INDEX auctions_status_ends_idx ON auctions(status, ends_at);

CREATE TABLE bids (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    auction_id uuid NOT NULL REFERENCES auctions(id),
    bidder_user_id uuid NOT NULL REFERENCES users(id),
    amount_minor bigint NOT NULL CHECK (amount_minor > 0),
    currency char(3) NOT NULL DEFAULT 'CAD',
    bid_type text NOT NULL CHECK (bid_type IN ('manual', 'proxy')),
    accepted_at timestamptz NOT NULL DEFAULT now(),
    request_id uuid NOT NULL UNIQUE
);
CREATE INDEX bids_auction_time_idx ON bids(auction_id, accepted_at DESC);
CREATE INDEX bids_auction_amount_idx ON bids(auction_id, amount_minor DESC);

CREATE TABLE proxy_bids (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    auction_id uuid NOT NULL REFERENCES auctions(id),
    bidder_user_id uuid NOT NULL REFERENCES users(id),
    max_amount_minor bigint NOT NULL CHECK (max_amount_minor > 0),
    currency char(3) NOT NULL DEFAULT 'CAD',
    is_active boolean NOT NULL DEFAULT true,
    created_at timestamptz NOT NULL DEFAULT now(),
    updated_at timestamptz NOT NULL DEFAULT now(),
    UNIQUE (auction_id, bidder_user_id)
);

ALTER TABLE auctions
    ADD CONSTRAINT auctions_winning_bid_fk
    FOREIGN KEY (winning_bid_id) REFERENCES bids(id);

CREATE TABLE watchlist_items (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id uuid NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    auction_id uuid NOT NULL REFERENCES auctions(id) ON DELETE CASCADE,
    created_at timestamptz NOT NULL DEFAULT now(),
    UNIQUE (user_id, auction_id)
);

CREATE TABLE orders (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    auction_id uuid NOT NULL UNIQUE REFERENCES auctions(id),
    buyer_user_id uuid NOT NULL REFERENCES users(id),
    seller_id uuid NOT NULL REFERENCES seller_profiles(id),
    status order_status NOT NULL DEFAULT 'awaiting_payment',
    currency char(3) NOT NULL DEFAULT 'CAD',
    hammer_price_minor bigint NOT NULL CHECK (hammer_price_minor >= 0),
    subtotal_minor bigint NOT NULL CHECK (subtotal_minor >= 0),
    total_minor bigint NOT NULL CHECK (total_minor >= 0),
    shipping_address_snapshot jsonb,
    created_at timestamptz NOT NULL DEFAULT now(),
    updated_at timestamptz NOT NULL DEFAULT now()
);
CREATE INDEX orders_buyer_created_idx ON orders(buyer_user_id, created_at DESC);
CREATE INDEX orders_seller_created_idx ON orders(seller_id, created_at DESC);

CREATE TABLE order_fee_lines (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id uuid NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    fee_code text NOT NULL,
    description text NOT NULL,
    amount_minor bigint NOT NULL,
    currency char(3) NOT NULL,
    created_at timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE payment_transactions (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id uuid NOT NULL REFERENCES orders(id),
    provider text NOT NULL,
    provider_payment_id text,
    status payment_status NOT NULL DEFAULT 'created',
    amount_minor bigint NOT NULL CHECK (amount_minor > 0),
    currency char(3) NOT NULL DEFAULT 'CAD',
    idempotency_key text NOT NULL UNIQUE,
    captured_at timestamptz,
    created_at timestamptz NOT NULL DEFAULT now(),
    updated_at timestamptz NOT NULL DEFAULT now()
);
CREATE UNIQUE INDEX payment_provider_reference_uq
    ON payment_transactions(provider, provider_payment_id)
    WHERE provider_payment_id IS NOT NULL;

CREATE TABLE webhook_events (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    provider text NOT NULL,
    provider_event_id text NOT NULL,
    event_type text NOT NULL,
    payload jsonb NOT NULL,
    processing_status text NOT NULL DEFAULT 'received',
    received_at timestamptz NOT NULL DEFAULT now(),
    processed_at timestamptz,
    UNIQUE (provider, provider_event_id)
);

CREATE TABLE refunds (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    payment_transaction_id uuid NOT NULL REFERENCES payment_transactions(id),
    provider_refund_id text,
    amount_minor bigint NOT NULL CHECK (amount_minor > 0),
    currency char(3) NOT NULL,
    reason text NOT NULL,
    status text NOT NULL,
    created_by uuid REFERENCES users(id),
    created_at timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE shipments (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id uuid NOT NULL REFERENCES orders(id),
    carrier text NOT NULL,
    tracking_number text,
    label_storage_key text,
    status text NOT NULL DEFAULT 'pending',
    shipped_at timestamptz,
    delivered_at timestamptz,
    created_at timestamptz NOT NULL DEFAULT now(),
    updated_at timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE disputes (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id uuid NOT NULL REFERENCES orders(id),
    opened_by uuid NOT NULL REFERENCES users(id),
    reason_code text NOT NULL,
    description text NOT NULL,
    status text NOT NULL DEFAULT 'open',
    resolution text,
    resolved_by uuid REFERENCES users(id),
    resolved_at timestamptz,
    created_at timestamptz NOT NULL DEFAULT now(),
    updated_at timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE notifications (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id uuid NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    notification_type text NOT NULL,
    title text NOT NULL,
    body text NOT NULL,
    data jsonb NOT NULL DEFAULT '{}'::jsonb,
    read_at timestamptz,
    created_at timestamptz NOT NULL DEFAULT now()
);
CREATE INDEX notifications_user_unread_idx ON notifications(user_id, created_at DESC) WHERE read_at IS NULL;

CREATE TABLE audit_logs (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    actor_user_id uuid REFERENCES users(id),
    action text NOT NULL,
    target_type text NOT NULL,
    target_id uuid,
    reason text,
    metadata jsonb NOT NULL DEFAULT '{}'::jsonb,
    created_at timestamptz NOT NULL DEFAULT now()
);
CREATE INDEX audit_logs_target_idx ON audit_logs(target_type, target_id, created_at DESC);

CREATE TABLE outbox_events (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    aggregate_type text NOT NULL,
    aggregate_id uuid NOT NULL,
    event_type text NOT NULL,
    payload jsonb NOT NULL,
    created_at timestamptz NOT NULL DEFAULT now(),
    published_at timestamptz,
    attempts integer NOT NULL DEFAULT 0
);
CREATE INDEX outbox_unpublished_idx ON outbox_events(created_at) WHERE published_at IS NULL;
```

## Implementation notes

- Laravel mapping: convert each `CREATE TYPE ... AS ENUM` to a `string` column with a CHECK constraint, and each table to a migration with a matching Eloquent model using the `HasUuids` trait.
- This schema omits several supporting tables from the full logical model: roles/permissions, seller verification, subscriptions, fee rules, shipping options, pickup appointments, listing moderation, reports, dispute evidence, notification preferences, promotions, bid increments and scheduled jobs.
- `shipping_address_snapshot` is sensitive personal data. Restrict access and define retention/deletion rules.
- JSONB webhook payloads may contain personal or payment metadata. Minimize stored fields, encrypt where appropriate and define retention.
- Add database-level protection and service logic for one winner per auction, reserve evaluation, bid minimums, seller self-bidding prohibition, refund totals and auction transitions.
- Do not treat this as a copy/paste production schema without review.
