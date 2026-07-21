---
name: python-development
description: Use when writing, reviewing, refactoring, or testing Python code; defining Python exceptions; or applying Python naming, branching, or pytest conventions. Do not use for non-Python changes.
---

# Python Development

Use this for reusable Python coding and testing conventions.

Honor the project's supported Python version when choosing syntax or standard-library typing features.

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

## Testing Rules

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

### Pytest Fixture Side Effects

Use `@pytest.mark.usefixtures()` for fixtures used only for side effects.

Use a normal fixture parameter only when the test reads the fixture return value.

```python
@pytest.mark.usefixtures("mock_cuda")
def test_callback_behavior():
    ...
```

This avoids unused-argument lint errors and makes fixture intent clear.
