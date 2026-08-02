# Application Requirements Reference

Application requirements describe what a product should do from the perspective of users, operators, and other systems. They should be detailed enough to guide implementation and verification without forcing every project into the same document structure.

## Scale the Documentation

Match the format to the size and maturity of the project:

* For a small project or isolated change, a focused section in an existing `README.md` or product specification may be enough.
* For an actively developed product, a requirements index plus topic pages can make ownership and status easier to maintain.
* For regulated, security-sensitive, or multi-team work, stable identifiers, traceability, and more explicit acceptance criteria may be worthwhile.

Prefer an existing canonical requirements location. Create a new structure only when the current documents cannot remain clear and maintainable.

One possible structure for a larger product is:

```text
<documentation-root>/requirements/
  index.md
  glossary-and-scope.md
  accounts-and-access.md
  core-product.md
  administration.md
  security-and-quality.md
  roadmap.md
```

Adapt names and grouping to the product. A smaller project may need only one file.

## Useful Content

A requirements collection may cover:

* Product purpose, intended users, and supported platforms.
* Current functionality and known limitations.
* Planned behavior and explicit exclusions.
* Domain terminology where ambiguity would affect implementation.
* Business rules, ownership, permissions, and important failure behavior.
* Acceptance criteria and relevant quality expectations.
* External dependencies, data impact, migration, or compatibility concerns.
* Roadmap or implementation dependencies when planning is part of the request.

Use only the sections that materially improve the specification. Do not add empty sections or large tables merely to match a template.

## Current, Planned, and Out of Scope

Keep current behavior distinguishable from future work. Plain prose can do this in a compact specification. Use the project's existing status vocabulary when it has one; introduce labels only when the project benefits from tracking requirements by state. A small vocabulary such as the following is often sufficient:

* `Done`: supported by the current product.
* `Planned`: intended but not implemented.
* `Out of scope`: intentionally excluded.
* `Required`: a cross-cutting constraint that applies to relevant work.

These labels are suggestions, not required names. Avoid marking behavior as done without evidence. When a current capability is changing, describe both the current state and intended replacement clearly enough that readers are not misled during implementation.

## Glossary and Scope

Add a glossary when the domain contains overloaded terms, multiple actors, important statuses, or permission concepts that are easy to confuse. A compact table with `Term`, `Meaning`, and optionally `Not the same as` is usually enough.

Define product boundaries when scope is otherwise ambiguous. State what is supported, who it serves, and important exclusions. Do not repeat the same exclusion throughout roadmaps, gaps, and feature lists; link back to the canonical scope statement.

## Writing Individual Requirements

For an important capability, consider:

1. What behavior is required?
2. Who or what uses it?
3. Which rules, constraints, permissions, or failure cases matter?
4. Does it change stored data or an external contract?
5. How can the result be observed or verified?

Do not reduce a full-stack capability to UI behavior when backend rules, persistence, ownership, or integrations determine whether it actually works. Conversely, do not demand backend, telemetry, migration, or NFR sections when they have no meaningful bearing on the change.

Stable IDs help when requirements are referenced from issues, tests, releases, or multiple documents. Do not introduce them for a short-lived or compact specification unless traceability would improve maintenance. When a table is useful, adapt its columns:

```markdown
| ID | Status | Area | Requirement | Acceptance |
| --- | --- | --- | --- | --- |
| AUTH-001 | Done | Backend | Valid users can create a session. | Valid credentials create a session; invalid credentials do not. |
```

Use one row per independently understandable behavior. Split UI and backend rows only when they can be delivered, tracked, or verified separately.

## Acceptance Criteria

Acceptance criteria should describe observable outcomes. A short sentence is usually sufficient for simple behavior.

Use `Given / When / Then` or equivalent scenarios when state, actor, or failure behavior would otherwise remain ambiguous, for example authorization decisions, payments, destructive actions, sharing, moderation, migrations, and retries:

```markdown
Given <relevant starting state>,
When <an actor or system performs an action>,
Then <an observable result occurs>.
```

Cover denial and failure behavior when users could lose data, access, money, privacy, or trust. Avoid enumerating low-value permutations that do not change the contract.

## Cross-Cutting Prompts

For each substantial feature, consider whether any of these topics affect its behavior:

* Authentication, authorization, ownership, paid access, or sharing.
* Personal or user-generated data, privacy, retention, export, or deletion.
* Accessibility, performance, reliability, browser or device support.
* Third-party services and degraded or duplicate-event behavior.
* Analytics, operational telemetry, or security audit records.
* Migration, backward compatibility, defaults, rollout, or rollback.

Document only the relevant answers. Use `architecture-and-cross-cutting.md` for deeper guidance.

## Gaps, Roadmaps, and Implementation Order

A gap list can distinguish incomplete current flows from genuinely planned features. Record concrete gaps such as a placeholder page, a form without submission, missing backend enforcement, or absent failure handling. Do not invent a gap register when an issue tracker already serves as the canonical source.

Roadmaps are useful when prioritization is requested. Keep them at the outcome level and link to detailed requirements rather than duplicating them.

When implementation order belongs in the documentation, derive it from real dependencies. Data compatibility, authorization, shared backend contracts, and rollout safety commonly need attention before dependent UI polish, but there is no universal sequence. For detailed execution steps, use the project's planning workflow rather than turning requirements into a code-level plan.

## Optional Status-Table Enhancements

Filterable status tables can help large requirements sets when the documentation system supports custom CSS or JavaScript. Keep assets with the documentation that uses them and follow the project's delivery configuration. A plain readable table should remain useful without custom filtering.

## Review Questions

Before finalizing requirements, ask:

* Does the document describe current behavior accurately?
* Are planned work and exclusions clear without being duplicated?
* Are important actors, rules, data effects, and failure cases covered?
* Are acceptance criteria proportional and observable?
* Can implementation and verification proceed without guessing about material behavior?
