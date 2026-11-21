# User Flows and Screen Inventory

This document is a functional UX blueprint, not a visual design system. Detailed wireframes and UI mockups are referenced by the source business plan in a separate companion document that was not included in the supplied PDF.

## 1. Public visitor journey

```text
Home
  -> Browse category / Search
  -> Listing detail
  -> Register or sign in
  -> Return to listing
  -> Place bid / Add to watchlist
```

Screens:
- Homepage.
- Category landing page.
- Search results.
- Listing detail.
- Sign up.
- Sign in.
- Email verification.
- Password reset.
- Public seller profile.
- Help / FAQ.
- Terms, privacy and auction policy.

## 2. Buyer journey

```text
Sign in
  -> Discover auction
  -> Review listing, seller and fulfilment details
  -> Place bid / set proxy maximum
  -> Receive outbid or auction result
  -> If winner: review order total
  -> Choose shipping or pickup
  -> Pay
  -> Track shipment / confirm pickup
  -> Complete or open dispute
```

Buyer screens:
- Buyer dashboard.
- Watchlist.
- Bid history.
- Notifications.
- Won auctions.
- Checkout.
- Payment result.
- Order detail.
- Shipment tracking.
- Dispute centre.
- Profile and addresses.
- Notification preferences.

## 3. Seller journey

```text
Register
  -> Verify email
  -> Create seller profile
  -> Complete required verification
  -> Create listing draft
  -> Upload photos and item details
  -> Configure auction and fulfilment
  -> Submit for review
  -> Publish
  -> Monitor bids
  -> Auction closes
  -> Review order
  -> Fulfil shipment or pickup
  -> Track payout
```

Seller screens:
- Seller onboarding.
- Verification status.
- Seller dashboard.
- Listing list.
- Create/edit listing wizard.
- Image manager.
- Auction configuration.
- Orders and fulfilment.
- Shipping label screen.
- Payout history.
- Subscription plans and billing.
- Store profile / branding.
- Analytics (Phase 2).

## 4. Admin and support journeys

Admin screens:
- Admin login with MFA.
- Overview dashboard.
- User search and account detail.
- Seller verification queue.
- Listing moderation queue.
- Listing detail / moderation actions.
- Auction monitoring.
- Reports queue.
- Dispute queue and case detail.
- Orders and payment status.
- Refund review.
- Subscription management.
- Categories and attributes.
- Fee configuration.
- Notification templates.
- Audit log.
- System health / job failures.

Support screens:
- Assigned tickets.
- User/order context with restricted fields.
- Dispute messages and evidence.
- Resolution actions available to the support role.

## 5. Navigation model

Public navigation:
- Home
- Browse categories
- Search
- How it works
- Sign in / Register

Authenticated buyer navigation:
- Discover
- Watchlist
- My bids
- Won items / Orders
- Notifications
- Account

Seller navigation:
- Dashboard
- Listings
- Orders
- Shipping
- Payouts
- Store settings
- Subscription

Admin navigation must be separate from consumer navigation and protected by explicit role permissions.

## 6. Responsive requirements

- Mobile-first layouts.
- Bidding controls must remain usable on small screens.
- Countdown and current bid must be readable without relying on colour alone.
- Tables should become cards or horizontally scroll only when appropriate.
- Forms should have clear labels, validation and error recovery.
- Admin interfaces may be desktop-optimized but must remain usable on tablet.
- Support keyboard navigation and screen readers.

## 7. UX states required for each major screen

Every data-driven screen should define:
- Loading state.
- Empty state.
- Error state.
- Permission-denied state.
- Offline/reconnecting state where real-time updates apply.
- Success confirmation.
- Retry behaviour.
- Stale-data handling.

## 8. Wireframe dependency

The business plan references **AliAuction.ca — Website Template & Design Specifications** as a companion wireframe document. That file was not supplied in the 25-page business plan PDF. The UX team should obtain it before treating any layout or screen design as approved.
