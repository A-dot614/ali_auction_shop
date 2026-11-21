# Auction Engine and Business Rules

This document defines the implementation areas required for the auction engine. The business plan names timed auctions, proxy bidding, bid increments and snipe protection but does not fully specify their rules. Unconfirmed rules below are proposals requiring business-owner approval.

## 1. Auction lifecycle

Proposed states:
- `scheduled`: approved and awaiting start time.
- `active`: bidding is open.
- `extended`: bidding remains open after a configured extension.
- `closed_sold`: auction ended and a winner met the sale conditions.
- `closed_unsold`: auction ended without a qualifying sale.
- `cancelled`: auction was cancelled by authorized action.

Transitions must be validated by the backend. A seller must not be able to edit auction-critical terms after bidding begins, except through explicitly approved administrative procedures.

## 2. Timed auction rules

Required fields:
- Start time.
- End time.
- Starting price.
- Currency.
- Optional reserve price.
- Bid increment policy.
- Optional extension policy.

Use server UTC time. Display localized time to the user. Reject malformed schedules and end times that are not after start times.

## 3. Manual bidding

Proposed default:
- Bidder must be authenticated, active and eligible.
- Bidder cannot bid on their own listing.
- Auction must be active at the server's decision time.
- Bid must meet the next valid minimum amount.
- Accepted bid is immutable.
- Each request includes an idempotency key to prevent duplicate submissions.
- The auction row is locked or otherwise protected against concurrent writes.

**Decision required:** Whether the bid input represents the exact amount the user offers or the maximum amount they are willing to pay.

## 4. Bid increments

The source plan mentions bid increments but supplies no increment schedule.

Proposed configuration table:
- `lower_bound_minor`
- `upper_bound_minor`
- `increment_minor`
- optional `category_id`
- `effective_from`
- `effective_to`
- `is_active`

The platform owner must approve the increment schedule and precedence rules. Do not hardcode increments in frontend code.

## 5. Proxy bidding

The source glossary describes proxy bidding as the system incrementally bidding for a user up to their maximum.

A safe implementation must:
- Store maximum bid privately.
- Reveal only the public current price and accepted bid history.
- Define how ties are resolved.
- Define the minimum increment.
- Define what happens when a bidder changes their maximum.
- Ensure a bidder's maximum is never exposed in REST responses, logs, broadcast events or admin screens without explicit need.
- Process competing proxy bids deterministically under transaction control.

**Decision required:** Exact price calculation and tie-breaking algorithm. This must be documented with examples before coding.

## 6. Snipe protection

The source defines snipe protection as extending the auction when a bid arrives in the final moments.

Configuration required:
- Extension trigger window (e.g. final N seconds).
- Extension duration.
- Whether each qualifying bid resets the end time.
- Maximum number of extensions or no cap.
- Whether extension applies to all categories or selected auction types.
- Whether the UI labels an auction as extended.

No numerical values are specified by the business plan. Do not assume them without approval.

## 7. Reserve price

The source glossary defines reserve price as the minimum price a seller is willing to accept.

Required decisions:
- Is reserve price optional?
- Is reserve met status public or private?
- Does a below-reserve high bid create an order?
- Can a seller lower a reserve after bidding starts?
- Is reserve price included in the public listing?

Recommended default: reserve is immutable after the first accepted bid; an auction below reserve closes unsold and creates no payable order. This recommendation requires approval.

## 8. Auction close and winner determination

Close process should be:
1. Acquire a concurrency-safe lock for the auction.
2. Confirm auction is not already closed.
3. Confirm end time has passed and no valid extension applies.
4. Resolve all pending proxy-bid actions.
5. Determine the highest accepted bid using approved tie-breaking rules.
6. Evaluate reserve price, if applicable.
7. Persist final auction state and winning bid.
8. Create one order if sold.
9. Write an outbox event in the same database transaction.
10. Notify winner, seller and relevant watchers asynchronously.

Closing must be idempotent. Re-running the close job must not create a second order or duplicate charges.

## 9. Buyer premium and fee calculation

The business plan lists an 8% buyer's premium/transaction commission on final sale price, but does not define whether this is charged to buyer, seller or split, and the revenue projection arithmetic needs reconciliation.

Before implementation, approve:
- Fee payer.
- Fee base (hammer price, shipping, taxes or another base).
- Fee rate and category exceptions.
- Minimum/maximum fee.
- Tax treatment.
- Refund treatment.
- Whether fee rules are snapshotted at auction/order creation.

Never rely on a frontend-calculated total. Store a fee snapshot against the order.

## 10. Seller cancellation and exceptional cases

Define policies for:
- Seller requests cancellation after bids.
- Item becomes unavailable.
- Suspected fraud or prohibited item.
- Winning bidder does not pay.
- Payment authorization fails.
- Buyer disputes item condition.
- Carrier loses or damages item.
- Chargeback after seller payout.

Every exceptional transition needs an authorized actor, reason, audit event and financial consequence.

## 11. Test matrix

Minimum tests:
- Bid at start boundary.
- Bid exactly at close boundary.
- Bid after close.
- Two bids submitted simultaneously.
- Same request retried with same idempotency key.
- Same request retried with a different key.
- Seller attempts to bid on own listing.
- User is suspended during an active auction.
- Proxy bids tie.
- Proxy maximum changed during competition.
- Bid triggers extension.
- Multiple bids during extension.
- Reserve is met / not met.
- Close job runs twice.
- Payment creation and webhook arrive more than once.
- Auction cancellation with existing bids.
