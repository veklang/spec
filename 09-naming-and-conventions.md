# Naming and Conventions

This chapter defines the default naming and casing conventions for Vek code and the standard library.

These conventions are intended to keep code readable and predictable. They are conventions, not semantic rules, unless a later section says otherwise.

## General Principle

Vek naming should be simple, boring, and consistent.

- prefer descriptive names over abbreviations
- prefer established words over clever naming
- keep related APIs visually consistent

## Type Surface Conventions

When both a sugared surface form and a generic runtime-shaped form are available, normal user code should prefer the sugared surface form.

For arrays, prefer postfix `[]` syntax over `Array<T>`.

Examples:

```vek
let nums: i32[] = [1, 2, 3];
let matrix: i32[][] = [[1, 2], [3, 4]];
```

Prefer:

```vek
fn push_one(mut xs: i32[]) -> void { ... }
```

over:

```vek
fn push_one(mut xs: Array<i32>) -> void { ... }
```

## Casing

### Types

Use `PascalCase` for:

- types
- enums
- traits
- type aliases
- generic type parameter names
- enum variants

Examples:

```vek
struct UserProfile {}
enum ParseResult {}
trait Displayable {}
type UserId = i32;
fn parse<T>(value: T) -> void {}
```

### Nullability

Prefer postfix `?` for nullable types.

Examples:

```vek
let maybe_count: i32? = null;
let maybe_user: User? = null;
```

### Values

Use `snake_case` for:

- variables
- functions
- methods
- parameters
- local bindings
- module-level constants
- fields

Examples:

```vek
const default_port = 5432;

fn parse_user_name(input: string) -> string {
  let user_name = input.trim();
  return user_name;
}

struct User {
  display_name: string;
}

enum Result<T, E> {
  Ok(T);
  Err(E);
}
```

## Modules and Files

Module names and source files should use `snake_case`.

Examples:

- `http_server.vek`
- `string_utils.vek`
- `std:io`
- `std:math/matrix`

Use directory structure to express hierarchy instead of long compound module names where practical.

Package names should also use `snake_case`.

Examples:

- `std`
- `my_project`
- `http_tools`

## Methods

Methods follow the same `snake_case` naming convention as functions.

Examples:

```vek
struct User {
  fn display_name(self) -> string { ... }
  fn set_display_name(mut self, name: string) -> void { ... }
  fn from_name(name: string) -> Self { ... }
}
```

### Instance vs Static Methods

- methods with `self` are instance methods
- methods without `self` are static methods

Static methods should usually be named like normal functions, not with special prefixes.

Examples:

- `User.new(...)`
- `User.from_name(...)`
- `User.parse(...)`

`new` is a common convention for constructor-like static methods, but it is not required.

### Returning the Enclosing Type

Methods may return either the concrete enclosing type name or `Self`.

Both of these are valid:

```vek
struct User {
  fn new(name: string) -> User {
    return Self { name };
  }
}
```

```vek
struct User {
  fn new(name: string) -> Self {
    return Self { name };
  }
}
```

`Self` is generally preferred and should be treated as the normal convention for methods that return their enclosing type.

## Constants

Vek uses `snake_case` for constants as well.

```vek
const max_connections = 1024;
const http_ok = "200 OK";
```

There is no separate all-caps constant naming convention in the standard style.

## Internal Symbols

Compiler or stdlib implementation-facing symbols may use a reserved-looking internal prefix such as `__vek_`.

Examples:

- `__vek_alloc`
- `__vek_array_detach`
- `__vek_panic`

Such names are for implementation and internal bindings, not normal user code.
