# Architecture and Cross-Cutting Reference

Use this reference when documentation needs to explain technical structure, important decisions, or concerns that span several product areas. Apply the sections that match the task; a small feature rarely needs every section.

## Architecture Documentation

Architecture documentation explains how the system actually works now. Follow the repository's existing format and location, whether it uses architecture pages, ADRs, decision logs, diagrams, or another convention.

Create or expand architecture documentation when the user requests it, an established architecture document would otherwise become inaccurate, or a durable explanation would materially reduce future implementation risk. Do not create an architecture tree merely because a feature changed.

Useful architecture documentation answers questions such as:

* Where does an important flow start, and which module owns each decision?
* What happens first, next, and last, including meaningful branches or retries?
* What data is read, transformed, persisted, published, or returned?
* Which boundaries, dependencies, and external systems matter?
* Which files or generated artifacts must change together?
* Which trade-offs or constraints explain a non-obvious design?

Use Mermaid, D2, sequence diagrams, or other visuals when they clarify relationships or event order better than prose. Keep diagrams consistent with the text and current code.

## Decisions and ADRs

Use the project's established decision-record format. A useful decision record typically captures the decision, context, alternatives considered, consequences, trade-offs, and relevant code or configuration paths.

Decision records may live beside architecture pages or in a dedicated ADR or decisions folder. Prefer one canonical record and link to it from related architecture or requirements documents. Do not rewrite historical decisions to pretend the original context was different; add a superseding decision when the project relies on decision history.

## Applying Cross-Cutting Guidance

Cross-cutting sections are prompts, not a mandatory documentation template. Document a concern when it changes behavior, risk, implementation constraints, operations, or acceptance criteria. For minor or unaffected features, a short statement or no new section may be appropriate.

## Security, Authorization, and Data Access

For protected or sensitive features, distinguish:

* Authentication: who the user or caller is.
* Authorization: what actions that identity may perform.
* Ownership: which records or resources belong to whom.
* Access: which paid, private, shared, or restricted content is available.

Document server-side enforcement where it matters. Frontend guards can improve usability but should not be described as the security boundary. Explain denial behavior, relevant roles, and object-level access when readers would otherwise need to guess.

An access matrix with `Role`, `Resource`, `Allowed actions`, and `Restrictions` can help complex systems, but prose is enough for a simple rule.

## User-Generated Content, Billing, and Sharing

When a feature accepts user content, consider visibility, validation, safe rendering, moderation, abuse handling, and whether sensitive content may appear in logs.

When billing controls access, consider trusted confirmation of payment, pending and failed states, idempotent fulfillment, refunds or revocation, and ownership of invoices or purchase history. Browser redirects alone should not be documented as proof of payment.

For shared objects or links, consider ownership, recipient permissions, authentication or paid-access requirements, visibility, expiry, revocation, and activity disclosure. Do not imply organization or administrative capabilities unless the product actually supports them.

## Privacy and Retention

For personal or sensitive data, document the relevant collection purpose, default visibility, sharing, administrative access, retention, deletion, export, and anonymization behavior. Explain where identifiers may appear in analytics, logs, search, reports, or shared views.

Prefer collecting and retaining only what the product needs. Never include real secrets, credentials, payment secrets, private keys, tokens, or sensitive user content in documentation examples.

## Non-Functional Requirements

Add measurable quality requirements when a feature has meaningful expectations for performance, throughput, availability, reliability, accessibility, browser or device support, observability, backup, or recovery.

Use a table such as `Category`, `Expectation`, `Measurement`, and `Acceptance` when several targets need tracking. For smaller work, concise prose is enough. If a target is unknown, record the need to measure it rather than inventing a number.

## External Integrations

For an integration that affects product behavior, consider:

* Purpose and data exchanged.
* Authentication and secret handling.
* Timeouts, retries, idempotency, and duplicate events.
* User-visible behavior when the provider is slow or unavailable.
* Whether incoming events are trusted and how authenticity is verified.
* Whether the dependency is required or optional.

Tables can help compare several providers, but a focused paragraph may be enough for one simple dependency.

## Analytics, Telemetry, and Audit Logging

Distinguish business analytics, operational telemetry, and security audit records when the difference matters. Document useful events, metrics, or audited actions together with allowed and forbidden data.

Do not log passwords, tokens, payment secrets, private keys, sensitive raw payloads, or unnecessary personal data. When relevant, describe redaction, identifier handling, and retention. Do not require telemetry for a feature that has no meaningful monitoring or product-analysis need.

## Data Impact, Migration, and Compatibility

When behavior changes existing data, permissions, billing, reports, defaults, or external contracts, consider:

* Whether a schema migration or backfill is required.
* How legacy records and existing users behave.
* Defaults for newly introduced fields or settings.
* Compatibility during staged deployment.
* Rollback safety and whether destructive changes are reversible.

A compatibility table is useful for complex rollouts; concise prose is sufficient when the answer is simple. Do not imply that existing data already contains new values.

## Operations, Runbooks, and Known Risks

Follow the project's established operations and runbook structure. Document operational procedures when they are repeatable, risky, time-sensitive, or difficult to reconstruct during an incident. Commands should state where they run, relevant prerequisites, expected results, and meaningful safety constraints.

Record known limitations and failure cases when they affect supported environments or delivery. Include reproduction details and references only to the degree needed for someone to understand or act on the issue; use an issue tracker instead when it is the canonical operational record.

## Review Questions

Before finalizing cross-cutting documentation, ask:

* Does it match the current implementation and deployed contract?
* Are diagrams, code paths, and commands still current?
* Are the relevant security, privacy, integration, and compatibility risks visible?
* Did the document avoid imposing irrelevant sections or speculative targets?
* Is each decision or procedure stored once and linked from related documents?
