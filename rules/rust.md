---
paths:
  - "**/*.rs"
---

# Rust

## Folder Structure

- Generally use one file per type. The file should be the snake_case string of the TypeName.
- A single type can span multiple files if it is large. In this case the folder should be the snake_case TypeName.
- Most projects should use `#![allow(clippy::module_inception)]` to allow good folder structure.

## File Layout

- `mod.rs` & `lib.rs` files hold only `mod` declarations & re-exports, never code. Mods with an associated re-export
  go in a separate block before the mods without one.
- Functions & constants belong in impl blocks, even when they don't use `self`.
- One impl block per concern (`//! Construction`, `//! Validation`, `//! Properties`, `//! Display`, ...), with the
  section comment inside the block at the top:
  ```rust
  impl Endpoint {
      //! Construction

      pub const fn new(domain: Domain, port: u16) -> Self { ... }
  }
  ```
- Order declarations in a type's file: imports, type declaration, constants, static construction utils (well-known
  instances like `LOCALHOST` & `example()`), construction, `Default` impl, `From` impls, properties, core
  functionality, `Debug`, `Display`.

## Types

- Value types derive `Copy, Clone, Ord, PartialOrd, Eq, PartialEq, Hash, Debug` (that order) when possible.
- Types whose `Display` round-trips through parsing impl `Debug` as `Display` & drop the `Debug` derive.
- Iterator structs derive only `Copy, Clone, Debug`.
- Put `#[must_use]` on value types (above the derive) & keep `#![warn(clippy::must_use_candidate)]` satisfied.
- Re-export public dependencies whose types appear in the API. (e.g. `pub use address;`)

## Functions

- Impl-block functions on `Copy` types take owned `self`, not `&self`. Trait signatures are exempt.
- Refer to the implemented type as `Self` in impl bodies: `Self::new(...)`, `Self::V4(...)`, `Self { ... }`.
- Unchecked constructors end in `_unchecked`, validate with `debug_assert!`, & are marked `unsafe` to protect
  struct invariants even when breaking them is not undefined behavior.
- Annotate local variable types explicitly where the type is nameable: `let scheme: Scheme = ...`.

## Imports

- One contiguous import block with no blank lines between groups; let rustfmt order it.

## Docs

- One-line docs for constructors & accessors. No "Returns ..." lines that restate the signature.
- Only add `# Safety` / `# Panics` sections when they carry a real constraint, stated in one plain sentence.
- `# Safety` sections go only on `unsafe fn` docs; never add `// SAFETY:` comments on unsafe blocks.

## Errors

- An error that consumes an owned value keeps it recoverable, like `std::string::FromUtf8Error`: wrap the value,
  expose `value()` & `into_value()` accessors, & impl `From<TheError>` for the plain error enum.

## Tests

- Table-driven tests: a `test_cases: &[(...)]` slice looped with a context message in each assert.
  (`assert_eq!(result, *expected, "input={}", input)`)
