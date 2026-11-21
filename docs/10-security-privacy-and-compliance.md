# Security, Privacy and Compliance Plan

**Important:** This is an engineering control checklist, not legal advice or a certification. Obtain qualified Canadian legal, privacy, tax, payment and security review.

## 1. Source-plan commitments

The business plan states the following intentions:
- PIPEDA compliance and Canadian data residency.
- AWS Canada `ca-central-1`.
- TLS across endpoints and HSTS.
- AES-256 encryption at rest and TLS 1.3 in transit.
- Two-factor authentication available to all users and mandatory for sellers.
- PCI-DSS Level 1 compliance via Stripe, with no raw card data stored on the platform.
- SOC 2 Type II targeted by end of Year 2.
- Annual third-party penetration testing and a Year 2 bug bounty program.

These are targets and proposed controls, not evidence of current compliance.

## 2. Identity and access security

- Use a modern password hashing algorithm. Laravel's `Hash` facade uses bcrypt by default; configure Argon2id with approved parameters if the security review requires it.
- Require email verification.
- Enforce MFA for staff and according to the approved seller policy.
- Secure session cookies (`HttpOnly`, `Secure`, `SameSite=Lax`) for the React SPA through Laravel Sanctum. Personal access tokens for Flutter are stored in platform secure storage and can be revoked individually.
- Rotate/revoke sessions after sensitive account events.
- Rate-limit authentication and recovery flows.
- Protect against credential stuffing and account enumeration.
- Enforce server-side RBAC and object-level authorization.

## 3. Application security

- Validate all inputs on the server with Form Request classes. Protect against mass assignment with explicit `$fillable` lists or `forceFill` in reviewed code only.
- Use parameterised queries through Eloquent and the query builder. Never interpolate user input into raw SQL. Keep raw SQL in reviewed, bound-parameter form only.
- Apply CSRF protections where cookie sessions are used (Sanctum CSRF cookie and `X-XSRF-TOKEN` header).
- Apply CSP, HSTS and secure headers.
- Protect file uploads with size/type validation (Laravel `mimes` and `max` rules), malware scanning and private S3 bucket policies with pre-signed URLs.
- Prevent IDOR/BOLA by checking ownership and permissions on every object request.
- Protect the REST API with rate limits per user and per IP (Laravel `RateLimiter` on Redis), request size limits and pagination maximums.
- Use rate limits for bidding, login, registration, reports and checkout.
- Perform dependency scanning (`composer audit`, `npm audit`, Dependabot), secret scanning and push protection (GitHub), and static analysis (Larastan, CodeQL) in GitHub Actions.
- Maintain a vulnerability remediation process.

## 4. Payment security boundaries

- Use provider-hosted/tokenized payment collection.
- Do not store raw card numbers or CVV.
- Verify webhook signatures.
- Use idempotency keys.
- Reconcile provider events with internal ledgers.
- Restrict finance operations.
- Store only necessary provider references and transaction metadata.
- Review the actual PCI scope with the payment provider and a qualified assessor.

## 5. Privacy engineering

Before launch, define:
- Personal information inventory and data-flow map.
- Purposes and legal basis/consent model for each collection point.
- Privacy notice and user-facing disclosures.
- Access, correction and deletion request workflow.
- Retention schedule for accounts, bids, orders, disputes, tax/financial records and audit logs.
- Data processor/vendor register and contracts.
- Cross-border access and data residency assessment.
- Incident/breach assessment and notification procedure.
- Privacy impact assessment where appropriate.
- Children's access policy and age-related requirements.
- Cookie/analytics consent approach where applicable.

Do not assume Canadian hosting alone establishes compliance with every federal or provincial privacy requirement.

## 6. Auction and marketplace safety

- Prohibited/restricted items policy.
- Seller verification policy.
- Counterfeit and authenticity reporting.
- Bid manipulation/shill bidding detection.
- Seller/buyer suspension and appeals process.
- Dispute and refund policy.
- Chargeback handling.
- Evidence retention.
- Transparent auction terms and fee disclosures.

## 7. Operational security

- Separate development, staging and production.
- Use managed secrets; never commit `.env` secrets.
- Restrict production access and review it periodically.
- Encrypt backups and test restores. Encrypt sensitive model fields with Laravel's `encrypted` cast and protect `APP_KEY` as a secret, with a documented rotation procedure.
- Monitor privileged access and unusual bidding/payment patterns.
- Maintain incident response contacts and runbooks.
- Conduct threat modelling for bidding, payments, account takeover and admin abuse.
- Perform penetration tests before public launch and periodically thereafter.

## 8. Compliance workstream requiring expert review

- Federal and provincial privacy laws.
- Consumer protection and online marketplace disclosures.
- Auction-specific legal requirements in relevant provinces.
- Sales tax / GST/HST/PST/QST collection and reporting responsibilities.
- Marketplace facilitator and seller reporting obligations, if applicable.
- Payment services and marketplace payout model.
- Escrow terminology and whether the proposed funds-holding arrangement is legally and operationally permissible.
- Accessibility obligations.
- Records retention and financial reporting.
- Terms of service, seller agreement, buyer agreement, privacy policy and prohibited-items policy.

## 9. Security release gate

No production launch until:
- No unresolved critical/high security findings.
- Access-control tests pass.
- Payment provider integration and webhook validation pass.
- Secrets and personal data are absent from logs.
- Backup restoration is tested.
- Incident response process is documented.
- Privacy/legal review is signed off.
