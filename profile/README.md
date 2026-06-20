<p align="center"><img src="https://raw.githubusercontent.com/go-composites/brand/main/social/go-composites.png" alt="go-composites" width="720"></p>

# go-composites

**Composition-Oriented Programming for Go.**

`go-composites` is a family of small, hand-written building blocks for Go that
model values as **composites** — types assembled by *composition* and *interfaces*,
not by class hierarchies or inheritance. There are no base classes to extend and no
`super` calls; behaviour is provided by embedding small pieces together and
programming against narrow interfaces.

## What "Composition-Oriented" means here

- **Interface-first.** Every component exposes an exported `Interface`. Callers
  depend on the interface, never on the concrete struct. The struct (`data`) is
  unexported and constructed through a `New(...)` function with functional options.
- **Composition over inheritance.** Richer behaviour is built by holding and
  delegating to smaller composites, not by subclassing. A `String` *has* the
  pieces it needs; it does not *extend* a parent.
- **No naked objects, no naked errors.** Two invariants run through every repo:

### The Null-Object safety pattern

Constructors and methods **never return a bare `nil`** for an interface. Each
interface that can be "empty" carries an `IsNull() bool` method, and there is a
real null-object implementation that answers `true`. Calling a method on an
"absent" value is always safe — you get a well-behaved null object instead of a
panic. Real values answer `IsNull() bool { return false }`.

### Result-based error handling

Fallible operations don't return `(value, error)` pairs and never panic. They
return a **`Result`** — a composite that wraps either a payload or an `error`
(itself a null-object-friendly composite). Division by zero, a missing key, or a
parse failure produces a `Result` whose `HasError()` is `true`, carrying an
`Error` built with `Error.New("...")`. Success carries the payload via
`Result.WithPayload(x)`.

```go
import (
    Array  "github.com/go-composites/array/src"
    Result "github.com/go-composites/result/src"
)

a   := Array.New()
r   := a.Fetch(0)        // out of range → a Result, not a panic
if r.HasError() {
    // handle r.Error(), which is never nil
}
```

## The components

| Repo | Module path | What it is |
| --- | --- | --- |
| [`base`](https://github.com/go-composites/base) | `github.com/go-composites/base/src` | The shared foundation every composite builds on. |
| [`boolean`](https://github.com/go-composites/boolean) | `github.com/go-composites/boolean/src` | A boxed boolean composite. |
| [`error`](https://github.com/go-composites/error) | `github.com/go-composites/error/src` | The `Error` interface (`Message`/`IsNull`) plus null and `method-not-implemented` sentinels. |
| [`string`](https://github.com/go-composites/string) | `github.com/go-composites/string/src` | A boxed string value with `Set`/`ToGoString`/`Split`. |
| [`number`](https://github.com/go-composites/number) | `github.com/go-composites/number/src` | A boxed numeric value (int/float) with arithmetic that returns a `result` (division by zero → an error, never a panic). |
| [`array`](https://github.com/go-composites/array) | `github.com/go-composites/array/src` | Interface-first slice with `Push`/`Pop`/`First`/`Fetch`/`Each` **plus combinators** `Map`/`Filter`/`Reduce`/`Find`/`Any`/`All`; every method returns a `result`. |
| [`dictionary`](https://github.com/go-composites/dictionary) | `github.com/go-composites/dictionary/src` | An interface-first key/value map; `Get` of a missing key returns a `result` error, never a panic. |
| [`set`](https://github.com/go-composites/set) | `github.com/go-composites/set/src` | An unordered unique collection: `Add`/`Delete`/`Has`/`Union`/`Intersection`/`Difference`/`IsSubset` + Null-Object. |
| [`range`](https://github.com/go-composites/range) | `github.com/go-composites/range/src` | A numeric interval (inclusive `..` or exclusive `...`, stepped): `Includes`/`Each`/`ToArray`; `New` returns a `result`. |
| [`pair`](https://github.com/go-composites/pair) | `github.com/go-composites/pair/src` | A two-element heterogeneous grouping (`First`/`Second`/`ToArray`) — the natural key/value entry. |
| [`symbol`](https://github.com/go-composites/symbol) | `github.com/go-composites/symbol/src` | An interned, immutable identifier (Ruby-style `:name`): `New("x") == New("x")`. |
| [`time`](https://github.com/go-composites/time) | `github.com/go-composites/time/src` | A `Time` (+ `Duration` subpackage) composite; `Parse` returns a `result`, `Before`/`After`/`Add`/`Sub`, never a panic. |
| [`proc`](https://github.com/go-composites/proc) | `github.com/go-composites/proc/src` | A first-class callable: `Call`/`Then` (railway-style), short-circuits on an error `result`. |
| [`enumerator`](https://github.com/go-composites/enumerator) | `github.com/go-composites/enumerator/src` | A **lazy** sequence (Ruby `Enumerator::Lazy`): lazy `Map`/`Filter`/`Take` over a finite or **infinite** source, with `result`-based `ToArray`/`Each`/`First`/`Reduce` terminals. |
| [`null`](https://github.com/go-composites/null) | `github.com/go-composites/null/src` | The null sentinel value used as the default `result` payload. |
| [`result`](https://github.com/go-composites/result) | `github.com/go-composites/result/src` | Wraps a `payload` plus an `error`, built with functional options. |
| [`compose`](https://github.com/go-composites/compose) | `github.com/go-composites/compose/src` | `Pipe`/`Run` compose `result`-returning steps into a single left-to-right, short-circuiting pipeline (`Then`/`Map`/`Recover`/`Fail`). |
| [`composites`](https://github.com/go-composites/composites) | `github.com/go-composites/composites` | **Meta-package**: one import re-exporting the whole vocabulary (Array, Boolean, String, Number, Dictionary, Set, Range, Pair, Symbol, Time, Proc, Enumerator, Result, Error, Null) as type aliases + constructors. |
| [`typed`](https://github.com/go-composites/typed) | `github.com/go-composites/typed/src/...` | A **generics** parallel track: `Result[T]`, `Optional[T]`, `Slice[T]` — the same patterns with compile-time type safety (payload-type bugs become build errors). Complementary to, not a replacement for, the dynamic composites. |

### Invariant enforcers (static analyzers)

| Repo | Role |
| --- | --- |
| [`nonnil`](https://github.com/go-composites/nonnil) | A `go vet`-style analyzer that enforces the **Null-Object** invariant — fails the build when an interface with `IsNull()` is returned, assigned, stored in a struct field or map value as a bare `nil`. Runs in CI on every repo. |
| [`respondto`](https://github.com/go-composites/respondto) | A `go vet`-style analyzer for **reflective dispatch** — flags `RespondTo`/method-name calls whose target method does not exist, catching typo'd dynamic sends at build time. |

## Conventions

- Module path: `module github.com/go-composites/<repo>`.
- Pure Go, `CGO_ENABLED=0`, builds and tests on all supported architectures.
- **100% statement coverage** on library (`src`) packages — an org hard rule.
- Licensed **BSD-3-Clause**.

## Docs & landing

- Landing page: <https://go-composites.github.io/>
- Documentation: <https://go-composites.github.io/docs/>
