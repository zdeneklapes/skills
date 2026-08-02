---
name: test-driven-development
description: Use when implementing or changing production runtime behavior, shipped libraries or CLIs, high-risk operational automation, or other production-affecting code. Do not use by default for documentation, generated artifacts, test-only changes, low-risk configuration, or developer and project-management helpers that do not ship or affect production.
---

# Test-Driven Development (TDD)

## Overview

For material production behavior, write a focused test first, observe the expected failure, then write the minimum code needed to pass. Apply TDD to behavior and risk, not every executable file, function, or line.

## Decide Whether TDD Applies

Classify the changed behavior before writing a test or implementation. Trace how the artifact is invoked, packaged, shipped, deployed, or consumed; do not classify it from its language or directory name alone.

### Production behavior: use TDD

Use TDD when changed code:

- Executes in a deployed application, service, worker, or scheduled runtime.
- Ships to users or consumers as a library, SDK, CLI, plugin, or public interface.
- Implements business rules, security, authorization, validation, data handling, persistence, or externally visible behavior.
- Automates production deployment, migration, backup, restore, incident recovery, infrastructure mutation, secrets, or availability.

A script under `scripts/` can therefore be production-critical: a restore script that writes to a production database needs regression coverage.

### Ancillary work: outside this skill

This skill does not require failing-test-first for documentation, comments, examples, plans, metadata, tests or fixtures that do not change a production contract, generated artifacts when only output changes, developer-only convenience scripts, recoverable project-management helpers, development-only CI glue, or throwaway exploration that will not ship.

These are boundaries, not blanket exemptions. A helper becomes in-scope when used in release, deployment, recovery, security, or another production-affecting workflow. If unclear, trace callers, packaging, CI, and deployment before deciding. Once classified as ancillary, stop applying this skill.

## The Scoped Rule

```text
NO NEW OR CHANGED PRODUCTION BEHAVIOR WITHOUT AN OBSERVED FAILING TEST FIRST
```

Apply this rule to each material observable behavior or invariant, not every new function or method. Prefer a stable public boundary; cover trivial helpers and private implementation details indirectly.

## Red-Green-Refactor

### RED: specify one behavior

Write the smallest test that demonstrates one desired production behavior, regression, contract, or invariant.

- Give it a clear behavioral name.
- Exercise real code; mock only external boundaries that cannot safely or deterministically run.
- Assert outcomes at a stable seam, not implementation shape.

Run the test before changing production behavior. Confirm it fails because behavior is missing or wrong, not because of a typo or broken setup. If it passes immediately, revise the test or confirm behavior already exists.

### GREEN: implement minimally

Write only enough production code to pass the failing test. Do not add unrelated features or refactors. Run the focused test, then the relevant broader suite. Fix production code when the behavioral expectation is correct; do not weaken the test merely to obtain green output.

### REFACTOR: improve while green

After green:

- Remove duplication.
- Improve names and structure.
- Keep behavior unchanged.
- Re-run relevant tests.

Repeat with another failing test for the next material behavior.

## Test Observable Contracts

One focused test may cover several internal helpers. Do not add a dedicated test for every private method, assert internal call order, or duplicate implementation logic in assertions. Cover important boundaries, errors, and edge cases without enumerating low-value permutations. Prefer a smaller durable suite over a large brittle one.

## Regression Tests and Test Seams

When fixing a production bug, reproduce it with a failing regression test and follow red-green-refactor. Tests written after implementation still provide valuable characterization or regression coverage, but they are not TDD evidence; when practical, show that the test detects the old behavior using the pre-change implementation or a controlled mutation.

If direct testing is impractical, seek a stable public seam, isolated fixture, sandbox, or integration test that exercises the real contract. Do not create brittle tests that cannot observe meaningful behavior. Use mocks only for external, nondeterministic, unavailable, or unsafe boundaries; keep mock assertions subordinate to the real contract.

## TDD Checklist

- [ ] Traced invocation, packaging, shipping, and deployment impact.
- [ ] Classified the change as production behavior or ancillary work.
- [ ] For in-scope behavior, observed each focused test fail for the expected reason before implementation.
- [ ] Implemented the minimum behavior, ran focused and relevant broader tests, then refactored while green.
- [ ] Covered material contracts, errors, and edge cases through stable seams.
- [ ] Avoided testing private helpers, implementation details, and mock behavior instead of the contract.

## Testing Anti-Patterns

When adding mocks or test utilities, read `testing-anti-patterns.md` and avoid:

- Testing mock behavior instead of the real contract.
- Adding test-only methods or production branches solely to make tests easier.
- Mocking dependencies without understanding their observable behavior.
- Making assertions mirror the implementation rather than user-visible outcomes.

## Final Rule

```text
Production behavior -> failing test first, then minimal implementation and refactoring
Ancillary helper -> outside this skill
```
