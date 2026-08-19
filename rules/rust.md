---
paths:
  - "**/*.rs"
---

# Rust

## Folder Structure

- Generally use one file per type. The file should be the snake_case string of the TypeName.
- A single type can span multiple files if it is large. In this case the folder should be the snake_case TypeName.
- Most projects should set `clippy::module_inception = "allow"` to allow good folder structure.
- A concern that spans every type (parsing, `Display`) gets its own top-level module with one file per type,
  mirroring the type folders. The type folders then hold only the type declarations & their conversions.
- Conversion impls live in `conversions.rs` files beside the type. Suffixes stack to split them by axis: `_ref` for
  borrowed variants, `_std` for standard library conversions, `_v4`/`_v6` for version variants. Standard library
  conversions never mix into the crate-internal files.

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
- Put `#[must_use]` on value types (above the derive) & keep the `clippy::must_use_candidate` warning satisfied.
- Re-export public dependencies whose types appear in the API. (e.g. `pub use address;`)
- Derive `Default` only where the default is meaningful; a zero port or an empty name is not.

## Owned & Reference Pairs

- Types that are not `Copy` come in owned & borrowed pairs: `T` & `TRef<'a>`. The reference type borrows its data, so
  it parses & converts without allocating.
- Owned to reference is `to_ref()`; reference to owned is named for the target type (`to_domain()`).
- `From<&'a T> for TRef<'a>` & `From<TRef<'a>> for T` both delegate to those methods.
- Impl `PartialEq` between the two in both directions, comparing through the reference type.
- Well-known instances are associated consts on the reference type; the owned constructor delegates to them.

## Conversions

- The inherent `to_*` method is the primitive; every `From` impl delegates to one, never the reverse.
- A `to_*` method lives on the source type, in the source type's folder. The cross-type `From` lives in the target
  type's folder, below the target's own impl block.
- Type files keep only the construction & deconstruction `From` pairs for tuples & raw data, grouped by direction.
- Narrowing conversions return `Option` & pair with `is_*` predicates in a `//! Matching` block.
- Skip a convenience conversion when composing two existing ones costs the same; add it only for a real allocation or
  ergonomic win.

## Functions

- Impl-block functions on `Copy` types take owned `self`, not `&self`. Trait signatures are exempt.
- Refer to the implemented type as `Self` in impl bodies: `Self::new(...)`, `Self::V4(...)`, `Self { ... }`.
- Unchecked constructors end in `_unchecked`, validate with `debug_assert!`, & are marked `unsafe` to protect
  struct invariants even when breaking them is not undefined behavior.
- Annotate local variable types explicitly where the type is nameable: `let scheme: Scheme = ...`.
- Validation predicates are `is_valid_x(&[u8])` with an `_str` twin taking `&str`. A variant that relaxes a rule names
  it as a suffix (`_ignore_case`), & the shared implementation takes the flag & is named `is_valid_x_op_<option>`.

## Parsing

- One hand-written parser per type: `parse_text(text: &[u8])`. `FromStr`, `TryFrom<&str>`, `TryFrom<String>` &
  `TryFrom<Vec<u8>>` all delegate to it.
- Byte input is a named method rather than `TryFrom<&[u8]>` when a byte slice could also read as raw data; the name
  says the bytes are text.
- Owned parsers normalize their input & document it: "... normalized to lowercase." Reference parsers require
  normalized input: "... must already be in lowercase. Use [`Owned`](crate::Owned) to parse mixed-case input."
- Consuming `TryFrom<String>` & `TryFrom<Vec<u8>>` impls reuse the input buffer instead of allocating & return the
  unmodified value in the error.

## Macros

- A declarative macro family lives in its own file named after the macros, with the `pub(crate) use` re-exports at the
  bottom of the file.
- Inside a macro, path everything absolutely: `::std::`, `::serde::`, `crate::$ty`.
- Take the doc lines as trailing `$doc:expr` arguments so the generated impls carry the same docs as the hand-written
  primitive they delegate to.

## Display

- A leaf `Display` writes through `f.pad(...)` so width, fill, & precision specs work. An owned type forwards to its
  reference type; a composite forwards to a component, or pads a formatted string only when a spec is set.
- `AsRef<str>` & `Borrow<str>` impls belong with `Display`, not with the type: a string view is display.

## Imports

- One contiguous import block with no blank lines between groups; let rustfmt order it.

## Docs

- One-line docs for constructors & accessors. No "Returns ..." lines that restate the signature.
- Only add `# Safety` / `# Panics` sections when they carry a real constraint, stated in one plain sentence.
- `# Safety` sections go only on `unsafe fn` docs; never add `// SAFETY:` comments on unsafe blocks.
- Cite the spec a type implements with a link, & enumerate the divergences from it explicitly.
- Point at the one full explanation instead of repeating it: ``(see [`Self::is_valid_name`])``.
- A private module with a contract to uphold states where that contract is written, in a `//!` comment on its `mod.rs`.

## Errors

- An error that consumes an owned value keeps it recoverable, like `std::string::FromUtf8Error`: wrap the value,
  expose `value()` & `into_value()` accessors, & impl `From<TheError>` for the plain error enum.
- Public error enums are `#[non_exhaustive]`.
- An error names the most specific part that can be blamed; document that rule on the enum.

## Tests

- Table-driven tests: a `test_cases: &[(...)]` slice looped with a context message in each assert.
  (`assert_eq!(result, *expected, "input={}", input)`)
- Reach for a table at roughly 3 or more homogeneous cases. Below that use sequential `let value` / `let result` /
  `let expected` / `assert_eq!` blocks.
- Each impl is tested in the file where it lives, in a `#[cfg(test)] mod tests` at the bottom, with the test functions
  in impl order.
- Name tests after the impl block section: `construction`, `deconstruction`, `properties`, `matching`, `display_spec`.
  Conversion tests are `<type>_to_x` & `<type>_from`, or `ref_to_x` & `ref_from` for the reference type.
- Construction tests assert against the private fields directly; the test module is inside the type's own module.
- Anything with a canonical text form gets a round trip test: each canonical string parses & displays back byte for
  byte.
- A doc comment above a `#[test]` says why the case exists when the test name cannot.
