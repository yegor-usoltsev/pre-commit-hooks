# pre-commit-hooks

Shared [pre-commit](https://pre-commit.com) hooks for linting shell scripts, Dockerfiles, GitHub Actions workflows, Renovate configs and Go code, and for catching secrets before they are committed. Each hook pins its tool to an exact version, so every repository that uses this one runs the same linters, and Renovate keeps them current in one place.

- **Works with prek and pre-commit.** The hooks use the standard `.pre-commit-hooks.yaml` manifest and are tested with both [prek](https://prek.j178.dev) and pre-commit.
- **Native tools.** Each hook installs a prebuilt binary or builds a Go tool once; nothing runs in Docker.
- **One version per tool.** Versions live in this repository, so a consumer bumps a single `rev` to update every linter.

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/yegor-usoltsev/pre-commit-hooks
    rev: v1.0.0
    hooks:
      - id: gitleaks
      - id: shellcheck
      - id: hadolint
```

## Hooks

| ID | Tool | Checks |
| --- | --- | --- |
| `gitleaks` | [gitleaks](https://github.com/gitleaks/gitleaks) | secrets in the staged changes |
| `shellcheck` | [ShellCheck](https://www.shellcheck.net) | shell scripts |
| `shfmt` | [shfmt](https://github.com/mvdan/sh) | shell script formatting (fixes files; style from `.editorconfig`) |
| `hadolint` | [hadolint](https://github.com/hadolint/hadolint) | Dockerfiles; fails on errors, reports warnings |
| `actionlint` | [actionlint](https://github.com/rhysd/actionlint) | GitHub Actions workflows |
| `zizmor` | [zizmor](https://docs.zizmor.sh) | security of GitHub Actions workflows, offline |
| `check-renovate` | [check-jsonschema](https://github.com/python-jsonschema/check-jsonschema) | `renovate.json` against the Renovate schema |
| `golangci-lint` | [golangci-lint](https://golangci-lint.run) | the whole Go module, with `--fix` |

The exact tool versions are in [`.pre-commit-hooks.yaml`](.pre-commit-hooks.yaml).

## Usage

1. Install [prek](https://prek.j178.dev/installation/) or [pre-commit](https://pre-commit.com/#install).
2. Add this repository to `.pre-commit-config.yaml` with the hooks you need, as in the example above. Hooks with no matching files are skipped, so the same list works across repositories.
3. Install the Git hook with `prek install` (or `pre-commit install`), then check the existing files:

   ```bash
   prek run --all-files
   ```

The `golang` hooks build with the Go version pinned in their `language_version`, which prek and pre-commit download when the local Go differs. The `python` hooks install prebuilt wheels.

Override `args`, `files` or `exclude` per hook in your config like for any other hook. zizmor reads `.github/zizmor.yml`; to allow version tags such as `actions/checkout@v7` instead of commit hashes, add:

```yaml
rules:
  unpinned-uses:
    config:
      policies:
        "*": ref-pin
```

## Updates

Renovate updates the tool versions here and merges them when CI passes; every merge to `main` is tagged with the next patch version. To follow those tags, enable Renovate's [pre-commit manager](https://docs.renovatebot.com/modules/manager/pre-commit/) in your repository with the `:enablePreCommit` preset, or run `prek update` (`pre-commit autoupdate`).

## License

[MIT](LICENSE)
