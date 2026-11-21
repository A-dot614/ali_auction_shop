# Epics and User Stories — AliAuction.ca

This backlog converts the PRD into user-centred work items. Story priority is provisional and must be confirmed during backlog refinement.

## Epic A — Account and identity

### US-A01: Register
As a visitor, I want to create an account so that I can bid or sell.

Acceptance criteria:
- Given valid email and password, when I submit registration, an account is created in an unverified state.
- A verification email is sent.
- Duplicate email registration is rejected without exposing unnecessary account information.
- Rate limits protect registration endpoints.

### US-A02: Verify email
As a registered user, I want to verify my email so that the platform can confirm account ownership.

Acceptance criteria:
- Verification tokens are single-use and expire.
- A valid token marks the email verified.
- Invalid, expired or reused tokens return a safe error.

### US-A03: Sign in and reset password
As a user, I want secure access and password recovery.

Acceptance criteria:
- Valid credentials create a session.
- Invalid credentials return a generic error.
- Password reset tokens expire and are single-use.
- Sessions can be revoked after password reset or account suspension.

## Epic B — Seller onboarding and listings

### US-B01: Create seller profile
As a buyer, I want to become a seller so that I can list items.

Acceptance criteria:
- User can submit seller profile details.
- Seller status and identity verification status are distinct.
- Restricted seller actions remain unavailable until required checks pass.
- The system records who reviewed verification and when.

### US-B02: Create listing draft
As a seller, I want to save a listing draft so that I can complete it later.

Acceptance criteria:
- Required fields are validated.
- Drafts are visible only to the owner and authorized staff.
- Draft edits do not alter an already-running auction.

### US-B03: Publish auction listing
As an eligible seller, I want to publish an auction so that buyers can bid.

Acceptance criteria:
- Listing has a valid category, title, description, images, condition and fulfilment option.
- Auction start time is before end time.
- Starting bid and currency are valid.
- The listing is not public until moderation/approval rules are satisfied.
- Published listing displays the approved auction terms.

## Epic C — Discovery

### US-C01: Browse categories
As a buyer, I want to browse categories so that I can discover relevant auctions.

Acceptance criteria:
- Category pages show active listings.
- Empty categories have a useful empty state.
- Pagination or cursor-based loading avoids loading the full catalogue.

### US-C02: Search and filter
As a buyer, I want to search and filter listings.

Acceptance criteria:
- Search supports keyword and category filters.
- Filters include only fields supported by listing data.
- Sorting options are clearly labelled.
- Search results exclude hidden, expired or unpublished listings by default.

### US-C03: View auction details
As a buyer, I want to see complete auction information before bidding.

Acceptance criteria:
- Page shows current price, bid count, end time, seller information and item details.
- Shipping/pickup availability and fee disclosures are visible.
- Bid history does not expose private proxy maximums.

## Epic D — Bidding and auction lifecycle

### US-D01: Place bid
As a buyer, I want to place a bid so that I can compete for an item.

Acceptance criteria:
- Only authenticated and eligible users can bid.
- Auction must be open according to server time.
- Bid must meet the minimum valid amount.
- Concurrent bids are serialized or otherwise handled transactionally.
- Accepted bids are immutable and timestamped by the server.
- Rejected bids do not change auction state.

### US-D02: Proxy bid
As a buyer, I want to set a maximum bid so that the system can bid incrementally for me.

Acceptance criteria:
- Maximum is private to the bidder and authorized services.
- Public current price follows approved proxy-bidding rules.
- A user cannot retrieve another bidder's maximum.
- Proxy-bid conflicts are deterministic and covered by tests.

### US-D03: Close auction
As the platform, I want to close auctions reliably so that winners and outcomes are determined consistently.

Acceptance criteria:
- Closing is idempotent.
- Only one final outcome is created.
- Reserve-price logic is applied if enabled.
- Winner and final price are recorded.
- A winning order is created once.
- Notifications are queued after successful commit.

### US-D04: Watch auction
As a buyer, I want to watch an auction so that I can return to it.

Acceptance criteria:
- User can add/remove an auction from watchlist.
- Duplicate watch entries are prevented.
- Watchlist is private to the user.

## Epic E — Checkout and payments

### US-E01: Review checkout
As a winning buyer, I want to review my total before paying.

Acceptance criteria:
- Checkout displays item amount, buyer premium/fees, shipping/tax where applicable and total.
- Fee calculations come from server-side configuration.
- The buyer must accept the displayed total before payment.

### US-E02: Pay for order
As a buyer, I want to pay securely.

Acceptance criteria:
- Payment is created with the configured provider.
- Raw card details are not stored by AliAuction.
- Provider webhook signature is verified.
- Duplicate webhook delivery does not duplicate payment/order transitions.
- Failure and cancellation states are visible.

### US-E03: Refund
As an authorized staff member, I want to process an approved refund.

Acceptance criteria:
- Refund amount cannot exceed refundable captured amount.
- Refund reason and actor are recorded.
- Provider result is reconciled.
- Buyer and seller receive appropriate notifications.

## Epic F — Shipping and pickup

### US-F01: Choose fulfilment
As a buyer, I want to choose an available shipping or pickup method.

Acceptance criteria:
- Only seller-enabled fulfilment options are shown.
- Shipping estimates use item package data and destination postal code.
- Pickup instructions are shown only to authorized order participants.

### US-F02: Track shipment
As a buyer, I want to see shipment status.

Acceptance criteria:
- Tracking number and carrier are displayed.
- Provider updates are timestamped.
- Manual status changes are audited.

## Epic G — Seller tools and subscriptions

### US-G01: View seller dashboard
As a seller, I want to see my listings, bids and sales.

Acceptance criteria:
- Seller can see only their own seller data.
- Metrics are based on documented definitions.
- Dashboard data can be paginated or summarized.

### US-G02: Upgrade subscription
As a seller, I want to subscribe to a paid plan to access higher limits and features.

Acceptance criteria:
- Plan price and billing interval are disclosed.
- Provider subscription state is reconciled by webhook.
- Entitlements change only after confirmed provider state.
- Cancellation and failed renewal states are handled.

## Epic H — Trust and administration

### US-H01: Report listing
As a user, I want to report suspicious or prohibited content.

Acceptance criteria:
- Report category and optional details are submitted.
- Reporter identity is protected from public display.
- Report enters a review queue.

### US-H02: Moderate listing
As a moderator, I want to hide or reject listings that violate policy.

Acceptance criteria:
- Only authorized staff can moderate.
- Reason is mandatory.
- Action is audited.
- Seller receives a notification where policy allows.

### US-H03: Resolve dispute
As a support agent, I want to review a transaction dispute.

Acceptance criteria:
- Dispute links to an order.
- Evidence and messages are access-controlled.
- Resolution decision, reason and financial action are recorded.
- Dispute status changes are auditable.

## Epic I — Notifications

### US-I01: Receive auction notifications
As a buyer, I want notifications when I am outbid, win or lose an auction.

Acceptance criteria:
- Events are generated from committed state changes.
- User notification preferences are respected, except legally/operationally required messages.
- Delivery attempts and failures are recorded.
- Notification links require authentication where appropriate.

## Epic J — Platform operations

### US-J01: Audit admin actions
As a platform owner, I want a record of privileged actions so that activity can be reviewed.

Acceptance criteria:
- Actor, action, target, timestamp and relevant before/after metadata are recorded.
- Audit events cannot be edited through ordinary admin UI.
- Sensitive values such as passwords, tokens and full payment details are never logged.
