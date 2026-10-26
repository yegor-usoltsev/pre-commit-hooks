# Allow Go toolchain patch upgrades in golangci-lint

The golangci-lint hook pins Go 1.27.2 through `language_version`. prek runs that environment with `GOTOOLCHAIN=local`, so loading a consumer module requiring Go 1.27.3 fails before linting starts.

## Evidence

- Gitea `yegor/gitea-config-webhook` PR #13, Drone build #36: the Go tests/build passed; the golangci-lint hook failed with `go.mod requires go >= 1.27.3 (running go 1.27.2; GOTOOLCHAIN=local)`.
- Reproduced locally with `prek run golangci-lint --all-files --show-diff-on-failure`.
- Consumer commit `d45657f` overrides the entry with `env GOTOOLCHAIN=auto golangci-lint run --fix`; the failing command and full hook suite then pass.
- Four other consumers pin v1.0.2 without an override: `renovate-bot/renovate-triggerer`, `yegor-usoltsev/drone-email-webhook`, `yegor-usoltsev/drone-trigger-build`, and `yegor-usoltsev/go-serve-s3`. They currently require Go 1.27.2. Each fails when its prepared hook environment loads an isolated module requiring Go 1.27.3; the same check passes with `GOTOOLCHAIN=auto`.
- golangci-lint v2.14.0 permits newer Go patch versions in the same language version; a newer minor Go version still requires a compatible linter binary.

## Resolution

The shared golangci-lint entry enables `GOTOOLCHAIN=auto` while keeping the pinned compiler for installation. The fixture now requires Go 1.27.3, newer than the hook's Go 1.27.2, so both prek and pre-commit exercise automatic toolchain selection. Consumers can update the hook tag and remove their temporary entry overrides.
