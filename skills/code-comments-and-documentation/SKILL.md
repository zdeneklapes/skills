---
name: code-comments-and-documentation
description: Use when writing, reviewing, or refactoring code comments, docstrings, public API documentation, TODO/FIXME notes, or code that is non-trivial, public, risky, security-sensitive, performance-sensitive, stateful, or hard to understand. Do not use for project-level documentation such as README, requirements, or architecture docs.
---

# Clean Code, Comments, and Documentation

Use this skill when writing, reviewing, or refactoring code that is non-trivial, public, risky, security-sensitive, performance-sensitive, or hard to understand.

## Core Rule

Optimize for the next reader.

Prefer clearer code before adding comments. Use better names, smaller functions, stronger types, simpler control flow, and tests first. Add comments only when they preserve information that the code itself cannot clearly express.

## First Make the Code Self-Explaining

Before writing a comment, check whether the code can be improved instead:

* Rename variables, functions, classes, files, or modules to express intent.
* Extract complex branches into named helper functions.
* Replace magic values with named constants.
* Use types, enums, dataclasses, schemas, or domain objects where they clarify meaning.
* Split unrelated responsibilities.
* Add tests that document expected behavior.
* Remove dead code instead of commenting it out.

A comment should not compensate for unnecessarily confusing code.

## What Comments Are For

Write comments for information that is important but not obvious from the code:

* Why this approach exists.
* Why a simpler-looking alternative is wrong.
* Business/domain rules that are not visible from implementation.
* Security, privacy, permission, or abuse-prevention reasoning.
* Performance trade-offs and complexity constraints.
* Concurrency, locking, ordering, transaction, retry, idempotency, and cache invalidation rules.
* Data invariants and assumptions.
* Units, ranges, time zones, nullability, precision, and normalization rules.
* External system constraints, protocol quirks, API limitations, or compatibility behavior.
* Failure modes and why an error is retried, ignored, swallowed, escalated, or transformed.
* Historical context only when the reason cannot be clearly explained without it.

Good comments explain intent, rationale, constraints, and consequences.

## What Comments Are Not For

Avoid comments that:

* Repeat what the code already says.
* Restate a function name, argument name, type, or obvious branch.
* Describe every line of code.
* Explain bad code instead of improving the code.
* Contain vague claims like "temporary," "hack," or "fix later" without a tracking reason.
* Mention outdated behavior.
* Preserve commented-out code.
* Add emotional or apologetic wording.
* Use "we" narration when a direct imperative is clearer.

Bad:

```python
# Loop over users.
for user in users:
    ...
```

Better:

```python
# Process active users first so quota checks fail before expensive enrichment.
for user in active_users:
    ...
```

## Public API Documentation

Public functions, classes, modules, endpoints, commands, schemas, and reusable utilities need documentation when their behavior is not trivial.

Document the contract, not the implementation:

* What it does.
* When to use it.
* Arguments and important constraints.
* Return value.
* Raised errors or failure behavior.
* Side effects.
* Preconditions and postconditions.
* Security or permission requirements.
* Examples when usage is not obvious.

Do not duplicate type information unless the type alone is not enough.

Bad:

```python
def retry(count: int) -> None:
    """Takes count as an int and returns None."""
```

Better:

```python
def retry(count: int) -> None:
    """Retry the operation up to `count` times before surfacing the last error."""
```

## Complex Logic Checklist

For complex logic, document the reasoning at the highest useful level.

Before finishing the code, check whether the reader can answer:

* What problem is this logic solving?
* What invariant must always hold?
* What inputs are valid or invalid?
* What edge cases are intentionally handled?
* What edge cases are intentionally rejected?
* Why is this algorithm or structure used?
* What are the performance expectations?
* What can break if this order changes?
* Is this logic security-sensitive?
* Is this logic coupled to external behavior?
* Which tests protect the documented behavior?

If any answer is not obvious from the code, add a concise comment near the relevant code.

## Invariants and State Machines

For stateful code, always document:

* Valid states.
* Invalid states.
* Allowed transitions.
* Ownership of state changes.
* Cleanup rules.
* Race conditions or ordering requirements.
* What must be true before and after each operation.

Prefer comments close to the data structure or transition function.

## TODO, FIXME, and Technical-Debt Comments

Only keep debt comments when they are actionable.

Use this shape:

```text
TODO(<owner-or-issue>): <specific problem>. Remove when <condition>.
```

Good:

```python
# TODO(#1842): Replace polling with webhook delivery after partner API v2 is enabled.
```

Bad:

```python
# TODO: clean this up later
```

Delete stale TODOs. Do not use comments as a backlog replacement.

## Documentation Layers

Use the right documentation layer:

* Inline comments: local reasoning, invariants, surprising decisions.
* Function/class/module docs: public contract and usage.
* Tests: executable examples of expected behavior.
* README: setup, running, testing, debugging, deployment basics.
* Architecture docs: cross-module design, data flow, major trade-offs, decisions.
* ADRs or decision notes: durable architectural choices and alternatives considered.

Do not put architectural essays inside a small helper function.

## Keep Comments and Docs Synchronized

When code changes, update nearby comments, docstrings, tests, README sections, and architecture docs in the same change.

A stale comment is worse than no comment.

During review, reject or revise comments that are:

* Incorrect.
* Outdated.
* Duplicative.
* Too vague.
* Too far from the code they describe.
* Written in a different style than the surrounding project.
* Explaining behavior not covered by tests.

## Follow Project-Native Style

Use the documentation format already used by the project:

* Python: PEP 257 / project docstring style.
* TypeScript/JavaScript: TSDoc or JSDoc if the project uses it.
* C/C++: Doxygen or the project's existing convention.
* Rust: rustdoc.
* Go: Go doc comments for exported names.
* Django projects: follow existing Django/PEP 8/Black-style conventions.
* Generated docs: keep comments compatible with the generator.

Do not introduce a competing documentation style unless the project explicitly adopts it.

## Review Checklist

Before finishing, verify:

* The code is understandable without unnecessary comments.
* Important "why" comments are present.
* Public APIs have clear contracts.
* Complex logic has invariants, assumptions, and edge cases documented.
* TODO/FIXME comments are specific and trackable.
* Comments do not duplicate obvious code.
* Comments are close to the code they describe.
* Documentation changed together with behavior.
* Tests cover documented behavior.
* Existing project style is respected.
