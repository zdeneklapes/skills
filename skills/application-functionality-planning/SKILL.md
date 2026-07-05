---
name: application-functionality-planning
description: Use when creating, reviewing, or improving application requirements, product scope, feature checklists, permissions, authorization, data access, security, non-functional requirements, integrations, telemetry, migrations, roadmaps, or acceptance criteria. Do not use for code-only implementation work.
---

# Application Functionality Requirements Planning Skill

Use this skill when writing, reviewing, or improving application functionality documentation, product requirements, feature checklists, scope constraints, roadmaps, or security/authorization requirements.

The goal is to produce requirements that are clear, testable, implementation-aware, and safe to build.

## Core Principle

Application functionality documentation must describe:

1. What the system does.
2. Who can use each capability.
3. What is intentionally out of scope.
4. What security and permission rules apply.
5. How well the system must perform.
6. What data, integrations, telemetry, and migration impacts exist.
7. What counts as done.

Do not write only UI behavior. For every important capability, also describe backend rules, data ownership, constraints, and acceptance criteria.

## Recommended Document Structure

Use this structure for full application functionality documentation:

```markdown
# Application Functionality

## Status Legend
## Domain Glossary
## Product Scope
## Supported Now
## Planned Requirements
## Security, Authorization, Permissions, And Data Access
## Non-Functional Requirements
## External Integrations And Dependencies
## Analytics, Telemetry, And Audit Logging
## Data Impact, Migration, And Backward Compatibility
## Current Product Boundaries To Close
## Priority Roadmap
## Recommended Next Implementation Order
```

For smaller documents, keep only the sections that materially affect the product.

## Status Rules

Use a small, consistent status vocabulary.

Recommended statuses:

* `Done`: Supported by the current application.
* `Planned`: In scope, but not implemented yet.
* `Out of scope`: Intentionally not planned unless product strategy changes.
* `Required`: Mandatory cross-cutting rule, usually security, privacy, reliability, or compliance.

Rules:

* Do not list intentionally rejected features as missing functionality.
* Do not include out-of-scope items in the roadmap.
* Do not mix current functionality with planned functionality.
* If a competitor has a feature that the product will not support, document it as out of scope, not as missing.

## Domain Glossary

Define a strict glossary early in the document.

Use one canonical term for each domain concept and reuse it everywhere.

The glossary should define:

* Main actors.
* Main business objects.
* Important statuses.
* Important permission concepts.
* Product-specific terms.
* Terms that must not be used as synonyms.

Rules:

* If the document calls something a `Test`, do not also call it an `Exam`, `Quiz`, or `Assessment` unless those are separate glossary terms.
* If two terms mean different things, define the difference.
* If a term is user-facing and technical, state both meanings.
* Keep the glossary short, but strict.

Recommended format:

```markdown
## Domain Glossary

| Term | Meaning | Do not confuse with |
| --- | --- | --- |
| ... | ... | ... |
```

## Product Scope

Define product boundaries before listing features.

The scope section should answer:

* What kind of product is this?
* Who is it for?
* Which platforms are supported?
* Which account model is supported?
* Which billing or access model is supported?
* Which major areas are explicitly out of scope?

Rules:

* Make intentional limitations visible.
* Do not let out-of-scope items appear later as planned work.
* If a feature is out of scope, also state what is in scope instead.

Recommended format:

```markdown
## Product Scope

| Area | Status | Requirement |
| --- | --- | --- |
| ... | ... | ... |
```

## Supported Now

The `Supported Now` section describes only current behavior.

Group current functionality by product area.

Examples of possible areas:

* Public Website
* Accounts
* Plans And Purchases
* Access Control
* Core Workflow
* Workspace
* Search
* Reports
* Settings
* Admin, if applicable

Rules:

* Keep this section concise.
* Do not describe future improvements here.
* Do not mark something as done unless it is truly implemented.
* Do not hide known limitations. Mention them under product boundaries or planned requirements.

Recommended format:

```markdown
### Area Name

| Status | Functionality |
| --- | --- |
| Done | ... |
```

## Planned Requirements

Planned requirements should be implementation-ready but not overly technical.

Each requirement should answer:

1. What is required?
2. Who uses it?
3. What is the user-facing behavior?
4. What rules or constraints apply?
5. What is the data impact?
6. What is the acceptance criterion?

Recommended format:

```markdown
| Status | Requirement | User-facing behavior | Rules / Constraints | Acceptance |
| --- | --- | --- | --- | --- |
| Planned | ... | ... | ... | ... |
```

If the table becomes too wide, split constraints into a separate section.

Rules:

* Avoid vague requirements.
* Avoid implementation details that do not affect product behavior.
* Include edge cases when they change behavior.
* Include failure behavior when users could lose data, access, or trust.
* Include permission rules when data is user-owned, paid, private, or shared.

## Acceptance Criteria

Acceptance criteria must be observable and testable.

Do not write vague acceptance criteria such as:

```markdown
The feature works well.
```

Write criteria that can be verified by QA, automated tests, or review.

For simple requirements, a short acceptance sentence is enough.

For complex state changes, use `Given / When / Then`.

Recommended format:

```markdown
Given <initial state>,
When <action happens>,
Then <expected result>.
```

Use `Given / When / Then` for:

* Authorization decisions.
* Payment or access changes.
* Status transitions.
* Destructive actions.
* Sharing workflows.
* Moderation workflows.
* Data migrations.
* Retry or failure behavior.

Rules:

* Include the actor.
* Include the relevant state.
* Include the expected API behavior when relevant.
* Include the expected UI behavior when relevant.
* Include denial behavior for forbidden actions.
* Use one scenario per meaningful rule.

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

## Non-Functional Requirements

Non-functional requirements describe how well the system must behave.

Add a dedicated section for NFRs when the product has performance, reliability, accessibility, security, compliance, or operational expectations.

Do not write vague NFRs such as:

```markdown
The app must be fast.
The app must be reliable.
The app must be accessible.
```

Write measurable requirements.

NFR categories:

* Performance and latency.
* Throughput and concurrency.
* Availability.
* Reliability and failure recovery.
* Accessibility.
* Browser/device support.
* Security.
* Privacy.
* Maintainability.
* Observability.
* Data retention.
* Backup and recovery, if relevant.

Recommended format:

```markdown
| Category | Requirement | Measurement / Target | Acceptance |
| --- | --- | --- | --- |
| Performance | ... | ... | ... |
| Reliability | ... | ... | ... |
| Accessibility | ... | ... | ... |
```

Rules:

* Every important user flow should have a performance expectation.
* Every critical dependency should have failure behavior.
* Accessibility requirements should be explicit.
* Keyboard-only use should be specified where relevant.
* Avoid unrealistic targets unless the product has validated them.
* If the target is unknown, write that it must be measured before setting the final threshold.

Accessibility requirements should cover:

* Keyboard navigation.
* Screen reader compatibility.
* Focus states.
* Color contrast.
* Form labels and errors.
* Responsive layout.
* Target accessibility standard, such as WCAG level, when applicable.

## External Integrations And Dependencies

Document every third-party service that affects product behavior.

For each integration, state:

* Purpose.
* Data sent.
* Data received.
* Authentication or secret handling.
* Failure behavior.
* Retry behavior.
* User-facing fallback.
* Security/privacy considerations.
* Whether the integration is required or optional.
* Whether events from the provider are trusted and how they are verified.

Recommended format:

```markdown
| Service | Purpose | Data exchanged | Failure behavior | Security notes |
| --- | --- | --- | --- | --- |
| ... | ... | ... | ... | ... |
```

Rules:

* Do not treat external services as always available.
* Define what happens when a provider is slow, down, returns an error, or sends duplicate events.
* Define whether user actions should fail, retry, queue, or continue without the integration.
* Never expose integration secrets in UI, logs, errors, or analytics.

## Analytics, Telemetry, And Audit Logging

Plan telemetry together with the feature.

Differentiate:

* Business analytics: product-facing events used to understand user behavior and funnels.
* Operational telemetry: metrics, traces, and logs used to debug and operate the system.
* Audit logs: security-sensitive records of important actions.

For each important feature, specify:

* Which business events are tracked.
* Which system events are logged.
* Which metrics are measured.
* Which admin/security actions are audited.
* Which identifiers are allowed.
* Which data must never be logged.

Recommended format:

```markdown
| Event type | Purpose | Trigger | Data allowed | Data forbidden |
| --- | --- | --- | --- | --- |
| Business analytics | ... | ... | ... | ... |
| Operational log | ... | ... | ... | ... |
| Audit log | ... | ... | ... | ... |
```

Rules:

* Do not log passwords, tokens, payment secrets, webhook secrets, private keys, full card data, or sensitive raw payloads.
* Avoid logging unnecessary PII.
* Redact or hash identifiers where possible.
* Audit important admin actions.
* Log security-relevant failures without leaking sensitive details.
* Define retention for logs and audit records when relevant.

## Data Impact, Migration, And Backward Compatibility

Any modification to existing functionality should describe its data impact.

Add data impact notes when a requirement:

* Adds a field.
* Changes a status.
* Changes permissions.
* Changes billing/access behavior.
* Changes existing reports.
* Changes meaning of old data.
* Introduces new default settings.
* Deprecates or archives existing records.

For each change, document:

* Whether a data migration is required.
* What happens to existing records.
* The default state for existing users.
* Whether old and new application versions must coexist during deployment.
* Whether rollback is safe.
* Whether backfill is needed.
* Whether historical reports should change or preserve old meaning.

Recommended format:

```markdown
| Change | Migration needed | Legacy data behavior | Default for existing users | Rollback / compatibility |
| --- | --- | --- | --- | --- |
| ... | ... | ... | ... | ... |
```

Rules:

* Do not assume existing data already has values for new fields.
* Define safe defaults.
* Preserve user access and progress unless the requirement explicitly says otherwise.
* Do not silently reinterpret historical data in a way that misleads users.
* For destructive changes, state whether the change is reversible.

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

## Product Boundaries To Close

Use this section for known gaps in existing pages or flows.

Examples of boundary types:

* Existing page is placeholder-only.
* Existing form validates locally but does not submit.
* Existing workflow has UI but no backend action.
* Existing feature exists but lacks permissions, telemetry, or acceptance criteria.

Recommended format:

```markdown
| Status | Boundary | Requirement | Acceptance |
| --- | --- | --- | --- |
| Planned | ... | ... | ... |
```

Rules:

* Only include real current gaps.
* Do not include out-of-scope items.
* Use this section to prevent half-built UI from being mistaken for complete functionality.

## Priority Roadmap

Group planned work by priority.

Use:

```markdown
## Priority Roadmap

### Must Add First
### Add Second
### Add Third
```

Each roadmap item should include:

* Requirement.
* Reason.
* Expected result.

Recommended format:

```markdown
| Status | Requirement | Reason | Expected result |
| --- | --- | --- | --- |
| Planned | ... | ... | ... |
```

Rules:

* Prioritize foundational data, permissions, and core workflows before polish.
* Do not include out-of-scope work.
* Do not include duplicates unless the roadmap summarizes a detailed section.
* Security, authorization, data integrity, and migration work should not be buried as optional polish.

## Recommended Implementation Order

Implementation order should follow dependencies.

Prefer this order:

1. Domain model and terminology cleanup.
2. Data model and migration planning.
3. Authorization and ownership rules.
4. Backend services and validation.
5. Core user-facing workflows.
6. Reports and analytics.
7. Telemetry and audit logs.
8. Performance and reliability hardening.
9. UI polish and productivity features.

Recommended format:

```markdown
| Order | Requirement | Start with | Output |
| ---: | --- | --- | --- |
| 1 | ... | ... | ... |
```

Rules:

* Put shared backend logic before multiple UI features depend on it.
* Put migration and compatibility planning before changing existing behavior.
* Put permissions before sharing or admin workflows.
* Put telemetry before production rollout when the feature needs monitoring.

## Review Checklist

Before finalizing documentation, verify:

* Product scope is explicit.
* Out-of-scope items are not listed as planned.
* A domain glossary exists for important terms.
* Supported-now items describe only current behavior.
* Planned items are testable.
* Each important requirement has user-facing behavior.
* Each important requirement has acceptance criteria.
* Complex acceptance criteria use Given / When / Then.
* Security is enforced server-side, not only in UI.
* Authentication and authorization are separate.
* User-owned data has ownership rules.
* Paid content has backend access checks.
* Shared features cannot bypass permissions.
* Billing does not trust browser-only success states.
* Admin actions are permissioned and audited.
* Private and shared data are clearly separated.
* User-generated content has validation, safe rendering, and rate limits.
* Non-functional requirements are measurable.
* External integrations define failure and fallback behavior.
* Analytics, telemetry, and audit logging are planned.
* Sensitive PII, tokens, and secrets are forbidden in logs.
* Data migration and backward compatibility are defined for changed features.
* Roadmap order follows dependencies.
* The document stays product-focused and avoids unnecessary implementation details.

## Writing Style

Use precise product language.

Rules:

* Be clear and direct.
* Prefer measurable requirements over adjectives.
* Use one canonical term from the glossary.
* Avoid too many concrete examples.
* Avoid implementation details unless they affect behavior, security, data, or delivery risk.
* Avoid duplicating the same rule in many places.
* Keep the skill lean by including only rules that change product behavior, implementation decisions, testing, security, or operational readiness.
