# Frontend Implementation Guide

**Confirmed web stack:** React.js with TypeScript, consuming the Laravel REST API.
**Confirmed mobile stack (Phase 2):** Flutter, consuming the same REST API.
**Open decisions:** public-page SEO rendering, styling approach (Tailwind CSS or other), PWA support. See document 17.

## 1. Frontend architecture

The web application is a React single-page application (SPA) built with Vite and TypeScript in strict mode.

- **Routing:** React Router, with route-level code splitting.
- **Server state:** TanStack Query for fetching, caching, retries and invalidation. Query keys are defined per feature.
- **Forms:** React Hook Form with schemas. Validation rules mirror the Laravel Form Requests where practical. The server is always the final authority.
- **API client:** a typed client generated from the OpenAPI document (section 4).
- **Real-time:** Laravel Echo subscribes to authorised Reverb channels for auction updates.
- **Auth:** Sanctum cookie-based session. CSRF token read from the `XSRF-TOKEN` cookie for mutating requests.
- **Public SEO pages:** decision required (document 17, section 6). Until decided, public listing and seller pages must be built so that they can be prerendered or server-rendered without rewriting components.

## 2. Suggested route map

```text
/                          Home
/categories/:slug          Category listing
/search                    Search results
/listings/:slug            Listing and auction detail (public)
/sellers/:storeSlug        Public seller profile
/auth/register
/auth/login
/auth/verify-email
/auth/forgot-password
/auth/reset-password
/account
/account/profile
/buyer/watchlist
/buyer/bids
/buyer/orders
/buyer/orders/:orderId
/seller/onboarding
/seller/dashboard
/seller/listings
/seller/listings/new
/seller/listings/:listingId/edit
/seller/orders
/seller/payouts
/seller/subscription
/admin                     Admin (separate layout, MFA required)
/admin/users
/admin/listings
/admin/reports
/admin/disputes
/admin/finance
/admin/audit
```

Route paths are proposals and may change with the final information architecture.

## 3. Suggested code organisation

```text
web/
  src/
    app/                    App shell, providers, router
    routes/                 Route components (public, auth, buyer, seller, admin)
    features/
      auth/
      catalogue/
      listings/
      auctions/             Countdown, bid form, bid history, live updates
      checkout/
      orders/
      shipping/
      subscriptions/
      notifications/
      disputes/
      admin/
    components/
      ui/                   Buttons, inputs, dialogs, toasts
      layout/               Header, navigation, footer
      feedback/             Loading, empty, error and permission-denied states
    api/
      generated/            Types and client generated from OpenAPI (do not edit)
      client.ts             Fetch wrapper, CSRF handling, error mapping
    realtime/               Echo setup and channel hooks
    lib/
      formatting/           Money (minor units to display), dates, time zones
      validation/           Shared form schemas
    hooks/
    styles/
    types/
  tests/
    unit/                   Vitest
    components/             React Testing Library
    e2e/                    Playwright
```

Rules:
- `features/` folders own their components, hooks and query definitions. Cross-feature imports go through the feature's public `index.ts`.
- `api/generated/` is produced by a script in CI. Pull requests that edit it by hand are rejected.
- No business rules in components. A component displays `auction.current_price` and sends a bid. It does not decide whether the bid is valid.

## 4. TypeScript and API contract

- `tsconfig` uses `strict: true`, `noUncheckedIndexedAccess: true` and `exactOptionalPropertyTypes: true`.
- Types for every request and response come from the OpenAPI document. A backend change that breaks a type fails `tsc --noEmit` in CI.
- Money values are typed as `Money = { amount_minor: number; currency: "CAD" }`. Formatting happens only in `lib/formatting`. Floating-point arithmetic is never used on money.

## 5. UI component requirements

Shared components:
- Header, navigation and footer.
- Search input and filter panel.
- Listing card and image gallery.
- Auction countdown, synchronised with server time.
- Current bid display and bid count.
- Bid form and proxy-bid form (proxy form hidden until approved).
- Bid history.
- Seller badge and profile summary.
- Price and fee breakdown.
- Status badge (text plus colour, never colour alone).
- Image uploader.
- Address form.
- Data table with cursor-based pagination.
- Confirmation dialog.
- Toast and inline error components.
- Loading, empty, error and permission-denied states.

## 6. Form handling

- Schema-based validation shared with the backend rules where practical.
- Client validation for usability. Server validation for security.
- Field-level errors mapped from Laravel's 422 `errors` object.
- Preserve draft input after recoverable errors (for example, in `sessionStorage` for the listing draft wizard).
- Never place sensitive values in URLs, query strings or analytics events.
- Explicit confirmation for irreversible actions (cancel auction, refund, suspend user).

## 7. Real-time UI

- Display the server-provided `ends_at` and synchronise the countdown with a server time offset returned by the API.
- Subscribe only to authorised auction channels. Unsubscribe when the component unmounts.
- Handle disconnect and reconnect. Refetch the auction snapshot after reconnecting.
- Do not show a bid as accepted until the server confirms it. Show distinct states: submitting, accepted, rejected.
- De-duplicate updates when a socket event and a query response contain the same bid ID.

## 8. Accessibility and responsive design

- Target WCAG 2.2 AA for core flows.
- Semantic HTML, correct labels and ARIA only where native elements are insufficient.
- Keyboard access for bidding, menus, dialogs and filters, with visible focus.
- Status is never conveyed by colour alone.
- Support reduced-motion preferences for countdown animations.
- Touch targets and form controls usable on mobile.
- Test at 200% zoom and on narrow viewports.
- Automated checks with axe in Playwright tests.

## 9. Frontend performance

- Images resized and optimised by the backend before delivery. Lazy-load below-the-fold media.
- Public media served from the CDN.
- Route-level code splitting. Admin code is not loaded on public pages.
- Monitor Core Web Vitals in production.
- Cache public catalogue queries carefully. Never cache private account, order or bid data in shared caches.

## 10. Frontend security

- No secret keys in the bundle. Only public configuration (API base URL, Stripe publishable key, Reverb app key) is exposed through environment variables prefixed for Vite (`VITE_`).
- Hidden buttons are not authorisation. Every action is checked by the API.
- Render user-generated content safely. React escapes text by default. Never use `dangerouslySetInnerHTML` with user content. Sanitise any rich text on the server.
- Restrictive Content-Security-Policy (see document 10).
- No personal information in browser analytics events.
- Authenticated pages are marked `Cache-Control: private` and `no-store` where appropriate.

## 11. Flutter mobile application (Phase 2)

- **Language and framework:** Dart and Flutter, one codebase for iOS and Android.
- **API:** the same REST API. Authentication uses Sanctum personal access tokens stored in platform secure storage (Keychain on iOS, Keystore on Android). Never in plain preferences.
- **Client:** a generated Dart client from the OpenAPI document.
- **Real-time:** Pusher-protocol client connected to Reverb, subscribed to the same private channels.
- **State management and navigation:** choose one standard before the mobile MVP starts (decision in document 17).
- **Push notifications:** Firebase Cloud Messaging and Apple Push Notification service, with device tokens stored per user and removed on logout.
- **Testing:** unit and widget tests with `flutter test`, integration tests with `integration_test`, and device testing on supported OS versions.
- **Release:** signed builds produced by GitHub Actions. App store submission follows the release checklist in document 13.
