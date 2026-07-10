# Application Requirements Reference

Application requirements describe what the final product should do.

Do not write only UI behavior. For every important capability, also describe backend rules, data ownership, constraints, and acceptance criteria.

## Recommended Structure

Recommended files:

```text
docs/docs/requirements/
  index.md
  00-glossary-and-scope.md
  01-public-website.md
  02-auth-billing-accounts.md
  03-main-product-area.md
  04-admin.md
  05-security-privacy.md
  06-non-functional-requirements.md
  07-integrations.md
  99-roadmap.md
```

Use product-area names that fit the project when the example names do not.

`index.md` should be the high-level index: product summary, document list, status legend, target legend, source-of-truth summary, core user journeys, supported-now summary, and roadmap summary.

For full application functionality documentation, use these sections where relevant:

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

For the security, non-functional, integrations, telemetry, and data-impact sections, see `architecture-and-cross-cutting.md`.

The detailed numbered files are the canonical home for full sections; keep `index.md` as the summary index. Use the full section list in `index.md` only when it is the single requirements document.

For smaller documents, keep only the sections that materially affect the product.

## Status Rules

Use a small, consistent status vocabulary.

Recommended statuses:

* `Done`: Supported by the current application.
* `Planned`: In scope, but not implemented yet.
* `Out of scope`: Intentionally not planned unless product strategy changes.
* `Required`: Mandatory cross-cutting rule, usually security, privacy, reliability, or compliance.

Do not mix current functionality with planned functionality. When changing a `Done` requirement, edit the row in place to describe the new behavior and set its status to `Planned` until the change is implemented.

## Status-Filter Assets

Requirement status tables can be filtered with shared MkDocs assets.

* Put the status-filter CSS and JavaScript in `docs/docs/assets/` and use them only for requirement/status tables.
* Filtering requires a `Status` column and the status vocabulary above, used consistently everywhere.
* Register assets in `mkdocs.yml`; `extra_css` and `extra_javascript` paths are relative to the configured docs directory:

```yaml
extra_css:
  - assets/status-filter.css

extra_javascript:
  - assets/status-filter.js
```

## Domain Glossary

Define a strict glossary early in the document, as a table with columns `Term`, `Meaning`, and `Do not confuse with`.

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

## Product Scope

Define product boundaries before listing features, as a table with columns `Area`, `Status`, and `Requirement`.

The scope section should answer:

* What kind of product is this?
* Who is it for?
* Which platforms are supported?
* Which account model is supported?
* Which billing or access model is supported?
* Which major areas are explicitly out of scope?

Rules:

* Make intentional limitations visible.
* Out-of-scope items appear only here — never as missing features, planned work, roadmap items, or product boundaries. This includes competitor features the product will not support.
* If a feature is out of scope, also state what is in scope instead.

## Supported Now

The `Supported Now` section describes only current behavior.

Group current functionality by product area (for example Public Website, Accounts, Plans And Purchases, Access Control, Core Workflow, Search, Reports, Settings, Admin), each area as a table with columns `Status` and `Functionality`.

Rules:

* Keep this section concise.
* Do not describe future improvements here.
* Do not mark something as done unless it is truly implemented.
* Do not hide known limitations. Mention them under product boundaries or planned requirements.

## Planned Requirements

Planned requirements should be implementation-ready but not overly technical.

Each requirement should answer:

1. What is required?
2. Who uses it?
3. What is the user-facing behavior?
4. What rules or constraints apply?
5. What is the data impact?
6. What is the acceptance criterion?

Detailed requirement files under `docs/docs/requirements/` should use stable IDs and target columns:

```markdown
| ID | Status | Target | Requirement | Acceptance |
| --- | --- | --- | --- | --- |
| AUTH-001 | Done | Backend | Users can log in with email and password. | Valid users can create a session and invalid credentials are rejected. |
| AUTH-002 | Planned | Tests | Add login rate-limit tests. | Tests verify repeated failed login attempts are limited. |
```

If the table becomes too wide, split constraints into a separate section.

Rules:

* Use stable IDs before implementation starts.
* Use targets like `Public Frontend`, `User Frontend`, `Admin UI`, `Backend`, `Database`, `Content`, `Integrations`, `Operations`, and `Tests`.
* One row should describe one implementable requirement.
* If only the UI exists but backend behavior is missing, split it into separate rows.
* Avoid implementation details that do not affect product behavior.
* Include edge cases when they change behavior.
* Include failure behavior when users could lose data, access, or trust.
* Include permission rules when data is user-owned, paid, private, or shared.

## Acceptance Criteria

Acceptance criteria must be observable and testable: verifiable by QA, automated tests, or review.

For simple requirements, a short acceptance sentence is enough.

For complex state changes, use `Given / When / Then`:

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

## Product Boundaries To Close

Use this section for known gaps in existing pages or flows, as a table with columns `Status`, `Boundary`, `Requirement`, and `Acceptance`.

Examples of boundary types:

* Existing page is placeholder-only.
* Existing form validates locally but does not submit.
* Existing workflow has UI but no backend action.
* Existing feature exists but lacks permissions, telemetry, or acceptance criteria.

Only include real current gaps. Use this section to prevent half-built UI from being mistaken for complete functionality.

## Priority Roadmap

Group planned work by priority:

```markdown
## Priority Roadmap

### Must Add First
### Add Second
### Add Third
```

Each roadmap item should include the requirement, the reason, and the expected result, as a table with columns `Status`, `Requirement`, `Reason`, and `Expected result`.

Do not include duplicates unless the roadmap summarizes a detailed section.

## Recommended Implementation Order

Implementation order should follow dependencies, as a table with columns `Order`, `Requirement`, `Start with`, and `Output`.

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

Rules:

* Prioritize foundational data, permissions, and core workflows before polish. Security, authorization, data integrity, and migration work must not be buried as optional polish.
* Put shared backend logic before multiple UI features depend on it.
* Put migration and compatibility planning before changing existing behavior.
* Put permissions before sharing or admin workflows.
* Put telemetry before production rollout when the feature needs monitoring.
