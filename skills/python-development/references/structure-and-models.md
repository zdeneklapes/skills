# Python Structure and Models

Read this when choosing between Pydantic models, Python dataclasses, `TypedDict`, or plain classes; reviewing API/config/cache/stats schemas; or deciding where validation, serialization, JSON contracts, and internal runtime state should live.

## Core Rule

Use both Pydantic models and dataclasses.

- Use Pydantic `BaseModel` at system boundaries where data is parsed, validated, serialized, deserialized, stored, sent, received, or treated as a stable contract.
- Use standard-library `dataclass` for internal runtime state, temporary grouping objects, simple helper returns, and mutable state machines that never need JSON schema or external validation.

Dataclasses mainly reduce boilerplate by generating methods such as `__init__`, `__repr__`, and comparisons. Standard dataclasses do not deeply validate field types at runtime.

Pydantic `BaseModel` provides runtime validation, serialization, `model_validate`, `model_dump`, and JSON-schema-related functionality.

## Decision Table

| Use case | Better choice | Why |
|---|---|---|
| API request or response models | Pydantic | External data can be malformed and needs useful validation errors. |
| Config models | Pydantic | Config often comes from YAML, env vars, files, or APIs. |
| Cache payloads | Pydantic | Saved and loaded data needs validation and serialization. |
| Exported stats or backend payloads | Pydantic | Stable JSON contracts matter. |
| Validation issues and errors | Pydantic | Structured records may be stored, returned, or sent elsewhere. |
| Internal mutable state machines | `dataclass` | No validation, schema, or serialization needed. |
| Temporary grouped pipeline state | `dataclass` | Cleaner than scattering many `self.*` attributes. |
| Pure helper return objects | `dataclass` | Simple, fast, readable internal containers. |
| Immutable internal constants or metadata | `dataclass(frozen=True)` | Prefer this unless schema or serialization is required. |
| Immutable serialized records | Pydantic frozen model | Use when validation/schema/dump behavior still matters. |

## Default Workflow

1. Classify whether the object crosses a boundary.
   Boundary objects include API payloads, config, cache files, exported stats, validation issues, serialized records, process/module contracts, and data from untrusted or loosely typed sources.

2. Use Pydantic for boundary objects.
   Prefer explicit fields, validators where needed, and `model_validate`/`model_dump` at parse and serialization points.

3. Use dataclasses for internal runtime structures.
   Prefer `@dataclass(slots=True)` for mutable internal state and `@dataclass(frozen=True, slots=True)` for immutable internal metadata.

4. Freeze Pydantic records when mutation is not part of the contract.
   Use `ConfigDict(frozen=True)` and update with `model_copy(update={...})` instead of mutating fields across a pipeline.

5. Do not choose Pydantic or bypass validation because of performance guesses.
   Pydantic is usually not the bottleneck; profile before adding special construction paths or replacing clear validation.

## Preferred Patterns

### Boundary Record

```python
from pydantic import BaseModel, ConfigDict


class DatasetPreparationCounts(BaseModel):
    model_config = ConfigDict(frozen=True)

    source_samples: int = 0
    usable_canonical_samples: int = 0
    final_train: int = 0
    final_validation: int = 0
    final_test: int = 0


counts = counts.model_copy(update={"final_train": len(train_samples)})
```

Use this for stats, cache payloads, config, API contracts, and records that may be persisted or sent outside the current runtime path.

### Internal Runtime State

```python
from dataclasses import dataclass


@dataclass(slots=True)
class _TurnContext:
    """Mutable state for one agentic user turn."""

    state: _TurnState = _TurnState.EXPECT_USER
    pending_thought: VLMSampleStepResponse | None = None
    open_tool_call_id: str | None = None
    last_step: VLMSampleStepResponse | None = None
```

Use this for internal-only mutable validator state, pipeline scratch state, grouped in-memory objects, and simple helper return values.

## Common Mistakes

- Do not use Pydantic just to get a convenient constructor or repr for internal objects. Use `dataclass`.
- Do not use a plain dataclass for data loaded from JSON, YAML, env vars, API responses, cache files, or user input unless another layer already validates it.
- Do not keep `arbitrary_types_allowed=True` on a Pydantic model just to store internal runtime objects. That is usually a sign the type should be a dataclass.
- Do not use Pydantic dataclasses as a default compromise. They are useful in specific cases, but they are not a replacement for `BaseModel`.
- Do not mutate Pydantic stats/config records throughout a pipeline when a frozen model and `model_copy(update=...)` would make the contract clearer.

## Review Checklist

- Is this object parsed from or dumped to an external format?
- Can malformed input reach this type?
- Does this type need `model_validate`, `model_dump`, schema generation, or structured validation errors?
- Is this just internal mutable state for an algorithm, validator, or pipeline step?
- Would `@dataclass(slots=True)` remove Pydantic config such as `arbitrary_types_allowed=True`?
- If the record is Pydantic, should it be frozen?
- If the record is a dataclass, is runtime validation intentionally unnecessary?
