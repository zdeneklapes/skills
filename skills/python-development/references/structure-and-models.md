# Python Types, Structures, and Models

Read this when designing or reviewing Python types: data models, validation boundaries, interfaces, generic APIs, narrowing, third-party integrations, or typed tests.

## Core Model

Annotations describe a program to a static type checker; they do not normally validate values at runtime. Treat static typing and runtime validation as complementary:

```text
untrusted JSON / YAML / environment / API input
                  │
                  ▼
       validate and convert at the boundary
                  │
                  ▼
  precisely typed, trusted domain values and interfaces
                  │
                  ▼
       typed implementations, fakes, and tests
```

Validate untrusted data once at the boundary. Do not pass raw dictionaries, `Any`, or SDK-specific dynamic objects deep into the application.

Use the Python version supported by the project. Prefer built-in generic syntax (`list[str]`, `dict[str, int]`, `str | None`) when that version supports it; do not modernize syntax past the project's compatibility target.

For newer typing features unavailable on the supported interpreter, use `typing_extensions` when the project permits it instead of maintaining local compatibility copies.

## Choose the Smallest Useful Tool

| Need | Prefer | Notes |
|---|---|---|
| Behavior-rich internal object | Ordinary class | Explicit methods and invariants. |
| Trusted internal record | `dataclass` | No runtime type validation. |
| Internal record with converters or validators | `attrs` | Richer class-generation features. |
| Untrusted config, request, response, or persisted data | Pydantic `BaseModel` | Validation, serialization, schema support. |
| Validate a type without naming a model | Pydantic `TypeAdapter` | Useful for `list[Model]`, unions, and aliases. |
| Plain dictionary with known keys | `TypedDict` | Static key checking only; remains a `dict`. |
| Tiny immutable positional record | `NamedTuple` | Prefer named fields over anonymous tuples. |
| Dependency or capability contract | `Protocol` | Structural typing; implementations need not inherit. |
| Nominal inheritance or shared runtime base | ABC | Use only when runtime inheritance is meaningful. |
| High-throughput validated serialization | `msgspec.Struct` | Consider after profiling and evaluating its trade-offs. |
| Convert dataclasses/attrs at a boundary | `cattrs` | Keep conversion separate from domain types. |
| Distinct semantic identifier | `NewType` | Prevents accidental interchange by the checker. |
| Finite values | `Literal` or an enum | Use enum when runtime behavior or iteration matters. |

Do not use `dict[str, object]`, `dict[str, Any]`, or `list[dict[...]]` for a stable record merely because it is convenient. Give the record a model, `TypedDict`, or a small typed value object.

## Static Types and Runtime Validation

```python
def add_one(count: int) -> int:
    return count + 1


add_one("five")  # A checker reports this; Python does not validate the call.
```

Runtime validation is separate. At a Pydantic boundary, make conversion policy explicit:

```python
from pydantic import BaseModel, ConfigDict, TypeAdapter


class JobRequest(BaseModel):
    model_config = ConfigDict(strict=True, extra="forbid")

    name: str
    retries: int


request = JobRequest.model_validate(raw_request)
requests = TypeAdapter(list[JobRequest]).validate_python(raw_requests)
```

`strict=True` rejects many coercions; `extra="forbid"` rejects unknown fields. They are excellent for strict contracts, but only enable them when the contract really requires that behavior. `validate_assignment=True` is useful for mutable validated models, but immutability or explicit replacement is often clearer.

Use `model_validate(raw)` when the test or code is intentionally exercising parsing. Use direct, typed construction when values are already trusted and the goal is to exercise internal behavior. Use `model_dump()` or the appropriate encoder at the output boundary rather than hand-assembling a supposedly equivalent dictionary.

`Annotated[T, metadata]` attaches metadata while preserving `T` for ordinary static checking. Use it for framework constraints and metadata, not to hide the primary meaning of a value:

```python
from typing import Annotated
from pydantic import Field

PositiveCount = Annotated[int, Field(ge=1)]
```

## Typed Data Objects

### Dataclasses

Use dataclasses for trusted internal values. A good default for an immutable value object is:

```python
from dataclasses import dataclass
from decimal import Decimal
from uuid import UUID


@dataclass(frozen=True, slots=True, kw_only=True)
class SchedulingDecision:
    job_id: UUID
    queue: str
    estimated_cost: Decimal
```

- `slots=True` prevents accidental new attributes and can reduce per-instance overhead.
- `frozen=True` prevents attribute reassignment, not deep mutation of contained lists or dictionaries.
- `kw_only=True` makes call sites durable when fields evolve.
- Use `field(default_factory=list)` or `field(default_factory=dict)` for mutable defaults.

Use a mutable, slotted dataclass for local state that genuinely changes. Do not expect a standard dataclass to validate annotations at runtime.

### attrs

Use `attrs` when an internal domain type benefits from its converters, validators, inheritance support, or class-building ergonomics:

```python
import attrs


@attrs.define(frozen=True, slots=True, kw_only=True)
class VramRequirement:
    gigabytes: int = attrs.field(
        validator=[attrs.validators.instance_of(int), attrs.validators.gt(0)]
    )
```

`attrs.define` enables slots by default. A converter changes incoming values; a validator checks an invariant. Do not choose `attrs` merely to validate untrusted JSON when a boundary-validation library is a better fit.

### Pydantic, msgspec, and cattrs

Choose Pydantic for externally supplied data that needs validation errors, serialization, nested parsing, or schema-oriented behavior. Keep its conversion and mutability settings intentional.

`msgspec.Struct` is a strong option for performance-sensitive serialization and decoding. Validation happens when decoding with a target type; direct construction does not turn every internal call into runtime type checking. It supports JSON, MessagePack, YAML, and TOML. Profile first, then compare error reporting, schema needs, ecosystem support, and team familiarity with Pydantic.

Use `cattrs` when dataclasses or attrs classes should remain framework-independent while a boundary layer structures and unstructures them. Keep conversion policy in that boundary layer.

## Aliases, NewTypes, and Closed Values

An alias improves readability but does not make values distinct:

```python
type QueueName = str  # Python 3.12+
```

For older targets, use `TypeAlias` instead. Use `NewType` when mixing values would be a bug:

```python
from typing import NewType
from uuid import UUID

JobId = NewType("JobId", UUID)
WorkspaceId = NewType("WorkspaceId", UUID)
```

Use a `Literal` alias for a small local set:

```python
from typing import Literal

type RetryMode = Literal["never", "safe", "always"]
```

Use `StrEnum` or `Enum` when values have behavior, need iteration, or are shared broadly:

```python
from enum import StrEnum


class JobStatus(StrEnum):
    PENDING = "pending"
    RUNNING = "running"
    FAILED = "failed"
```

Do not use plain `str` for a known finite domain just to avoid defining a type.

## Class Members and Fluent APIs

Annotate instance fields, including fields initialized in `__init__`, so readers and checkers know the complete object shape:

```python
from typing import ClassVar, Final, Self, final


@final
class Builder:
    DEFAULT_LIMIT: ClassVar[Final[int]] = 10
    name: str

    def __init__(self, name: str) -> None:
        self.name = name

    def with_name(self, name: str) -> Self:
        self.name = name
        return self
```

Use:

- `ClassVar` for attributes that belong to the class rather than each instance.
- `Final` for a name that must not be reassigned.
- `@final` for a class or method that must not be subclassed or overridden.
- `Self` for a method returning the same concrete class.
- `@override` when supported by the project target, to have a checker verify an intended override.

These are static contracts. Enforce runtime restrictions separately only when required.

## Generics, Callbacks, and Overloads

Use generics when a relationship between inputs and outputs matters. On Python 3.12+, PEP 695 syntax is concise:

```python
class Repository[T]:
    def get(self, key: str) -> T | None:
        ...
```

For older targets, use `TypeVar` and `Generic`. Prefer a read-only abstract collection such as `Sequence[T]` for inputs when callers need not provide a mutable `list[T]`; return a concrete collection only when that is part of the promise.

Use `Callable` for simple callbacks. Preserve a decorator's callable signature with `ParamSpec` rather than `Callable[..., object]`:

```python
from collections.abc import Callable
from typing import ParamSpec, TypeVar

P = ParamSpec("P")
T = TypeVar("T")


def traced(function: Callable[P, T]) -> Callable[P, T]:
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> T:
        return function(*args, **kwargs)

    return wrapper
```

Use `@overload` when callers receive meaningfully different return types for different valid argument forms. Put overload declarations first, then one broad implementation. Do not use overloads to conceal a confusing API; consider separate functions or a small tagged result type instead.

## TypedDict and State Unions

Use `TypedDict` for data that must stay a dictionary, such as a wire shape owned by another API:

```python
from typing import NotRequired, ReadOnly, TypedDict


class StatusPayload(TypedDict):
    job_id: ReadOnly[str]
    status: str
    error: NotRequired[str]
```

`TypedDict` does not validate data at runtime and does not create a normal runtime class. Use Pydantic, explicit checks, or another validator before trusting external payloads.

Avoid a record with unrelated optional fields when it actually represents states. Model each valid state and use a union:

```python
from dataclasses import dataclass
from typing import assert_never


@dataclass(frozen=True, slots=True)
class Pending:
    request_id: str


@dataclass(frozen=True, slots=True)
class Running:
    request_id: str
    worker: str


@dataclass(frozen=True, slots=True)
class Failed:
    request_id: str
    reason: str


type JobState = Pending | Running | Failed


def describe(state: JobState) -> str:
    match state:
        case Pending(request_id=request_id):
            return f"{request_id} is pending"
        case Running(worker=worker):
            return f"running on {worker}"
        case Failed(reason=reason):
            return reason
        case _:
            assert_never(state)
```

`Never` and `assert_never()` make a missing union case visible to a checker.

## Protocols and Dependency Boundaries

Prefer a small `Protocol` for a capability or dependency boundary. It uses structural typing: an implementation satisfies the contract by providing compatible members, without inheriting from the protocol.

```python
from collections.abc import Sequence
from dataclasses import dataclass, field
from typing import Protocol


@dataclass(frozen=True, slots=True)
class Job:
    name: str


class JobQueue(Protocol):
    def pull(self, limit: int | None = None) -> Sequence[Job]: ...

    def mark_complete(self, job: Job) -> None: ...


@dataclass(slots=True)
class QueueSpy:
    jobs: list[Job] = field(default_factory=list)
    completed: list[Job] = field(default_factory=list)

    def pull(self, limit: int | None = None) -> list[Job]:
        return self.jobs.copy() if limit is None else self.jobs[:limit]

    def mark_complete(self, job: Job) -> None:
        self.completed.append(job)
```

Inject `JobQueue` into application code. The real client and `QueueSpy` are both checked against the same useful contract. Keep protocols narrow and owned by the consumer; do not mirror an entire SDK.

Use an ABC only when nominal identity, shared implementation, registration, or a real runtime inheritance contract matters. Add `@runtime_checkable` to a protocol only when an `isinstance` check is genuinely required. Such checks verify attribute presence, not full static signature compatibility.

Wrap dynamic or weakly typed third-party libraries in one adapter. Convert their values to application-owned types at that edge. This contains `Any`, broad SDK unions, incomplete stubs, and version-specific details.

For a dependency with weak types, first check for inline annotations and a `py.typed` marker, then for maintained stubs. If neither is adequate, define a narrow local protocol or `.pyi` stub for only the API used. Keep unavoidable `Any` in the adapter and report reusable gaps upstream when practical.

## Narrowing, `Any`, `object`, and `cast`

`Any` turns checking off and propagates through expressions. Use it only at an unavoidable dynamic edge, then validate or narrow immediately.

`object` means an arbitrary Python value. It is safe to receive but cannot be used as a structured record until narrowed:

```python
def render(value: object) -> str:
    if isinstance(value, str):
        return value.upper()
    return repr(value)
```

Do not use `object` as a vague substitute for an omitted record type. Values inferred as `Unknown` by a checker similarly signal missing information; find the untyped dependency or dynamic operation that introduced it.

Use ordinary control flow, `isinstance`, `is not None`, and structural pattern matching before reaching for casts:

```python
state = store.get(job_id)
assert state is not None
```

Use `TypeIs` for a reusable predicate whose narrowed type is compatible with its input and should narrow both branches:

```python
from typing import TypeIs


def is_job_list(value: Job | list[Job]) -> TypeIs[list[Job]]:
    return isinstance(value, list)
```

Use `TypeGuard` when the desired true-branch type is not a subtype of the input type. Its false branch does not receive the same complementary narrowing guarantee.

`cast(T, value)` performs no conversion or runtime check. Keep a cast local to a well-understood seam, such as incomplete third-party stubs after a documented invariant. Do not cast a model mismatch, a union you have not narrowed, or a dynamic test double into correctness.

## Typed Tests and Test Doubles

Type errors in tests often expose erased information, not a limitation of Python typing. Preserve types in test state and fakes.

Use real production models for domain data. Create factories for models that are expensive or verbose to construct. Use `.model_validate(raw)` to test parsing and a direct typed constructor to test already-validated behavior.

Prefer a typed spy or fake to `SimpleNamespace`, `dict[str, object]`, dynamically modified modules, or arbitrary lambda monkeypatches:

```python
@dataclass(frozen=True, slots=True)
class StatusCall:
    job: Job


@dataclass(slots=True)
class QueueSpy:
    completed: list[StatusCall] = field(default_factory=list)

    def mark_complete(self, job: Job) -> None:
        self.completed.append(StatusCall(job))
```

If a replacement is unavoidable, give it the exact callable signature. For mocks of an existing API, use an autospecced mock where the project already uses `unittest.mock`. Prefer injecting a protocol-backed dependency over replacing a private method.

Keep captured values precise:

```python
completed_jobs: list[Job] = []
```

not `list[object]`. Narrow optional values from `.get()` before asserting their fields. Retain a typed reference to an injected spy instead of reaching through a client's private, dynamically typed internals.

## Type Checkers and Type-Level Tests

Choose one static checker as the authoritative CI gate. Configure it intentionally for the project and run it with the formatter, linter, and runtime tests. A second checker can be a compatibility signal, but do not require several checkers with different interpretations to agree without a concrete need.

Require annotations where they add a useful contract, and enable checks for missing generic arguments and untyped dependencies as the project can sustain them. Use a checker-specific, line-level suppression only at a real dynamic seam; include the diagnostic code and a reason. Never use broad ignores or globally relax a rule to hide local mismatches.

Use `assert_type()` to lock down inference for public generic helpers and decorators. Use `reveal_type()` while investigating, then remove it unless the project supports type-check-only tests:

```python
from typing import assert_type

jobs = queue.pull(limit=2)
assert_type(jobs, Sequence[Job])
```

Runtime tests answer whether behavior works. Static checks answer whether operations match declared contracts. Type-level tests answer whether an API infers the intended contract. Use all three when a public typed abstraction is non-trivial.

## Review Checklist

- Does untrusted data validate at a boundary before becoming domain data?
- Is the chosen structure the smallest one that supplies the needed behavior?
- Are finite values, semantic identifiers, and state variants represented explicitly?
- Do function signatures preserve useful generic, optional, and callback relationships?
- Is a `Protocol` narrower and clearer than an SDK type or large ABC?
- Does `Any`, `object`, `Unknown`, or `cast` remain localized and justified?
- Do tests use real models and typed fakes instead of erasing type information?
- Are checker suppressions narrow, named, and unavoidable?
