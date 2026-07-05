# Operations, Quality, And Risk Reference

Read this when documenting non-functional requirements, external integrations, telemetry, audit logging, data impact, migration, backward compatibility, known limitations, or failure cases.

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

## Possible Issues

If implementation has compatibility issues, migration risks, environment constraints, known limitations, or failure cases, document them in a `Possible issues` section or column.

Each non-empty possible issue should include:

* Concrete description.
* Affected inputs, configs, or environments.
* Reproduction steps.
* Expected behavior.
* Actual behavior.
* Relevant URLs or references.
* Related code paths.
