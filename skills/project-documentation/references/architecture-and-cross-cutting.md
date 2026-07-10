# Architecture And Cross-Cutting Reference

## Architecture Docs

Write architecture documentation only when the user or developer explicitly asks for it. Do not create or expand `docs/docs/architecture/` as a side effect of feature or requirements work.

Architecture docs explain how the code actually works now. Organize `docs/docs/architecture/` by area, for example `index.md`, `backend.md`, `frontend.md`, `data-model.md`, `execution-flows.md`, `integrations.md`.

Good architecture docs answer:

* Where does the flow start, and which function, class, or module owns the logic?
* What is executed first, next, and last, and what branches, conditions, or loops matter?
* What data is read, transformed, saved, or returned?
* Which libraries and packages are used, why, and what trade-offs were accepted?
* Which files should be updated together?

Use Mermaid or D2 diagrams when they clarify flows.

Record important technical decisions inside the relevant architecture page: the decision, why it was made, alternatives considered, consequences and trade-offs, and relevant code paths.

## Security, Authorization, Permissions, And Data Access

Security must be part of the requirements, not an afterthought. Always distinguish authentication (who the user is), authorization (what the user may do), ownership (which objects belong to the user), and access (which paid, private, shared, or restricted content the user may use).

Requirements docs must state these invariants:

* Backend API authorization is mandatory; frontend guards are usability helpers, not security boundaries.
* Every protected endpoint checks authentication, every user-owned object checks ownership, and every restricted feature checks permissions server-side.
* Paid or locked content is not returned by the API to unauthorized users.
* Permissions fail closed when unclear, with consistent denial behavior.

Define roles and permission levels as a table with columns `Role`, `Purpose`, `Allowed actions`, and `Restrictions`.

For every sensitive object type (account data, billing data, user-created content, shared objects, admin-only objects, reports), document who can create, read, edit, and delete it, and how long it is retained. The sections below add only their specific concerns.

## User-Generated Content

For any feature that accepts user text, additionally document whether it is private, shared, public, or moderated; how it is validated and rendered safely; and whether rate limits, abuse reporting, or moderation apply.

Invariants: user-generated text is never assumed safe; private text cannot become shared accidentally; sensitive user content is never logged (see Analytics, Telemetry, And Audit Logging).

## Paid Access And Billing

Additionally document free and paid access rules, expired-access behavior, refund or revocation behavior, plan-to-permission mapping, checkout and fulfillment rules, and user ownership of orders, invoices, and payment history.

Invariants: access is granted only after trusted backend confirmation, never on browser redirect success; fulfillment is idempotent, with failed, cancelled, pending, and fulfilled states kept distinct; billing metadata does not imply organization or team access unless explicitly supported.

## Shared Features

Additionally document who owns the shared object, who can access it, whether recipients need their own access, whether links expire and can be revoked, what recipients can do (copy, answer, edit, comment, or only view), and whether the creator sees recipient activity.

Invariants: sharing does not bypass paid access and does not expose private user data; shared links have clear visibility, expiration, and revocation behavior; sharing does not create organization, instructor, or admin behavior unless explicitly in scope.

## Data Privacy And Retention

If the app stores personal or sensitive data, additionally document what is collected and why, what is private by default, what can be shared, what admins can access, how deletion, export, or anonymization works, and whether analytics uses personal identifiers.

Invariants: collect only necessary data; private data is not exposed through reports, search, shared links, logs, or admin tools without explicit permission; retention is intentional, not accidental.

## Non-Functional Requirements

Write measurable non-functional requirements as a table with columns `Category`, `Requirement`, `Measurement / Target`, and `Acceptance`, covering the relevant categories: performance, throughput, availability, reliability, accessibility (keyboard navigation, screen readers, focus states, color contrast, form labels, target WCAG level), browser/device support, security, privacy, observability, data retention, and backup.

Rules: every important user flow has a performance expectation; every critical dependency has failure behavior; if a target is unknown, state that it must be measured before setting the final threshold.

## External Integrations And Dependencies

Document every third-party service that affects product behavior as a table with columns `Service`, `Purpose`, `Data exchanged`, `Failure behavior`, and `Security notes`, plus authentication and secret handling, retry behavior, user-facing fallback, whether the integration is required or optional, and whether provider events are trusted and how they are verified.

Invariants: external services are not treated as always available — define behavior for slow, down, erroring, or duplicate-event providers; user actions have defined outcomes when the integration fails (fail, retry, queue, or continue without it); integration secrets never appear in UI, logs, errors, or analytics.

## Analytics, Telemetry, And Audit Logging

Plan telemetry together with the feature. Differentiate business analytics (product events and funnels), operational telemetry (metrics, traces, logs), and audit logs (security-sensitive records of important actions). For each important feature, specify tracked events, logged system events, measured metrics, audited admin/security actions, allowed identifiers, and forbidden data — as a table with columns `Event type`, `Purpose`, `Trigger`, `Data allowed`, and `Data forbidden`.

Invariants: passwords, tokens, payment secrets, webhook secrets, private keys, card data, sensitive raw payloads, and sensitive user content are never logged; unnecessary PII is avoided and identifiers are redacted or hashed where possible; important admin actions are audited, and security-relevant failures are logged without leaking sensitive details; log and audit retention is defined when relevant.

## Data Impact, Migration, And Backward Compatibility

When a requirement changes existing data, statuses, permissions, billing behavior, reports, or defaults, document migration need, legacy data behavior, defaults for existing users, deployment coexistence, rollback safety, and backfill — as a table with columns `Change`, `Migration needed`, `Legacy data behavior`, `Default for existing users`, and `Rollback / compatibility`.

Invariants: existing data is not assumed to have values for new fields — define safe defaults; user access and progress are preserved unless the requirement explicitly says otherwise; historical data is not silently reinterpreted in a way that misleads users; destructive changes state whether they are reversible.

## Possible Issues

Document compatibility issues, migration risks, environment constraints, known limitations, and failure cases in a `Possible issues` section or column. Each entry: concrete description, affected inputs, configs, or environments, reproduction steps, expected and actual behavior, and relevant references or code paths.
