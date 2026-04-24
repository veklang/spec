# Types and Bindings

## Primitive Types

- signed integers: `i8`, `i16`, `i32`, `i64`
- unsigned integers: `u8`, `u16`, `u32`, `u64`
- floating point: `f32`, `f64`
- other primitives: `bool`, `string`
- special types: `void`, `null`

Future versions may add additional primitive numeric types such as `f16`, `i128`, `u128`, and `f128`.

## Composite Surface Syntax

The language establishes the following literal forms:

- array literal: `[1, 2, 3]`
- tuple literal: `(6, 9)`

The exact public standard-library surface for arrays and maps is still evolving.

## Array Type Syntax

Vek supports postfix array type syntax as sugar over `Array<T>`.

Examples:

- `i32[]` is equivalent to `Array<i32>`
- `string[]` is equivalent to `Array<string>`
- `i32[][]` is equivalent to `Array<Array<i32>>`

The postfix `[]` form may be repeated for multidimensional array types.

## Tuple Types

Tuple types use comma-separated element types in parentheses.

Examples:

- `(i32, i32)`
- `(string, bool, i32)`

Rules:

- tuple arity is part of the type
- tuple element order is part of the type
- `(T)` is just a parenthesized type, not a 1-tuple
- 1-tuples use a trailing comma: `(T,)`
- `()` is the empty tuple type

Tuple literals follow the same shape rules.

- `(1, 2)` is a 2-tuple
- `(1)` is a parenthesized expression
- `(1,)` is a 1-tuple

Tuple element access is by zero-based integer index using `.0`, `.1`, and so on.

Example:

```vek
let pair: (i32, string) = (1, "hi");
let first = pair.0;
let second = pair.1;
```

Tuples are fixed-size values. Their length is known statically.

## Map Types

Map types use `Map<K, V>`.

`Map<K, V>` is a standard-library type.

Example:

```vek
import Map from "std:collections";

let counts = Map<string, i32>.new();
```

Rules:

- maps do not have dedicated literal syntax in v1
- maps are constructed and manipulated through standard-library APIs
- non-string map keys require `K: Equal<K>` and `K: Hashable`
- `Map<K, V>` is an aliasable heap-backed container and follows the normal readonly and copy-on-write rules

The language does not guarantee a particular iteration order for maps.

## Indexing

Indexing uses `value[index]`.

Supported built-in indexing cases are:

- arrays indexed by `i32`
- strings indexed by `i32`

### Array Indexing

- `arr[i]` returns a value of type `T` for `arr: T[]`
- array indexing is zero-based
- negative indexes are out of bounds
- out-of-bounds array indexing is a runtime panic
- `arr[i] = value;` is allowed only through a mutable receiver

### String Indexing

- `s[i]` returns a `string` containing exactly one Unicode scalar value
- string indexing is zero-based by Unicode scalar value, not UTF-8 byte offset
- negative indexes are out of bounds
- out-of-bounds string indexing is a runtime panic
- strings are immutable, so indexed assignment is not allowed

## Composite Literal Rules

Array and tuple literals are ordinary expression forms.

Rules:

- array literals require elements that can be unified to a single element type
- tuple literals preserve per-position types
- empty array literals may require contextual type information when the element type cannot be inferred

Examples:

```vek
let nums = [1, 2, 3];
let pair = (1, "hi");
```

## Type Aliases

```vek
type MaybeCount = i32?;
```

Type aliases may reference nullable forms and other named types.

## Nullability

Vek uses a dedicated postfix nullable type operator: `?`.

- `T?` means either a value of type `T` or `null`
- `?` binds to the immediately preceding type expression
- `null` may be assigned to any nullable type
- nullable values must be checked before use
- control-flow narrowing applies for explicit checks the compiler can prove, such as `x != null`

Examples:

- `i32?`
- `string?`
- `(i32, string)?`
- `User?`

General type unions are not supported in v1. Use `enum` for multi-branch sum types.

Example:

```vek
import "std:io" as io;

let maybe_num: i32? = null;

if maybe_num != null {
  io.print("has value\n");
} else {
  io.print("was null\n");
}
```

## Generics

Generics are monomorphized.

- the compiler emits specialized concrete instances for each used type argument set
- type arguments may be inferred where unambiguous
- explicit type arguments remain allowed

## Bindings

### `let`

`let` declares a mutable binding.

```vek
let x = 10;
x = 20;
```

### `const`

`const` declares an immutable binding.

- reassignment is a compile-time error
- mutating through a `const` binding is disallowed
- `const` is deeply readonly through that binding

```vek
const x = 10;
x = 20; // compile-time error
```

Deep readonly means the entire reachable value is readonly when accessed through that binding, not just the top-level binding slot.

Example of deep readonly:

```vek
const user = User {
  name: "bob",
  tags: ["lang", "compiler"],
};

user.name = "other";    // error
user.tags.push("spec"); // error
```

This differs from shallow readonly, where only rebinding the top-level name would be forbidden while nested values could still be mutated. Vek `const` is not shallow readonly.

### Compile-Time Constants

Vek does not expose a separate language form for compile-time constants. The compiler may constant-fold expressions whenever it can prove the value at compile time.

## Value Passing and Mutability

### Default Argument Passing

Function arguments are passed as readonly values by default.

- values are readonly inside the callee unless the parameter is declared `mut`
- strings are immutable

```vek
fn f(a: i32[]) {
  a.push(1); // compile-time error
}
```

### `mut` Parameters

A `mut` parameter allows the callee to mutate the caller's value.

```vek
fn push_one(mut a: i32[]) {
  a.push(1);
}
```

Passing a value as `mut` does not by itself force a copy or detachment. If the callee only reads the value, no CoW action occurs.

### Copy-on-Write

Aliasable heap-backed values use copy-on-write semantics.

- assigning an aliasable value to another binding aliases the underlying storage
- any actual mutation of shared storage detaches before the write is applied
- passing a value to a `mut` parameter does not detach it by itself
- Vek never writes through shared storage

Example:

```vek
fn main() {
  let a = [1, 2];
  let b = a;

  a.push(3);

  // a is now [1, 2, 3]
  // b remains [1, 2]
}
```

This same rule applies inside `mut` callees:

```vek
fn push_one(mut xs: i32[]) {
  xs.push(1);
}

fn main() {
  let a = [1, 2];
  let b = a;

  push_one(a);

  // a is now [1, 2, 1]
  // b remains [1, 2]
}
```

The detach happens at the point of mutation, not at the point where the value is passed as `mut`.

### Mutating Methods

Mutating methods require a mutable receiver:

- a `let` binding

Calling a mutating method on a `const`/readonly value is a compile-time error.
