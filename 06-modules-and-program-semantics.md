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

## Packages

A package is a directory tree rooted at a folder that contains `package.toml`.

That manifest defines the package identity used by `name:path/...` imports.

Minimal example:

```toml
name = "my_project"
version = "0.1.0"
```

Rules:

- a directory is a package iff it contains `package.toml`
- the package root is the directory that contains `package.toml`
- `package.toml` must define `name`
- `name` is the package-import prefix used in source code, such as `my_project:utils/math`
- the package name is semantic; it is not inferred from the folder name
- package names must be unique within the active dependency graph
- paths without a package prefix continue to resolve relative to the importing file
- package-prefixed paths resolve relative to the target package root
- `core` is a reserved package name for the core library package
- `std` is a reserved package name for the standard library package

`package.toml` is intended to stay small in v1.

The currently defined top-level keys are:

- `name: string`
- `version: string`

Implementations may later add dependency and build metadata, but the existence of a package and its import prefix are defined by `package.toml` plus `name`.

## Package Resolution

Package resolution happens in two layers:

1. discover a package by its manifest and `name`
2. resolve the module path inside that package using the normal file rules

For a package import such as `collections:hash/map`:

- the compiler first locates the package whose `package.toml` declares `name = "collections"`
- it then resolves `hash/map` relative to that package root
- module-file lookup inside the package uses the same ordered rules as local imports:
  1. `path`
  2. `path.vek`
  3. `path/index`
  4. `path/index.vek`

The exact dependency-graph and registry/distribution mechanics are intentionally left open for now.

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

The runtime/core/std layering is defined in chapters 7 through 9.
