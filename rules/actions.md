---
paths:
  - ".github/**"
---

# GitHub Actions

- Workflows run on pull requests to the default branch, with `permissions: contents: read` & a concurrency group keyed
  on the workflow & ref with `cancel-in-progress: true`.
- Split jobs into gates & advisories. The gates must pass: format, test, & package. Advisories carry
  `continue-on-error: true` & a comment saying why — they cover what can break for reasons outside the pull request,
  like nightly toolchains & the latest resolvable dependency versions.
- End with a job named `build` that has `needs: [ ... ]`, `if: always()`, & one
  `test "${{ needs.<job>.result }}" = "success"` line per gate. That job is the single required check.
- Gate jobs pass `--locked` so they test the committed lock file. Deny warnings everywhere:
  `cargo clippy --all-targets -- -D warnings` & `RUSTDOCFLAGS: -D warnings` on `cargo doc`.
- Test a matrix over the features: no features, each feature alone, & `--all-features`.
- Cache with `Swatinem/rust-cache` & a distinct `key` per job.
- Dependabot runs monthly. Group the cargo ecosystem into one pull request, & set `commit-message.prefix` per
  ecosystem to the repo's commit scope for it.
