# Minimal C Runtime and Bindings

## Purpose

Vek uses a tiny native C support layer we call the "runtime".

That layer is intentionally minimal. It exists only to support:

- the global `panic(...)` surface from the core library
- the runtime representation of `string`
- the runtime representation of `Array<T>`
- the small set of native bindings needed by `std:*`

Everything else should live in normal Vek code in `core:*`, `std:*`, or third-party packages.

## Runtime Boundary

The runtime boundary should stay narrow.

The C layer should not become a second standard library. It should expose only the operations that the generated program and the Vek-written libraries cannot express for themselves.

In practice, that means:

- reference counting and copy-on-write support for heap-backed values
- native backing storage for `string`
- native backing storage for `Array<T>`
- process termination and panic emission
- basic C-facing handles and functions that `std:*` wraps into Vek APIs

The runtime boundary is implementation-facing. It is not part of the stable user-facing Vek API.

## Memory Strategy

Current direction:

- primitives lower to native stack or register values where possible
- heap-backed values use reference counting
- shared heap-backed values use copy-on-write
- v1 does not use a tracing GC

Reference cycles are a known limitation for now and may leak until a later cycle-collection story exists.

Normal safe Vek does not expose manual memory management such as `malloc`, `free`, or raw pointer arithmetic.

A future `unsafe` feature is allowed as a later extension, but it should not shape the safe core language prematurely.

## Runtime-Backed Types

The minimal runtime-backed set is:

- `string`
- `Array<T>`

No other container is language- or runtime-required in v1.

Types such as `Map<K, V>`, file handles, process handles, sockets, and similar library-facing abstractions belong in `std:*` or other Vek packages, even when they wrap native resources internally.

## Required Runtime Operations

The runtime needs enough internal support to implement the language and the Vek-written libraries above it.

That support includes operations along these lines:

- retain and release for heap-backed values
- uniqueness checks and detach helpers for copy-on-write writes
- string creation, cloning, concatenation, comparison, length, and indexed access support
- array creation, cloning, growth, length, and indexed access support
- panic emission and abnormal termination
- a small binding layer for I/O, filesystem, process, environment, time, and other platform services that `std:*` wraps

Implementation symbols may use prefixes such as `__vek_*`.

Those symbol names are implementation details, not public APIs.

## Binding Model for `std:*`

The standard library should feel like ordinary Vek code, not direct libc usage.

Conceptually the stack is:

```text
public stdlib in Vek
  -> private Vek wrappers over native bindings
  -> tiny C runtime/binding layer
  -> libc / platform C API
```

The native binding layer may expose opaque handles or low-level helpers for stdlib implementation needs, but those should stay private to the stdlib package surface.

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

## The `extern` Keyword

The `extern` modifier marks a function as having no Vek body — its implementation is provided by the native runtime or a linked C layer.

```vek
extern fn panic(message: string) -> void;
```

An `extern fn` declaration has a signature but no body block. The compiler treats it as an opaque symbol resolved at link time.

**`extern` is not user-accessible in v1.**

There is no user-facing C foreign-function interface yet. `extern` is reserved for compiler-internal use to declare the small set of runtime-backed symbols (such as `panic`) that the core library requires. User code cannot declare `extern fn` items.

This restriction may be lifted in a future version when a proper FFI story is designed.

## Interop

C interop is a major strength of the design.

- Vek emits C
- native APIs can be wrapped cleanly
- C libraries are the primary interop target
- C++ interop can be handled through wrapper shims when needed

High-level facilities should still prefer Vek libraries on top of those bindings rather than exposing raw native APIs directly as the default user experience.
