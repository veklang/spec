# Diagnostics and Examples

## Diagnostics

The diagnostic format is:

- `error: {msg} ({code})`
- `warning: {msg} ({code})`

### Code Ranges

| Range | Area |
|-------|------|
| E00xx | Lexer |
| E10xx | Parser — structural |
| E11xx | Parser — declaration rules |
| E20xx | Checker — names and declarations |
| E21xx | Checker — type mismatches |
| E22xx | Checker — calls and arguments |
| E23xx | Checker — return types |
| E24xx | Checker — literal values |
| E25xx | Checker — mutability and assignment |
| E26xx | Checker — patterns and match |
| E27xx | Checker — top-level initializers and modules |
| E28xx | Checker — generics and traits |
| W26xx | Warnings — match coverage |
| W29xx | Warnings — reachability and unused |

### Lexer Errors (E00xx)

| Code | Message |
|------|---------|
| E0001 | Unexpected character `'{c}'`. |
| E0002 | Unterminated string literal. |
| E0003 | Unterminated block comment. |
| E0004 | Invalid escape sequence in string literal. |
| E0010 | Invalid hex literal. |
| E0011 | Invalid binary literal. |
| E0013 | Invalid exponent in numeric literal. |

### Parser Errors (E10xx–E11xx)

| Code | Message |
|------|---------|
| E1001 | Expected identifier. |
| E1002 | Expected `{kind}` token. |
| E1003 | Expected keyword `'{kw}'`. |
| E1004 | Expected operator `'{op}'`. |
| E1005 | Expected `'{punctuator}'`. |
| E1011 | Const declarations require an initializer. |
| E1020 | Expected `;`. |
| E1030 | Invalid match pattern. |
| E1040 | Unexpected keyword `'{kw}'`. |
| E1041 | Unexpected token `'{tok}'`. |
| E1050 | Expected type. |
| E1051 | Keyword not allowed as a type name. |
| E1099 | Parser made no progress. (internal guard) |

### Checker Errors — Names and Declarations (E20xx)

| Code | Message |
|------|---------|
| E2001 | Unknown identifier `'{name}'`. |
| E2002 | Duplicate field `'{name}'`. |
| E2003 | Unknown type `'{name}'` / Unknown struct `'{name}'`. |
| E2005 | Type argument count mismatch. |
| E2106 | Const declarations require an initializer. |

### Checker Errors — Type Mismatches (E21xx)

| Code | Message (examples) |
|------|---------|
| E2101 | Type mismatch (variable initializer, assignment, binary/unary operands, condition, struct field, index type). |
| E2102 | Cannot infer type of `'{name}'`. |
| E2103 | Missing struct field `'{name}'`. |
| E2104 | Unknown member `'{name}'` / Tuple index out of range. |
| E2105 | Invalid cast from `{from}` to `{to}`. |

### Checker Errors — Calls and Arguments (E22xx)

| Code | Message |
|------|---------|
| E2204 | Mut parameter requires a mutable identifier. |
| E2207 | Callee is not callable / Argument type mismatch / Missing arguments / Too many arguments. |

### Checker Errors — Return Types (E23xx)

| Code | Message |
|------|---------|
| E2302 | Return type mismatch. |

### Checker Errors — Literals (E24xx)

| Code | Message |
|------|---------|
| E2401 | Integer literal out of range. |
| E2402 | Compile-time integer overflow. |
| E2403 | Compile-time invalid shift. |
| E2404 | Compile-time division or modulo by zero. |

### Checker Errors — Mutability and Assignment (E25xx)

| Code | Message |
|------|---------|
| E2501 | Cannot assign to / mutate through const binding. |
| E2503 | Cannot assign to / mutate through readonly parameter. |
| E2504 | Invalid assignment target. |

### Checker Errors — Patterns and Match (E26xx)

| Code | Message |
|------|---------|
| E2601 | Enum payload arity mismatch / Tuple pattern arity mismatch. |
| E2602 | Pattern type mismatch. |
| E2603 | Unknown enum variant `'{name}'`. |
| E2604 | Enum pattern used with non-enum match target. |
| E2606 | Match expression requires a catch-all `_` arm. |

### Checker Errors — Initializers and Modules (E27xx)

| Code | Message |
|------|---------|
| E2700 | Cyclic top-level initializer: `{A} → {B} → {A}`. |
| E2702 | Module `'{mod}'` has no exported symbol `'{name}'`. |
| E2706 | Unsupported package import `'{spec}'`. |
| E2707 | Cannot resolve import `'{spec}'`. |
| E2708 | Failed to read module `'{path}'`. |

### Checker Errors — Generics and Traits (E28xx)

| Code | Message |
|------|---------|
| E2812 | Unknown trait in satisfies block / Unknown type parameter in where clause. |
| E2813 | `self` must be the first parameter / `self` is only valid inside methods. |
| E2814 | Missing trait method `'{name}'`. |
| E2815 | Trait method signature mismatch for `'{name}'`. |
| E2816 | Type does not satisfy trait bound `'{bound}'`. |
| E2817 | Duplicate satisfies block / Trait method conflicts with inherent method / Provided by multiple traits. |
| E2818 | Trait name used as parameter type; use a generic bound instead. |
| E2819 | Ambiguous trait method `'{name}'`. |
| E2820 | Cannot infer type argument for `'{T}'`. |

### Warnings — Match Coverage (W26xx)

| Code | Message |
|------|---------|
| W2601 | Match is not exhaustive; missing `{variants}` / Match may be non-exhaustive; add a catch-all `_` arm. |
| W2602 | Match arm is shadowed by an earlier arm. |

### Warnings — Reachability and Unused (W29xx)

| Code | Message |
|------|---------|
| W2900 | Declaration `'{name}'` is never used. |
| W2901 | Unused local `'{name}'`. |
| W2902 | Unused parameter `'{name}'`. |
| W2903 | `inline` is only a backend hint; the emitted C may still not be inlined by the downstream compiler. |
| W2904 | `inline` has no effect on `extern fn`; no local function body is emitted to inline. |

## Example: Minimal Program

```vek
import "std:io" as io;

fn main() {
  io.print("hello world\n");
  io.eprint("this is stderr\n");
}
```

## Example: Functions and Constants

```vek
import "std:io" as io;

const constant_value = 50;

fn add(x: i32, y: i32) {
  return x + y;
}

fn main() {
  let float_addition = 6.9 + 4.2;
  io.print("sum: " + add(6, 9).format() + "\n");
}
```

## Example: First-Class Functions

```vek
let add_one = fn(x: i32) -> i32 {
  return x + 1;
};

fn apply(x: i32, f: fn(i32) -> i32) -> i32 {
  return f(x);
}

let result = apply(41, add_one);
```

```vek
struct User {
  name: string;

  fn display_name(self) -> string {
    return self.name;
  }
}

let show_name: fn(User) -> string = User.display_name;
```

```vek
fn bad() -> fn(i32) -> i32 {
  let offset = 2;

  return fn(x: i32) -> i32 {
    return x + offset; // compile-time error: anonymous functions may not capture outer locals
  };
}
```

## Example: Match and Loops

```vek
import "std:io" as io;

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

fn main() {
  let label = match 50 {
    1 => "one",
    50 => "fifty",
    _ => "other",
  };

  io.print(label + "\n");

  let counter = Counter { cur: 0, end: 5 };

  for i in counter {
    io.print(i.format() + "\n");
  }

  for val in [6, 9, 4, 2, 0] {
    io.print(val.format() + "\n");
  }
}
```

## Example: Tuples, Maps, and Indexing

```vek
import Map from "std:collections";

let pair: (i32, string) = (1, "hi");
let first = pair.0;
let second = pair.1;

let counts = Map<string, i32>.new();
counts.set("apples", 3);
let maybe_apples = counts.get("apples");
let maybe_pears = counts.get("pears"); // null

let nums = [10, 20, 30];
let n = nums[1];

let s = "cat";
let c = s[0]; // "c"
```

## Example: Result-Based Errors

This example uses the global `Result<T, E>` from the core-library prelude.

```vek
import "std:io" as io;

fn might_fail(flag: bool) -> Result<i32, string> {
  if flag {
    return Ok(42);
  } else {
    return Err("bad flag");
  }
}

fn main() {
  match might_fail(false) {
    Ok(v)  => io.print("value: " + v.format() + "\n"),
    Err(e) => io.eprint("error: " + e + "\n"),
  }
}
```

## Example: Copy-on-Write Through a Normal Binding

```vek
fn main() {
  let a = [1, 2];
  let b = a;

  a.push(3);

  // a is now [1, 2, 3]
  // b remains [1, 2]
}
```

## Example: Mutation Through `mut` Still Uses CoW

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

## Example: Static Method by Convention

```vek
struct User {
  id: i32;
  name: string;
 
  fn new(id: i32, name: string) -> Self {
    return Self { id, name };
  }

  fn from_name(name: string) -> Self {
    return Self { id: 0, name };
  }
}
```

`new` is only a convention. There is no required constructor name and no `User(...)` constructor magic.

## Example: Generic Trait

This section shows an explicit manual implementation with `satisfies` using a global core trait.

```vek
struct UserId {
  value: i32;

  satisfies Equal<UserId> {
    fn equals(self, other: UserId) -> bool {
      return self.value == other.value;
    }
  }
}

fn same<T: Equal<T>>(a: T, b: T) -> bool {
  return a.equals(b);
}
```

Example of a constrained generic using multiple trait bounds:

```vek
import Map from "std:collections";

fn lookup<K, V>(map: Map<K, V>, key: K) -> V?
where K: Equal<K>, K: Hashable
{
  ...
}
```

## Example: Reachable Top-Level Initialization

```vek
import "std:io" as io;

const THING = "hello world".encode();

fn main() {
  io.print(THING);
}
```

In this program, `THING` is reachable, so its initializer executes. Unreachable top-level declarations may be dropped entirely.

## Example: Discarded Top-Level Side Effect Warning

```vek
import "std:io" as io;

const _announce = io.print("loaded\n");

fn main() {
  io.print("hello\n");
}
```

The initializer for `_announce` is not executed just because the module is imported or compiled. If `_announce` is never read, the compiler may discard it entirely and should warn.
