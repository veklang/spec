# Vek Specification

## Structure

- [Language Spec](./language/README.md)
- [Compiler / Internal Spec](./compiler/README.md)
- [Runtime Spec](./runtime/README.md)

## Scope

The `language/` documents define:

- language surface syntax
- static semantics and typing behavior
- module and reachability semantics
- compilation model and target strategy
- core-library surface
- standard-library surface
- naming and style conventions

They do not freeze every implementation detail. Some areas remain explicitly `TBD`.

The `compiler/` and `runtime/` folders are for implementation-facing specifications that should stay separate from the public language surface.
