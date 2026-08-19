# Git

- Never commit or modify git state. When git operations are needed, give the commands instead of running them.
- Commit messages & pull request titles are `[scope] Sentence describing the change.` — a bracketed scope, a
  capitalized sentence, & a trailing period.
- The scope is the module or folder touched, `cargo` for the manifest & build config, `actions` for CI, or `*` when
  the change spans the project. Several scopes concatenate: `[ip][socket]`.
