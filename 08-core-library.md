# Core Library

## Purpose

The core library is the small language-adjacent library layer that every Vek program depends on.

Conceptually it lives in the `core` package, with declarations grouped into modules such as:

- `core:traits`
- `core:enums`
- `core:panic`

Unlike the broader standard library, the public core surface is available globally in every module without an explicit import.

## Global Prelude

The public declarations from the core library are part of the global prelude.

That means user code may write:

```vek
fn same<T: Equal<T>>(a: T, b: T) -> bool {
  return a.equals(b);
}
```

without importing `Equal`.

Likewise, user code may refer to:

- `panic`
- `Result`
- `Ordering`
- the core traits listed below

without explicit imports.

The global-prelude rule applies only to the public core library surface. It does not imply that arbitrary stdlib symbols are global.

## `core:traits`

The core traits are:

```vek
trait Equal<T> {
  fn equals(self, other: T) -> bool;
}

trait Hashable {
  fn hash(self) -> u64;
}

trait Ordered<T> {
  fn compare(self, other: T) -> Ordering;
}

trait Cloneable {
  fn clone(self) -> Self;
}

trait Iterable<T> {
  fn next(mut self) -> T?;
}

trait Defaultable {
  fn default() -> Self;
}

trait Formattable {
  fn format(self) -> string;
}

trait Unwrappable<T> {
  fn unwrap(self) -> T;
}
```

Notes:

- `Equal<T>` is the core equality trait used by user-defined `==`
- `Ordered<T>` powers explicit ordering comparisons through `compare(...)`
- `Iterable<T>` is the trait used by `for item in value { ... }`
- `Formattable` is the core explicit string-conversion trait
- core primitive numerics, `bool`, and `string` satisfy `Formattable`

## `core:enums`

The core enums are:

```vek
enum Ordering {
  Less;
  Equal;
  Greater;
}

enum Result<T, E> {
  Ok(T);
  Err(E);

  satisfies Unwrappable<T> {
    fn unwrap(self) -> T {
      match self {
        Ok(v) => return v,
        Err(_) => panic("unwrap on Err"),
      }
    }
  }
}
```

`Result<T, E>` is the standard explicit error carrier used by fallible APIs.

## `core:panic`

The core panic surface is:

```vek
fn panic(message: string) -> void;
```

`panic` is for unrecoverable failure paths.

Calling `panic` terminates normal control flow and transfers to the implementation-defined panic path provided by the minimal runtime.

## Relationship to the Compiler

The core library should be treated as ordinary Vek code as much as possible.

Implementations may still give special treatment to parts of the core prelude where needed for bootstrapping, operator semantics, or early frontend bring-up, but that is an implementation constraint, not a user-facing language distinction.
