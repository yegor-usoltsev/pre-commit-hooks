# pre-commit-hooks

`.pre-commit-hooks.yaml` defines one hook per tool. Each hook pins its tool in `additional_dependencies` on one line, `[name==X.Y.Z]` for PyPI and `[module/path@vX.Y.Z]` for Go: the regex managers in `.github/renovate.json` only match that form. Prefer a PyPI wheel that ships the prebuilt binary; otherwise build the Go module with `language: golang`. Do not add Docker hooks. `pyproject.toml` and `go.mod` exist only so the python and golang installs succeed.

A new hook needs a fixture in `tests/fixture/` that exercises it, a row in the README table, and a passing run of the CI steps: `prek validate-manifest .pre-commit-hooks.yaml`, then `prek try-repo` and `pre-commit try-repo --ref` against a Git repository made from `tests/fixture/` (see `.github/workflows/ci.yml`).

Consumers pin a tag, and prek and pre-commit cache a repository per tag, so never move or delete a tag. CI tags every commit on `main` with the next patch version. Renovate skips Go major updates because the new module path (`/v2` → `/v3`) needs a manual edit.

Hook IDs are a public contract: renaming or removing one breaks consumer configs, so add a new ID instead.
