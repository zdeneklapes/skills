---
name: python-development
description: Use when writing, reviewing, refactoring, or testing Python code; defining Python exceptions; or applying Python naming, branching, pytest, tooling-discovery, or logging conventions. Do not use for non-Python changes.
---

# Python Development

Use this for reusable Python coding and testing conventions.

First follow the project's own instructions for Python version, execution environment, dependency manager, framework, formatter, linter, type checker, test runner, and command wrappers.

## Types, Structures, and Models

For type design, data structures, validation, type checking, adapters, and typed tests, read [structure-and-models.md](references/structure-and-models.md).

## Exceptions

Follow the project's existing exception-placement convention.

If no convention exists, define custom exception classes in the nearest domain/package `exceptions.py`.

Avoid defining reusable exception classes inside workflow, loader, API client, factory, type, or utility modules.

## Branching

Use `match`/`case` when the project supports Python 3.10+ and it makes structural branching clearer.

Keep simple priority-ordered guard clauses as `if` statements.

## Reuse Results

If the same function result is needed more than once, call it once, store it in a descriptive variable, and reuse it.

## Variable Names

Prefer descriptive names over short abbreviations.

Examples:

```text
hugging_face_access_token instead of hf_token
model_identifier instead of model_id
configuration instead of cfg
parameters instead of params
environment instead of env
```

Exception: keep external API names when matching third-party schemas.

## Comments With References

When adding comments that explain complex problems or non-obvious fixes, include useful URLs close to the implementation.

Use:

```python
# See: https://example.com/relevant-docs
# NOTE: Problem described in: https://example.com/issue
```

Do not add comments that only repeat obvious code.

## Testing Rules

Use the project's test runner and test helpers.

For pytest projects, apply the pytest-specific rules in this section.

### Test Real Behavior

Test core logic and user-visible behavior.

Do not test incidental implementation mechanics such as import side effects, `sys.modules`, import cache state, exact log text, or whether a transitive import happened.

Do not add tests for behavior that is not implemented, removed, or absent from the public surface.

### Config Fixtures

Do not make tests depend on operational example config files.

When a test needs config data:

- Create temporary config files with `tmp_path` for config-loading behavior.
- Build production config objects directly for runtime behavior.
- Use colocated fixtures only when several tests need the same static shape.

### Standalone Scripts

Do not write full unit tests for standalone operational scripts.

Verify scripts with formatting, linting, type checks, CLI help, dry-run checks, or small smoke checks.

### Pytest Fixture Side Effects

Use `@pytest.mark.usefixtures()` for fixtures used only for side effects.

Use a normal fixture parameter only when the test reads the fixture return value.

```python
@pytest.mark.usefixtures("mock_cuda")
def test_callback_behavior():
    ...
```

This avoids unused-argument lint errors and makes fixture intent clear.

### Logging Tests

Do not test logging internals unless log output is a documented user, audit, compliance, or integration contract.

Do not assert emitted log text, log formatting, log levels, logger names, or logger internals when logs are only diagnostics.

Tests may stub logging only to suppress noise while asserting real behavior.

## Code Quality

Use the project's configured execution, formatting, linting, and type-checking tools.

### Python Execution Environment

Find the project's Python execution environment before running Python commands.

Prefer project-provided commands and docs first: `just`, `make`, scripts, `tox`, `nox`, Docker/Compose, CI config, `README.md`, `AGENTS.md`, or nested agent instructions.

Use the environment manager already present in the project, such as `uv`, Poetry, Hatch, PDM, `tox`, `nox`, an activated virtualenv, container commands, or system Python.

Do not introduce or bypass an environment manager just for convenience.

### Linting and Formatting

Use the project's configured formatter and linter.

Use project-specific wrapper commands when the repository defines them.

### Logging Policy

Use the project's logging levels and logger namespaces.

Use `INFO` for operator-relevant lifecycle events, such as before processing starts or when entering a long-running state.

Completion summaries, counts, metrics, config dumps, environment dumps, sample details, and diagnostics should use lower-verbosity levels when the project supports them.

Do not invent manual logging categories when standard logging levels and logger namespaces are enough.
