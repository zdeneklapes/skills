# Security And Access Reference

Read this when documenting authentication, authorization, ownership, access control, user-generated content, paid access, sharing, billing trust boundaries, private data, or retention.

## Security, Authorization, Permissions, And Data Access

Security must be part of the requirements, not an afterthought.

Always distinguish:

* Authentication: who the user is.
* Authorization: what the user is allowed to do.
* Ownership: which objects belong to the user.
* Access: which paid, private, shared, or restricted content the user may use.

Core rules:

* Backend API authorization is mandatory.
* Frontend guards are usability helpers, not security boundaries.
* Every protected endpoint must check authentication.
* Every user-owned object must check ownership.
* Every restricted feature must check permissions server-side.
* Paid or locked content must not be returned by the API to unauthorized users.
* Fail closed when permissions are unclear.
* Use consistent denial behavior.

Define roles and permission levels.

Recommended role table:

```markdown
| Role | Purpose | Allowed actions | Restrictions |
| --- | --- | --- | --- |
| ... | ... | ... | ... |
```

Define data access rules for:

* Account data.
* Billing data.
* User-created content.
* Progress/history data.
* Shared objects.
* Admin-only objects.
* Moderation data.
* Reports and analytics.

## User-Generated Content

Any feature that accepts user text needs explicit constraints.

Apply this to:

* Notes.
* Comments.
* Support messages.
* Reports.
* Feedback.
* Shared content.
* Admin notes.

Document:

* Who can create it.
* Who can read it.
* Who can edit or delete it.
* Whether it is private, shared, public, or moderated.
* How it is validated.
* How it is rendered safely.
* Whether rate limits apply.
* Whether abuse reporting or moderation exists.
* How long it is retained.

Rules:

* Never assume user-generated text is safe.
* Do not allow private text to become shared accidentally.
* Do not log sensitive user-generated content unless explicitly required and protected.

## Paid Access And Billing

For paid applications, document access rules and payment trust boundaries.

Include:

* Free access rules.
* Paid access rules.
* Time windows.
* Expired access behavior.
* Refund or revocation behavior if supported.
* Plan-to-permission mapping.
* Checkout creation rules.
* Purchase fulfillment rules.
* Webhook or external payment confirmation rules.
* User ownership of orders, invoices, and payment history.

Rules:

* Do not trust browser redirect success as proof of payment.
* Grant access only after trusted backend confirmation.
* Fulfillment must be idempotent.
* Failed, cancelled, pending, and fulfilled states must be distinct.
* Billing metadata must not imply organization or team access unless explicitly supported.

## Shared Features

Shared features are permission-sensitive and must be constrained.

For any sharing feature, document:

* Who owns the shared object.
* Who can access it.
* Whether recipients need their own access.
* Whether links expire.
* Whether links can be revoked.
* Whether recipients can copy, answer, edit, comment, or only view.
* Whether the creator can see recipient activity.
* Whether rate limits or abuse protection apply.

Rules:

* Sharing must not bypass paid access.
* Sharing must not expose private user data.
* Shared links should have clear visibility, expiration, and revocation behavior.
* Shared workflows must not accidentally create organization, instructor, or admin behavior unless that is explicitly in scope.

## Data Privacy And Retention

If the app stores personal or sensitive data, define privacy rules.

Document:

* What data is collected.
* Why it is collected.
* Who can see it.
* What is private by default.
* What can be shared.
* What admins can access.
* How long data is retained.
* How deletion, export, or anonymization works if required.
* Whether analytics uses personal identifiers.

Rules:

* Collect only necessary data.
* Separate private data from shared data.
* Do not expose private data through reports, search, shared links, logs, or admin tools without explicit permission.
* Avoid storing sensitive data in places designed for debugging or analytics.
* Retention should be intentional, not accidental.
