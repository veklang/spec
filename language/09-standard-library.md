# Standard Library

## Purpose

The standard library is the ordinary-library layer shipped with Vek beyond the global core prelude.

Its public modules live under the reserved `std:` package prefix.

Unlike the core library, stdlib symbols are not global. They must be imported explicitly.

Example:

```vek
import "std:io" as io;

fn main() {
  io.print("hello\n");
}
```

## Design Direction

The standard library should be:

- written in Vek as much as possible
- layered on top of the language runtime boundary and the implementation runtime beneath it
- exposed as ordinary modules and types
- broad enough to cover normal systems-programming and application needs
- conservative about hidden magic

The stdlib should not be treated as compiler-baked syntax. It is library surface.

## Required Areas

The standard library should cover at least the following areas.

### I/O

Modules such as `std:io` should provide:

- stdout and stderr printing
- input stream and output stream abstractions
- buffered I/O utilities
- formatting-oriented helpers built on core traits such as `Formattable`

### Filesystem and Paths

Modules such as `std:fs` and `std:path` should provide:

- file reading and writing
- directory creation, listing, and metadata queries
- path construction, normalization, joining, and splitting

### Process and Environment

Modules such as `std:process` and `std:env` should provide:

- process exit helpers
- subprocess spawning and status inspection
- environment-variable access
- argument inspection for program startup

### Collections

Modules such as `std:collections` should provide higher-level containers beyond the language/runtime-backed `Array<T>`.

That includes facilities such as:

- maps
- sets
- queues
- stacks
- collection-oriented helper utilities

These are library types, not built-in language primitives.

### Strings and Text Utilities

The language defines the `string` type itself, but the stdlib should provide text-oriented helpers around it.

That includes facilities such as:

- parsing helpers
- splitting and joining helpers
- trimming and normalization helpers
- encoding and decoding helpers
- formatting helpers that build on `Formattable`

### Iteration and Algorithms

Modules such as `std:iter` or collection-specific helpers should provide:

- iterator adapters
- traversal helpers
- searching
- filtering
- mapping
- folding or reduction
- sorting utilities where applicable

### Math and Numeric Utilities

Modules such as `std:math` should provide:

- common numeric helpers
- transcendental functions
- rounding helpers
- constants
- matrix, vector, or other structured math helpers as the library surface grows

### Time and Randomness

Modules such as `std:time` and `std:rand` should provide:

- timestamps and durations
- clocks and sleep helpers
- deterministic pseudorandom generation
- APIs for accessing secure randomness where the platform permits it

### Networking

Networking is standard-library-appropriate.

Modules such as `std:net` should cover the basic transport surface:

- sockets
- addresses
- listeners
- client connections

Higher-level protocol libraries may still live outside the stdlib when appropriate.

## Layering Rules

The public stdlib should sit above the core library and the implementation runtime.

Conceptually:

```text
user packages
  -> std:*
  -> core prelude + core:*
  -> minimal C runtime/bindings
```

Private implementation modules may exist inside the stdlib package, but they are not part of the stable public API.

## Stability Boundary

The `std:*` public modules are the user-facing standard library surface.

Implementation-oriented helper modules, private wrappers over native bindings, and low-level support code may exist behind that surface, but they should not be specified as part of the public API unless intentionally exported and documented.
