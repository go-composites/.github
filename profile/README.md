<p align="center"><img src="https://raw.githubusercontent.com/go-composites/brand/main/social/golang-oop.png" alt="go-composites" width="720"></p>

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
| [`array`](https://github.com/go-composites/array) | `github.com/go-composites/array/src` | Interface-first slice with `Push`/`Pop`/`First`/`Fetch`/`Each`; every method returns a `result`. |
| [`null`](https://github.com/go-composites/null) | `github.com/go-composites/null/src` | The null sentinel value used as the default `result` payload. |
| [`result`](https://github.com/go-composites/result) | `github.com/go-composites/result/src` | Wraps a `payload` plus an `error`, built with functional options. |
| [`is`](https://github.com/go-composites/is) | `github.com/go-composites/is/src` | Predicate helpers for asking questions about composites. |

### Invariant enforcers (static analyzers)

| Repo | Role |
| --- | --- |
| [`nonnil`](https://github.com/go-composites/nonnil) | A `go vet`-style analyzer that enforces the **Null-Object** invariant — fails the build when an interface with `IsNull()` is returned, assigned, stored in a struct field or map value as a bare `nil`. Runs in CI on every repo. |

## Conventions

- Module path: `module github.com/go-composites/<repo>`.
- Pure Go, `CGO_ENABLED=0`, builds and tests on all supported architectures.
- **100% statement coverage** on library (`src`) packages — an org hard rule.
- Licensed **BSD-3-Clause**.

## Docs & landing

- Landing page: <https://go-composites.github.io/>
- Documentation: <https://go-composites.github.io/docs/>
