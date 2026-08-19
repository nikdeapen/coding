---
paths:
  - "**/Cargo.toml"
---

# Cargo

- Lints go in the `[lints]` table, not in crate-level attributes. The baseline is `missing_docs` &
  `missing_debug_implementations` set to warn under `[lints.rust]`, & `clippy::must_use_candidate` set to warn with
  `clippy::module_inception` set to allow under `[lints.clippy]`.
- The README is the crate front page: `#![doc = include_str!("../README.md")]` in `lib.rs`. Its examples compile as
  doc tests, so they must build & pass.
- Libraries have no dependencies by default. Optional dependencies are wired with `dep:` behind a feature named after
  the dependency, with `default-features = false`.
- Publish full docs on docs.rs: `[package.metadata.docs.rs]` with `all-features = true` &
  `rustdoc-args = ["--cfg", "docsrs"]`, plus `#![cfg_attr(docsrs, feature(doc_cfg))]` in `lib.rs`.
- Commit a `rustfmt.toml` with `max_width = 120`.
- Bump the version once, at release, after the planned ISSUES.md items are cleared — never per change or per PR. The
  README install snippet carries the latest published stable version, never a release candidate.
