# Functions and Control Flow

## Function Declarations

- parameter types are required
- parameter type inference is not supported
- return types are inferred by default
- explicit return type annotations are allowed

```vek
fn add(x: i32, y: i32) -> i32 {
  return x + y;
}
```

## Parameters and Arguments

Vek supports positional arguments only in v1.

### Call Rules

- call arguments are matched to parameters by position
- the number of arguments must match the number of parameters exactly
- keyword arguments are not supported
- variadic parameters are not supported
- argument unpacking with `*` is not supported

Examples:

```vek
fn connect(host: string, port: i32) { ... }

connect("db.local", 5433);
```

### Parameter Rules

- parameters are positional only
- `mut` appears before the parameter name: `mut x: i32`
- `x: mut i32` is not valid syntax
- a trait name may appear in parameter type position as shorthand for an implicit constrained type parameter
- default values are not supported
- variadic parameters are not supported

Examples:

```vek
fn print(message: string, stream: Stream) { ... }
fn greet(name: string, suffix: string) { ... }
fn push_one(mut xs: i32[]) -> void { ... }
```

### Trait Parameter Sugar

In a function or method parameter declaration, a trait name may be used directly as the parameter type.

Example:

```vek
fn read_line(mut reader: Reader) -> Result<string, Error> {
  ...
}
```

This is shorthand for an implicit constrained type parameter with static dispatch, not a trait object.

Conceptually, the example above desugars to:

```vek
fn read_line<__Reader0>(mut reader: __Reader0) -> Result<string, Error>
where __Reader0: Reader
{
  ...
}
```

Rules:

- the sugar introduces a fresh hidden type parameter per parameter
- inside the function body, the parameter may be used through the trait surface guaranteed by that constraint
- this remains statically dispatched and monomorphized
- this sugar does not mean trait objects or dynamic dispatch
- if multiple parameters or a return type must refer to the same concrete type, write explicit generics instead
- this sugar is valid only in parameter declarations, not in local bindings, struct fields, enum payloads, return types, or function type syntax

## Function Value Status

Functions are first-class values in v1.

- functions may be stored in variables
- functions may be passed as arguments
- functions may be returned from other functions
- anonymous functions are supported
- function type syntax is supported

Anonymous functions are non-capturing in v1.

- they may refer to their own parameters
- they may refer to bindings declared inside their own body
- they may refer to global or module-level symbols
- they may not capture bindings from an enclosing local function scope

Using an outer local binding inside an anonymous function is a compile-time error.

Canonical function type syntax:

```vek
fn run_twice(func: fn() -> void) {
  func();
  func();
}
```

Function types consist of:

- an optional generic parameter list
- the ordered parameter list
- whether each parameter is readonly or `mut`
- the return type

Parameter names are not part of function type identity.

Function type parameter syntax mirrors ordinary function parameter syntax, except names are omitted.

- a readonly parameter is written as just its type, such as `i32`
- a mutable parameter is written as `mut T`, such as `mut i32[]`
- generic constraints may be written inline in the generic parameter list, such as `fn<T: Equal<T>>(T, T) -> bool`
- a `where` clause may also be used on function types when needed

Examples:

```vek
fn(i32, i32) -> i32
fn(mut i32[]) -> void
fn<T>(T) -> T
fn<T: Equal<T>>(T, T) -> bool
```

### Function Value Compatibility

Function-value compatibility is exact-match only.

- parameter count must match exactly
- parameter order must match exactly
- parameter types must match exactly
- parameter `mut` status must match exactly
- return type must match exactly
- if present, generic parameter count, order, and constraints must match exactly
- parameter names do not matter
- there is no variance-based compatibility or other implicit callable coercion

Named functions and non-capturing anonymous functions may be used anywhere a matching function type is expected.

Generic functions may be used as values when the expected function type is also generic and matches exactly.

Methods may be used as function values through a type-qualified method reference.

- referencing an instance method as a value uses the receiver type as the first parameter in the function type
- referencing a static method as a value uses its declared parameter list unchanged
- member access on a value remains ordinary method-call/member syntax; method-value formation is via `TypeName.method_name`

Examples:

```vek
struct User {
  name: string;

  fn display_name(self) -> string {
    return self.name;
  }
}

fn add_one(x: i32) -> i32 {
  return x + 1;
}

let f: fn(i32) -> i32 = add_one;
let g: fn(User) -> string = User.display_name;
```

## Generic Constraints

Generic functions may constrain type parameters with traits.

For simple cases, use inline `:` constraints:

```vek
fn same<T: Equal<T>>(a: T, b: T) -> bool {
  return a.equals(b);
}
```

For longer or combined constraints, use a `where` clause:

```vek
import Map from "std:collections";

fn lookup<K, V>(map: Map<K, V>, key: K) -> V?
where K: Equal<K>, K: Hashable
{
  ...
}
```

Rules:

- `T: TraitName` is the short form for a single clear constraint
- `where` is the long form for combined or more complex constraints
- multiple constraints are written as repeated entries, such as `where K: Equal<K>, K: Hashable`
- generic trait arguments are written explicitly, such as `Equal<T>`
- constraint matching is explicit and unambiguous
- parameter-position trait names are shorthand for hidden constrained type parameters, not trait-object types
- associated types and specialization are out of scope for v1

## Tuple Returns

Functions may return tuples directly.

```vek
fn pair() -> (i32, i32) {
  return (1, 2);
}
```

## Inline Functions

`inline fn` is available as a compile-time hint for small functions.

```vek
inline fn add(x: i32, y: i32) -> i32 {
  return x + y;
}
```

Inlining behavior is implementation-defined and may be handled entirely by the compiler and downstream C optimizer.

## Error Handling

Vek uses explicit `Result<T, E>`-style errors. There are no exceptions.

```vek
fn read_config() -> Result<string, string> {
  return Ok("config");
}
```

`Result` and `panic` come from the global core-library prelude described in chapter 8. They do not need to be imported in user code.

## Conditionals

```vek
if 1 == 1 {
  // ...
} else {
  // ...
}
```

`else if` is supported.

Condition expressions in `if` and `while` must be typed as `bool`.

### Logical Operators

`&&` and `||` require `bool` operands and always produce `bool`.

- `a && b` evaluates to `true` only when both operands are `true`
- `a || b` evaluates to `true` when either operand is `true`
- both operators short-circuit
- unlike JavaScript, these operators do not return one of their operand values

### Narrowing

Control-flow narrowing is supported for explicit tests that the compiler can prove.

This includes cases such as:

- `x != null`
- `x == null`
- enum-pattern matches

Example:

```vek
let maybe_num: i32? = 1;

if maybe_num != null {
  // here maybe_num is narrowed to i32
}
```

## Match

`match` is both a control-flow statement form and an expression form.

```vek
match 50 {
  1 => {},
  50 => {},
  _ => {},
}
```

When used as an expression, the arm result values must unify to a single type.

When used as an expression, a `match` must include a catch-all `_` arm. Omitting the catch-all arm is a compile-time error, even if the compiler could otherwise prove the listed arms are exhaustive.

```vek
let label = match 50 {
  1 => "one",
  50 => "fifty",
  _ => "other",
};
```

Supported pattern forms:

- literals
- wildcard `_`
- identifier binding
- enum payload patterns such as `Ok(v)` and `Err(e)`
- nested enum patterns
- tuple patterns

### Exhaustiveness

Statement `match` exhaustiveness is not mandatory, but the compiler warns in important cases.

- expression `match` must include a catch-all `_` arm and is otherwise a compile-time error
- statement `match` may omit a catch-all arm
- for finite compile-time-known domains, the compiler warns on non-exhaustive statement matches without a catch-all
- for unknown or unbounded domains, the compiler warns when a statement match has no catch-all
- shadowed and unreachable arms may also trigger warnings

## Loops

### `while`

```vek
while true {}
```

`for item in expr { ... }` iterates over values produced by `expr`.

- arrays satisfy the global `Iterable<T>` trait from the core library
- user-defined iterable types may participate by satisfying `Iterable<T>`
- iteration proceeds by repeatedly calling `next(mut self) -> T?` until `null` is returned
- the loop creates a hidden mutable temporary from `expr` and repeatedly calls `next` on that temporary
- this temporary owns the iteration state for the duration of the loop

### `for` Iterable Loop

```vek
for val in [6, 9, 4, 2, 0] {}
```

### Example Custom Iterable

```vek
struct Counter {
  cur: i32;
  end: i32;

  satisfies Iterable<i32> {
    fn next(mut self) -> i32? {
      if self.cur >= self.end {
        return null;
      }

      let value = self.cur;
      self.cur = self.cur + 1;
      return value;
    }
  }
}

let counter = Counter { cur: 0, end: 9 };

for i in counter {}
```

`break` and `continue` are supported.
