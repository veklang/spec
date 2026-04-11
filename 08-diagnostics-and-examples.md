# Diagnostics and Examples

## Diagnostics

The diagnostic format is:

- `error: {msg} ({code})`
- `warning: {msg} ({code})`

Suggested code families:

- `E0xxx` for lexing
- `E1xxx` for parsing
- `E2xxx` for semantic and static analysis
- `W2xxx` for warnings

Additional warnings should cover suspicious reachability cases.

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

This example defines a local `Result<T, E>` enum purely to demonstrate the pattern. The real `Result<T, E>` used by the language is provided globally by the internal core library.

```vek
import "std:io" as io;

enum Result<T, E> {
  Ok(T);
  Err(E);
}

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

This section shows two related forms:

- a generic trait definition
- an explicit manual implementation with `satisfies`

```vek
trait Equal<T> {
  fn equals(self, other: T) -> bool;
}
```

Manual implementation:

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
