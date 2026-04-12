# Data Types and Abstractions

## Types

`struct` is the main user-defined data-and-method declaration.

```vek
struct Stuff {
  num: i32;
  str: string;
}

let stuff = Stuff { num: 69, str: "lol" };
```

Field access uses dot syntax.

```vek
let user = Stuff { num: 1, str: "hi" };
let name = user.str;
```

Field shorthand is allowed in struct literals.

```vek
let num = 69;
let str = "lol";
let stuff = Stuff { num, str };
```

### Struct Literal Construction Rules

Struct literals construct a value of the named struct by explicitly initializing fields.

Rules:

- every required field must be provided exactly once
- field order in a struct literal is not semantically significant
- duplicate fields are a compile-time error
- unknown fields are a compile-time error
- field shorthand `Stuff { num, str }` is equivalent to `Stuff { num: num, str: str }`
- omitted fields are a compile-time error unless a later language feature explicitly defines field defaults

Example:

```vek
struct User {
  id: i32;
  name: string;
}

let ok = User { id: 1, name: "a" };
let bad = User { id: 1 }; // compile-time error: missing field `name`
```

Layout intent is contiguous and predictable, though exact low-level guarantees remain `TBD`.

## Struct Members

A `struct` body may contain:

- fields
- inherent methods
- `satisfies TraitName { ... }` blocks

Example:

```vek
struct User {
  id: i32;
  name: string;

  fn display_name(self) -> string {
    return self.name;
  }
}
```

### Method Rules

- dispatch is static
- instance methods use explicit `self`
- `mut self` is used for mutating methods
- methods without `self` are static methods
- `Self` names the enclosing concrete type within a `struct`, `enum`, or `trait` context

Example:

```vek
struct User {
  id: i32;
  name: string;

  fn new(id: i32, name: string) -> Self {
    return Self { id, name };
  }

  fn display_name(self) -> string {
    return self.name;
  }

  fn rename(mut self, name: string) -> void {
    self.name = name;
  }
}
```

### Constructors

Constructors are ordinary static methods by convention.

- structs are not callable
- `User(...)`-style constructor sugar does not exist
- `new` is a common convention, not a required language feature
- names such as `from`, `parse`, or `default` are equally valid
- methods returning the enclosing type should prefer `Self`

Example:

```vek
struct User {
  id: i32;
  name: string;

  fn from_name(name: string) -> Self {
    return Self { id: 0, name };
  }
}
```

## Traits

`trait` defines a compile-time behavior contract.

```vek
trait Printable {
  fn print(self) -> void;
}
```

Traits may have type parameters.

```vek
trait Equal<T> {
  fn equals(self, other: T) -> bool;
}
```

Manual trait conformance is written inside the struct or enum body with `satisfies`.

```vek
struct User {
  id: i32;
  name: string;

  satisfies Printable {
    fn print(self) -> void {
      io.print(self.name + "\n");
    }
  }
}
```

Generic traits use explicit trait arguments in `satisfies`.

```vek
struct UserId {
  value: i32;

  satisfies Equal<UserId> {
    fn equals(self, other: UserId) -> bool {
      return self.value == other.value;
    }
  }
}
```

Rules:

- trait dispatch is static
- traits may have first-order type parameters
- trait arguments are part of trait identity and must match exactly
- trait objects and dynamic dispatch are not supported
- associated types are not supported
- default generic parameters on traits are not supported
- specialization is not supported
- an inherent method and a trait method may not share the same name on the same struct or enum
- two satisfied traits may not expose the same method name on the same struct or enum
- a struct or enum may not contain more than one `satisfies` block for the same trait

The language-level trait mechanics are defined here. The standard global core traits themselves are specified in chapter 8.

## Enums

Enums are tagged variants with optional payloads, similar to Rust's enums.

An `enum` body may contain:

- variants
- inherent methods
- `satisfies TraitName { ... }` blocks

Enums support the same method and trait-conformance surface as `struct`.

Enum variants are terminated with `;`.

Example:

```vek
enum Message {
  Info(string);
  Warning(string);
  Error(string);
}
```

Trait conformance on enums uses the same `satisfies` form:

```vek
enum Message {
  Info(string);
  Warning(string);
  Error(string);

  satisfies Printable {
    fn print(self) -> void {
      match self {
        Info(_) => io.print("info\n"),
        Warning(_) => io.print("warning\n"),
        Error(_) => io.print("error\n"),
      }
    }
  }
}
```

Matching over payload enums:

```vek
match message {
  Info(text) => io.print(text + "\n"),
  Warning(text) => io.eprint("warning: " + text + "\n"),
  Error(text) => io.eprint("error: " + text + "\n"),
}
```

Payload arity matches the variant definition.
