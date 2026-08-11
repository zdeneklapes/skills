---
name: owasp-security-review
description: Use when auditing, reviewing, designing, or hardening web or API services against OWASP Top 10 risks, including access control, authentication, injection, SSRF, cryptography, configuration, dependencies, logging, and integrity failures.
---

# OWASP Security Review

Combine OWASP Top 10 assessment with concrete remediation. Treat server-side controls, tests, and operational safeguards as part of the security boundary.

## Workflow

1. **Scope**: identify services, trust boundaries, data, actors, entry points, and deployment assumptions.
2. **Assess**: select relevant OWASP categories; inspect code, configuration, dependencies, and tests. Load detailed references only for selected categories.
3. **Record**: for each finding, state category, severity, evidence, impact, affected path, remediation, and verification method. Never include secrets or sensitive payloads.
4. **Remediate**: apply the smallest complete server-side fix. Use established libraries, least privilege, allowlists, secure defaults, and defense in depth.
5. **Verify**: add or run focused regression/security tests, then run SAST, dependency, secret, or dynamic checks available in the project.

## Coverage

| Category | Check first |
| --- | --- |
| A01 Access control | Authentication, ownership, tenant scope, RBAC, IDOR, deny-by-default |
| A02 Cryptography | Password hashing, key storage, encryption, TLS, token handling |
| A03 Injection | Parameterized queries, typed validation, output encoding, shell safety |
| A04 Insecure design | Threat model, abuse cases, rate limits, safe failure, resource bounds |
| A05 Misconfiguration | Production errors, headers, cookies, defaults, exposed services |
| A06 Components | Dependency versions, advisories, lockfiles, transitive risk |
| A07 Authentication | MFA, session rotation, expiry, reset flows, CSRF |
| A08 Integrity | Signed updates, webhook verification, deserialization, CI/build trust |
| A09 Logging | Security events, redaction, retention, alerting, incident evidence |
| A10 SSRF | Scheme/host allowlists, DNS rebinding, redirects, private/metadata IPs, egress limits |

## Release Gate

Block release for unmitigated critical/high findings, missing authorization, exploitable injection, SSRF, authentication bypass, or unverified security fixes. Do not treat frontend checks as authorization. Do not claim remediation without evidence.

Use `$test-driven-development` for production behavior and `$systematic-debugging` before fixing a security failure.
