# Runtime Boundary

## Purpose

The Vek language depends on a small implementation runtime.

That runtime exists to support the parts of the language that cannot be expressed as pure source-level Vek alone, such as:

- panic termination
- the backing representation of `string`
- the backing representation of `Array<T>`

The runtime is not a public language module or user-facing API surface.

## Language-Facing Guarantees

The language currently assumes:

- `string` is a runtime-backed immutable value type
- `Array<T>` is a runtime-backed heap container with copy-on-write behavior
- `panic(...)` transfers control to an implementation-defined panic path and does not return normally

These are language guarantees. The exact runtime layout and helper symbols are implementation details.

## Scope Boundary

The language specification defines:

- the observable semantics of runtime-backed language features
- which built-in types require runtime support
- which operations panic or otherwise require implementation support

The runtime specification defines:

- concrete helper responsibilities
- runtime data representations
- toolchain and target assumptions
- native binding and interop details

## Non-Goals for the Language Surface

The runtime should not appear as a second standard library.

In particular, the language does not expose:

- manual memory management primitives such as `malloc` or `free`
- raw pointer arithmetic in the safe language surface
- implementation helper symbols as part of the user-facing API

Low-level implementation details belong in the runtime specification, not the public language chapters.
