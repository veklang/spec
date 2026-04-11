# Backend, Runtime, and Toolchain

## Compilation Pipeline

The current compilation model is:

```text
.vek source
  -> Vek compiler/frontend
  -> C code generation
  -> system C compiler
  -> native binary
```

## Frontend and Backend Roles

The compiler/frontend is responsible for:

- lexing
- parsing
- typechecking
- inference
- module resolution
- semantic analysis

The backend is responsible for:

- lowering checked Vek to C
- monomorphized code generation
- runtime support calls where needed
- producing buildable native output

## Runtime Strategy

Vek uses a small native C support layer.

Its role is to support:

- compiler-emitted operations
- core heap-backed types
- refcounting and copy-on-write
- panic and exit paths for the core global `panic(...)` surface
- basic I/O and process integration
- stdlib internals

## Memory Strategy

Current direction:

- primitives lower to native stack or register values where possible
- heap-backed values use reference counting
- shared mutable-ish heap structures use copy-on-write
- v1 does not use a tracing GC

Reference cycles are a known limitation for now and may leak until a later cycle-collection story exists.

Normal safe Vek does not expose manual memory management such as `malloc`, `free`, or raw pointer arithmetic.

A future `unsafe` feature is allowed as a later extension, but it should not shape the safe core language prematurely.

## Runtime-Backed Core Types

The minimal runtime-backed set includes:

- `string`
- `Array<T>`

Internal-only implementation-facing types may include:

- raw pointer or opaque handle types for stdlib internals
- file and process handle wrappers
- support types used to implement standard-library containers such as `Map<K, V>`

## Internal ABI Shape

The native support layer may expose internal helpers along lines such as:

- allocation and memory movement helpers
- retain and release operations
- uniqueness checks and detach helpers
- string creation, cloning, concatenation, comparison, and data access
- array creation, growth, cloning, and data access
- panic and exit operations
- basic file and stream I/O

Implementation symbols may use prefixes such as `__vek_*`.

These are internal interfaces used by the implementation and standard library.

## Toolchain and Linking

Vek is opinionated about its recommended toolchain story, but the underlying C toolchain remains configurable.

- the only required C-level dependency is the target platform's C standard library
- static linking is the recommended default where the platform meaningfully permits it
- extra runtime-library dependencies should be avoided by default
- on Linux, the recommended toolchain story is `musl`, ideally via `musl-gcc`, typically with static linking
- this is preferred for small binaries, simple distribution, and embeddability
- on Darwin, full static linking is not a normal target expectation
- on Windows and macOS, the exact system toolchain story follows the normal native platform toolchain

These are recommendations, not hard requirements. Toolchains and link modes may still be changed when needed.

## Target Support

Current supported target:

- `linux-x86_64`

Planned future target families:

- operating systems: `linux`, `windows`, `darwin`
- architectures: `x86`, `x86_64`, `arm`, `arm64`

Support should be described in terms of concrete target triples as the backend matures.

## Interop

C interop is a major strength of the design.

- Vek emits C
- native APIs can be wrapped cleanly
- C libraries are the primary interop target
- C++ interop can be handled through wrapper shims when needed

Networking can live in stdlib or libraries on top of native bindings.
