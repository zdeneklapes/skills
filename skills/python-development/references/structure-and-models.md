# Python Types, Structures, and Models

Use for data models, validation boundaries, interfaces, generics, narrowing, adapters, typed tests.

## Core

Annotations guide static checkers. Usually no runtime validation. Validate untrusted JSON, YAML, environment, API, cache input once at boundary. Internal values stay precise; no raw dictionaries, `Any`, or SDK dynamic objects deep inside.

Target Python version limits syntax. Prefer `list[str]`, `dict[str, int]`, `str | None` when supported. Use `typing_extensions` for newer features only when project permits it.

## Pick Tool

| Need | Use |
|---|---|
| Behavior-rich internal object | Ordinary class |
| Trusted record | `dataclass` |
| Record with converters/validators | `attrs` |
| Untrusted boundary data | Pydantic `BaseModel` |
| Validate unnamed type | Pydantic `TypeAdapter` |
| Fixed dictionary shape | `TypedDict` |
| Tiny immutable positional record | `NamedTuple` |
| Capability/dependency contract | `Protocol` |
| Runtime inheritance contract | ABC |
| High-throughput typed decode | `msgspec.Struct` |
| Convert dataclass/attrs boundary data | `cattrs` |
| Distinct semantic ID | `NewType` |
| Finite values | `Literal` or enum |

Stable record: no `dict[str, object]`, `dict[str, Any]`, `list[dict[...]]`. Use model, `TypedDict`, or value object.

## Boundary Validation

```python
from pydantic import BaseModel, ConfigDict, TypeAdapter


class JobRequest(BaseModel):
    model_config = ConfigDict(strict=True, extra="forbid")
    name: str
    retries: int


request = JobRequest.model_validate(raw_request)
requests = TypeAdapter(list[JobRequest]).validate_python(raw_requests)
```

- `strict=True`: reject coercion. `extra="forbid"`: reject unknown fields. Enable only when contract needs it.
- `validate_assignment=True`: mutable model checks. Frozen model or replacement often clearer.
- `model_validate(raw)`: parsing test/boundary. Direct typed constructor: trusted internal values. `model_dump()`: output boundary.
- `Annotated[T, metadata]`: framework metadata; `T` stays static meaning.

```python
from typing import Annotated
from pydantic import Field

PositiveCount = Annotated[int, Field(ge=1)]
```

## Data Objects

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

- `slots=True`: block accidental fields; less instance overhead.
- `frozen=True`: block reassignment, not deep mutation.
- `kw_only=True`: durable call sites. Mutable default: `field(default_factory=list)` or `dict`.
- Mutable local state: slotted dataclass. Standard dataclass does not runtime-validate annotations.

`attrs`: use converters, validators, richer class support. `@attrs.define` defaults `slots=True`.

Pydantic: validation errors, nested parse, serialization, schema. `msgspec.Struct`: fast JSON, MessagePack, YAML, TOML decode; target-type decoding validates, direct construction does not. Profile before switching. `cattrs`: structure/unstructure dataclass or attrs values without binding domain types to validation framework.

## Names, Values, Members

```python
from enum import StrEnum
from typing import ClassVar, Final, NewType, Self, final
from uuid import UUID

type QueueName = str  # Python 3.12+; older target: TypeAlias
JobId = NewType("JobId", UUID)


class JobStatus(StrEnum):
    PENDING = "pending"
    RUNNING = "running"


@final
class Builder:
    LIMIT: ClassVar[Final[int]] = 10

    def with_name(self, name: str) -> Self: ...
```

Alias improves readability; `NewType` blocks accidental mixing. `Literal` fits small local value sets; enum fits shared values, behavior, iteration. `ClassVar`: class field. `Final`: no reassignment. `@final`: no subclass/override. `Self`: same concrete class. `@override`: checker verifies override when target supports it.

## Generics, Callbacks, States

Use generic type when input/output relation matters. Python 3.12+: `class Repository[T]: ...`; older targets: `TypeVar`/`Generic`. Accept `Sequence[T]` when mutation irrelevant; return concrete collection only when promised.

Preserve decorator signatures with `ParamSpec`, not `Callable[..., object]`:

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

`@overload`: different valid input forms, different return types. Declarations first; one broad implementation. Confusing overloads: split API or use tagged result.

`TypedDict` describes dictionary keys only; no runtime validation. Use `NotRequired`, `ReadOnly`, `Required` as needed. State machine: model each valid state, union them, `match`, end unreachable arm with `assert_never()`. `Never`/`assert_never()` expose missing variants.

## Protocols, Adapters, Third Parties

```python
from collections.abc import Sequence
from dataclasses import dataclass
from typing import Protocol


@dataclass(frozen=True, slots=True)
class Job:
    name: str


class JobQueue(Protocol):
    def pull(self, limit: int | None = None) -> Sequence[Job]: ...
    def mark_complete(self, job: Job) -> None: ...
```

Protocol structural: compatible members, no inheritance. Keep consumer-owned contract narrow. Production client and typed spy implement same contract. ABC only for nominal identity, shared code, registration, or runtime inheritance. `@runtime_checkable` only for required `isinstance`; checks attribute presence, not full signatures.

Wrap dynamic SDK once. Convert SDK values to application-owned types there. Weak dependency types: check inline annotations/`py.typed`, then maintained stubs, then narrow local protocol or `.pyi`. Localize unavoidable `Any`; report reusable gaps upstream.

## Narrowing

- `Any`: checking off; dynamic edge only, then validate/narrow.
- `object`: arbitrary value; narrow before structured use. `Unknown`: missing type information; find source.
- Prefer `isinstance`, `is not None`, pattern matching. `cast()` does no conversion/check; local invariant or weak stub only.

```python
from typing import TypeIs


def is_job_list(value: Job | list[Job]) -> TypeIs[list[Job]]:
    return isinstance(value, list)
```

`TypeIs`: compatible narrowed type; narrows true and false paths. `TypeGuard`: true-path type need not subtype input; false path no complementary guarantee.

## Typed Tests and Checks

Real production models first. Verbose model: factory. `.model_validate(raw)` tests parsing; direct typed constructor tests trusted behavior. Typed spy/fake beats `SimpleNamespace`, `dict[str, object]`, dynamic modules, arbitrary lambda monkeypatches. Replacement must match exact signature. Prefer injected protocol over private-method patch. `create_autospec()` when project uses `unittest.mock`.

Typed captures: `completed_jobs: list[Job] = []`, not `list[object]`. Narrow optional `.get()` result before fields. Keep direct typed spy reference; no private dynamic-client assertions.

One checker gates CI. Optional second checker: compatibility signal, not competing gate. Require useful annotations; enable missing generic/untyped dependency checks when sustainable. Suppress only named line-level dynamic seam, with reason. No broad ignores/global relaxation.

```python
from typing import assert_type

assert_type(queue.pull(limit=2), Sequence[Job])
```

`assert_type()`: lock public inference. `reveal_type()`: investigate, then remove unless type-test suite allows it. Runtime tests check behavior; checker checks contracts; type tests check inference.

## Review

- Boundary validates untrusted input?
- Smallest useful structure?
- Finite values, IDs, states explicit?
- Generic/callback/optional relation preserved?
- Protocol narrower than SDK/ABC?
- `Any`, `object`, `Unknown`, `cast` local and justified?
- Tests use real models, typed fakes?
- Suppressions narrow, named, unavoidable?
