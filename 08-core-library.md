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
  fn unwrap_or(self, default: T) -> T;
  fn is_some(self) -> bool;
  fn is_none(self) -> bool;
}
```

Notes:

- `Equal<T>` is the core equality trait used by user-defined `==`
- `Ordered<T>` powers explicit ordering comparisons through `compare(...)`
- `Iterable<T>` is the trait used by `for item in value { ... }`
- `Formattable` is the core explicit string-conversion trait

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
      return match self {
        Ok(value) => value,
        Err(_) => panic("called unwrap on an Err value"),
      };
    }

    fn unwrap_or(self, default: T) -> T {
      return match self {
        Ok(value) => value,
        Err(_) => default,
      };
    }

    fn is_some(self) -> bool {
      return match self {
        Ok(_) => true,
        Err(_) => false,
      };
    }

    fn is_none(self) -> bool {
      return match self {
        Ok(_) => false,
        Err(_) => true,
      };
    }
  }
}
```

`Result<T, E>` is the standard explicit error carrier used by fallible APIs.

## `core:panic`

The core panic surface is:

```vek
extern fn panic(message: string) -> void;
```

`panic` is for unrecoverable failure paths.

Calling `panic` terminates normal control flow and transfers to the implementation-defined panic path provided by the minimal runtime.

## Built-in Satisfactions

The following satisfactions are provided implicitly by the language without a user-written `satisfies` block. They are part of the core specification.

### Primitives (integers, floats)

Integer types (`i8`, `i16`, `i32`, `i64`, `u8`, `u16`, `u32`, `u64`) and float types (`f32`, `f64`) satisfy:

- `Equal<T>` — value equality
- `Hashable` — hash of the bit pattern
- `Ordered<T>` — numeric ordering
- `Cloneable` — trivial copy
- `Defaultable` — default value is `0`
- `Formattable` — decimal string representation

### `bool`

`bool` satisfies `Equal<bool>`, `Hashable`, `Cloneable`, `Defaultable` (default `false`), and `Formattable`. `bool` does **not** satisfy `Ordered<bool>` — boolean values have no total ordering.

### `string`

`string` satisfies `Equal<string>`, `Hashable`, `Ordered<string>` (lexicographic), `Cloneable`, `Defaultable` (default `""`), and `Formattable`.

### `Array<T>`

`Array<T>` satisfies:

- `Iterable<T>` — always
- `Formattable` — when `T: Formattable`
- `Cloneable` — when `T: Cloneable`
- `Defaultable` — always; default is `[]`

### Tuples

A tuple `(T1, T2, ...)` satisfies:

- `Equal<(T1, T2, ...)>` — when every `Ti: Equal<Ti>` (element-wise)
- `Hashable` — when every `Ti: Hashable`
- `Formattable` — when every `Ti: Formattable`
- `Cloneable` — when every `Ti: Cloneable`

### Nullable `T?`

`T?` satisfies:

- `Equal<T?>` — when `T: Equal<T>`
- `Formattable` — when `T: Formattable`
- `Unwrappable<T>` — always; `unwrap` panics on `null`, `unwrap_or` returns the default on `null`, `is_some` and `is_none` check for null

### `Ordering`

`Ordering` satisfies `Equal<Ordering>`, `Hashable`, and `Formattable`.

### `Result<T, E>`

`Result<T, E>` satisfies `Unwrappable<T>` via its explicit `satisfies` block above. It additionally satisfies `Formattable` when `T: Formattable` and `E: Formattable`.

## Relationship to the Compiler

The core library should be treated as ordinary Vek code as much as possible.

Implementations may still give special treatment to parts of the core prelude where needed for bootstrapping, operator semantics, or early frontend bring-up, but that is an implementation constraint, not a user-facing language distinction.
