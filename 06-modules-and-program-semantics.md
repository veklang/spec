# Modules and Program Semantics

Modules are simply `.vek` source files.

## Exports

- `pub` marks a symbol as exported from a module
- everything is private by default
- there is no `private` keyword

```vek
pub fn add(a: i32, b: i32) -> i32 {
  return a + b;
}

pub const pi = 3.14;
```

## Namespace Imports

Namespace imports use `import "path" as name;`.

```vek
import "./math" as math;
math.add(1, 2);
```

Rules:

- `import "path" as name;` binds all exported `pub` symbols from that module under `name`
- the namespace shape is fixed at compile time

## Import Forms

Vek uses a small dedicated import syntax.

```vek
import "std:io" as io;
import add, pi from "./math";
import "./math" as math;
```

## Import Resolution

Resolution rules:

- `name:path/...` resolves relative to that package
- paths without a package prefix resolve relative to the current file
- relative imports resolve in this order:
  1. `path`
  2. `path.vek`
  3. `path/index`
  4. `path/index.vek`
- omitting `.vek` is preferred
- importing a folder resolves through its `index.vek` when present
- package paths such as `std:collections` likewise resolve through `index.vek` when they name a folder/module root

Named-import rules:

- `import a, b, c from "path";` imports the exported symbols `a`, `b`, and `c` directly
- braces are not used for named imports
- named imports and namespace imports are distinct forms

Examples:

- `std:io`
- `std:math/matrix`
- `some_package:some/path`
- `my_project:utils/thing`

Package discovery and registry mechanics remain `TBD`.

## Top-Level Initializers

Top-level declarations may have conservative initializers.

```vek
const HTTP_RESPONSE = "...".encode();
```

Evaluation is based on semantic reachability and actual value use, not on textual import alone.

- a top-level initializer runs only if its declaration is reachable in the final program and its value is actually needed at runtime
- unreachable declarations may be omitted entirely
- reachable-but-never-read declarations may also be omitted entirely
- importing a module does not mean "run all top-level code in that module"
- top-level initializers are lazy: a declaration's initializer executes when that declaration's value is first needed
- each top-level initializer executes at most once
- if a top-level declaration is never read, its initializer does not execute
- top-level initializers should be limited to conservative value construction rather than effectful startup work
- implementations may reject or warn on obviously effectful top-level initializers

Execution order is therefore demand-driven rather than import-driven.

- if `A` is first needed before `B`, then `A`'s initializer executes before `B`'s
- there is no guarantee that textual declaration order across a module, or import order across modules, will eagerly execute top-level initializers
- programs must not rely on top-level initializers for import-time side effects

Each top-level declaration with an initializer conceptually has one of three states:

- uninitialized
- initializing
- initialized

If evaluation of one top-level initializer attempts to read another declaration that is already in the `initializing` state, the program traps with a cyclic top-level initialization panic.

Implementations may diagnose obvious cycles earlier at compile time, but the semantic rule is that re-entrant top-level initialization is invalid.

If a side effect must happen, it should be placed in explicit startup code rather than in a top-level declaration that may be treeshaken, rejected, warned on, or never evaluated.

Compilers should warn when a discarded top-level initializer appears to exist only for side effects.

## Reachability and Treeshaking

Vek performs semantic reachability analysis after typechecking.

This applies to:

- user code
- library code
- standard library code
- unused generic instantiations
- unreachable private helpers

Typical roots include:

- `main` in executable builds
- compiler-defined special roots when needed

Unused declarations may be dropped before C emission. Linker-level dead-code elimination may further reduce the final binary.

## Standard Library Layering

The public standard library should feel like ordinary Vek code, not direct libc usage.

Conceptually the stack is:

```text
public stdlib in Vek
  -> internal stdlib modules / bindings
  -> tiny C support ABI
  -> libc / platform C API
```

Internal modules may use implementation-oriented names such as:

- `std:__core.mem`
- `std:__core.str`
- `std:__core.array`
- `std:__core.io`
- `std:__core.proc`

These internal modules are not part of the stable public user-facing API.
