# Roles, Permissions and Admin Controls

The business plan names buyer/seller trust, seller verification, support tiers and platform operations but does not define a complete role-based access control (RBAC) model. This is a proposed baseline.

## 1. Roles

- Guest
- Buyer
- Individual Seller
- Professional Seller
- Enterprise Seller
- Support Agent
- Moderator / Trust & Safety
- Finance Operator
- Administrator
- Super Administrator

A user may be both buyer and seller. Role assignment should not imply that every seller is verified.

## 2. Permission principles

- Deny by default.
- Enforce permissions on the server.
- Separate business role from account status and verification status.
- Apply object-level checks: users may access only their own private records unless a specific staff permission grants access.
- Use MFA for staff and require stronger authentication for privileged operations.
- Log role changes and sensitive actions.
- Avoid shared admin accounts.

## 3. Permission matrix

| Capability | Buyer | Seller | Support | Moderator | Finance | Admin |
|---|---:|---:|---:|---:|---:|---:|
| Browse public listings | Yes | Yes | Yes | Yes | Yes | Yes |
| Place bids | Yes | Yes | No by default | No by default | No | No |
| Manage own profile | Yes | Yes | Own only | Own only | Own only | Yes |
| Create/manage own listings | No | Yes | No | No | No | Yes |
| View own orders | Yes | Yes | Assigned cases | Assigned cases | Finance scope | Yes |
| Moderate listings | No | No | No | Yes | No | Yes |
| Review seller verification | No | No | Limited | Yes | No | Yes |
| View dispute case | Own case | Own case | Assigned | Assigned | Financial details only | Yes |
| Issue refund | No | No | No by default | No | Yes, subject to policy | Yes |
| View payout records | Own | Own | No | No | Yes | Yes |
| Manage fee configuration | No | No | No | No | Propose/review | Yes |
| Manage roles | No | No | No | No | No | Admin / Super Admin |
| View audit logs | No | No | Limited | Limited | Finance scope | Yes |

The table is intentionally high-level. Implement fine-grained permission codes such as `listing.moderate`, `seller.verify`, `payment.refund`, `fee_rule.update`, `audit.read`.

## 4. Staff controls

- Staff accounts must use MFA.
- Privileged role changes require a reason and audit record.
- Refunds above an approved threshold may require dual approval.
- Verification documents should be visible only to assigned reviewers.
- Support staff should see masked personal/financial data unless access is needed for a case.
- Admin actions should use a separate admin route and session policy.
- High-risk actions should require recent re-authentication.

## 5. Audit events

Record:
- Login security events and MFA changes.
- Role/permission changes.
- Seller verification decisions.
- Listing publish/reject/pause/remove actions.
- Auction cancellation and manual close.
- Fee and subscription configuration changes.
- Refund and payout adjustments.
- Dispute resolution.
- User suspension/reactivation.
- Access to sensitive documents where required.

Audit record fields:
- Actor ID.
- Action code.
- Target type and ID.
- Timestamp.
- Reason.
- Request/correlation ID.
- Safe metadata.
- Source IP/device metadata only where lawful and necessary.

Never log passwords, session tokens, full payment card data or unnecessary identity document contents.
