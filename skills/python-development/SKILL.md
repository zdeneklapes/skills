---
name: python-development
description: Use when writing, reviewing, refactoring, or testing Python code; defining Python exceptions; or applying Python naming, branching, or pytest conventions. Do not use for non-Python changes.
---

# Python Development

Reusable Python code and test rules.

Target Python version limits syntax and standard-library typing.

## Types, Structures, and Models

For types, models, validation, adapters, and typed tests: read [structure-and-models.md](references/structure-and-models.md).

## Exceptions

Existing exception layout first. Else nearest domain/package `exceptions.py`.

No reusable exceptions inside workflow, loader, client, factory, type, or utility modules.

## Branching

Use `match`/`case` when supported and structural branches clearer. Keep priority guards as `if`.

## Reuse Results

Call once. Store descriptive variable. Reuse.

## Variable Names

Prefer descriptive names: `model_identifier`, `configuration`, `parameters`, `environment`; not `model_id`, `cfg`, `params`, `env`. Keep third-party schema names.

## Testing Rules

### Pytest

- Test core and public behavior. Skip incidental import, cache, and log internals; no absent behavior.
- Config loading: temporary `tmp_path` files. Runtime: production config objects. Colocate repeated fixture shapes.
- Side-effect fixture: `@pytest.mark.usefixtures()`. Normal parameter only when test reads return value.

```python
@pytest.mark.usefixtures("mock_cuda")
def test_callback_behavior():
    ...
```

Avoid unused fixture arguments. Show side-effect intent.
