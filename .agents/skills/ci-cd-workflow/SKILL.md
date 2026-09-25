---
name: ci-cd-workflow
scope: generic
description: >-
  Rules, architectures, and guidelines for maintaining GitHub Actions CI/CD workflows,
  container image publishing (GHCR and Docker Hub), security scanning, releases, and deployments.
---

# CI/CD & Deployment Architecture Skill

This skill guides AI agents and contributors in maintaining GitHub Actions CI/CD pipelines, container image publishing, and automated deployments.

---

## 1. Local Branch-First Development & Commit Cadence

> [!IMPORTANT]
> **CREATE A LOCAL BRANCH FIRST & COMMIT FREQUENTLY**:
> Always start by creating a dedicated local branch from `main`:
> ```bash
> git switch -c <developer-or-agent>/main/<feature-name>
> ```
> Commit at each logical milestone (`more commits = more explanatory work`). Never commit directly on `main`.

---

## 2. Multi-Environment CI/CD Pipeline

```yaml
on:
  push:
    branches: ['**']        # every branch, including stacked PR branches
  pull_request:
    branches: [main]        # jobs only do work for PRs from forks (see below)

permissions:
  contents: read            # least privilege; jobs add only what they write

jobs:
  changes:                  # dorny/paths-filter: go / python / frontend / docker / scripts outputs
    if: github.event_name == 'push' || github.event.pull_request.head.repo.full_name != github.repository
  backend-test:
    needs: changes
    if: needs.changes.outputs.go == 'true'
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@<full-commit-sha> # vX.Y.Z
        with:
          persist-credentials: false
      - uses: actions/setup-go@<full-commit-sha> # vX.Y.Z
        with:
          go-version-file: go.mod   # honours the toolchain directive
      - run: git ls-files -z '*.go' | xargs -0 gofmt -l   # must print nothing
      - run: go mod tidy -diff
      - run: go vet ./...
      - run: go test -race -count=1 ./...
      - run: CGO_ENABLED=0 go build ./...
  service-test:             # astral-sh/setup-uv, uv sync --locked, type check, tests
  frontend-test:            # pnpm/action-setup (reads packageManager), install --frozen-lockfile, lint, tsc, build
  smoke:                    # build every image, start the compose stack, wait for each /health
  ci-status:                # needs every job, if: always(), fails on any failure or cancellation
```

### Workflow Hardening Rules

1. **Pin every action by full commit SHA** with a `# vX.Y.Z` comment, on its current major version (Node 24 runtime). Look SHAs up with `gh api repos/<owner>/<action>/commits/<tag> --jq .sha`; Dependabot (`github-actions` ecosystem) keeps SHA and comment in sync.
2. **Pin the runner image** (`ubuntu-24.04`), not `ubuntu-latest`, which moves to a new Ubuntu release without notice.
3. **Least privilege**: top-level `permissions: contents: read`; grant `packages: write`, `pull-requests: write` and similar only to the job that needs them. Check out with `persist-credentials: false` unless a later step must push.
4. **Path filtering with an aggregate check**: skip jobs whose files did not change, and add one always-running status job that fails when any job failed or was cancelled; require only that job in branch protection.
5. **One run per change**: CI runs on `push` to every branch, so the `pull_request` run does work only for PRs from forks (`github.event.pull_request.head.repo.full_name != github.repository`). Add `concurrency` so superseded runs are cancelled (except on the default branch).
6. **Never report work that did not happen**: a step whose secret is missing must skip with a `::warning::` or `::notice::`, and summaries must state what actually ran. Secrets cannot appear in `if:`; test them in a step (`[ -n "$VAR" ]`) and use its output.
7. **No real credentials in tests**: test jobs use mock values, never repository secrets.
8. **Private repositories on free plans**: artifacts and caches count against quota; disable build-record uploads (`DOCKER_BUILD_RECORD_UPLOAD: "false"`) and keep one cache scope per image.
9. Lint workflows with `actionlint` and `zizmor`; justify any `zizmor: ignore[...]` inline.

---

## 2b. Image Publishing, Security Audit & Releases

- **Images**: build `linux/amd64,linux/arm64` with `docker/build-push-action` (`sbom: true`, `provenance: mode=max`), push to `ghcr.io/<owner lowercase>/<image>` with the built-in `GITHUB_TOKEN` (`packages: write`), and to Docker Hub only when its secrets exist. Tag `sha-<short>`, `latest` on the default branch and semver on `v*` tags (`flavor: latest=false`). Use `paths-ignore` so bot data commits and documentation-only pushes do not publish.
- **Deployments** that a platform performs from Git (for example Vercel's Git integration or a Render blueprint with `autoDeploy`) need no deploy step; do not add steps that silently skip.
- **Smoke tests**: CI must run the images it builds. Start the compose stack as an isolated project (no published ports, generated secrets, fresh volumes), wait with `docker compose up --wait`, probe each health endpoint, and print `docker compose logs` on failure.
- **Security audit workflow** (PRs, default-branch pushes, weekly schedule): language audits (`govulncheck`, `pip-audit` on the exported lockfile, `pnpm audit --audit-level high`), `gitleaks` over git history with a value-based allowlist for test fixtures, a Trivy filesystem scan failing only on fixable CRITICAL findings, and workflow linting. Without GitHub code scanning (private repositories on free plans), report in job logs and step summaries.
- **Dependency updates**: `.github/dependabot.yml` with weekly grouped updates for every ecosystem (language packages, Dockerfile base images, GitHub Actions).
- **Releases**: a `v*` tag runs GoReleaser (static `CGO_ENABLED=0` binaries, archives, `checksums.txt`, GitHub release) with `contents: write` on that job only and no build cache; use the matching `CHANGELOG.md` section as release notes. Validate locally with `goreleaser check` and `goreleaser release --snapshot --clean --skip=publish`.

---

## 3. Local CI Mirroring Runbook

To guarantee that your changes pass CI before committing:

```bash
# 1. Run the fast pre-commit checks, then the pre-push builds
make pre-commit
make pre-push

# Build the images and smoke-test the container stack
make smoke

# 2. Alternatively, run individual CI jobs locally:
# Go Backend:
go test -v -race ./... && go build -v ./...

# Python Service:
uv run pytest

# Frontend:
pnpm run lint && pnpm run build
```

---

## 4. Secrets vs Centralized Configuration

- **Secrets**: Strictly defined in `.env` / CI Secrets and runtime dashboards (`DATABASE_URL`, `API_KEY`, `JWT_SECRET`).
- **Operational Defaults**: Centralized in code modules.
- Never hardcode secrets in CI workflow YAML or Git commits.

---

## 4. DSA Confidence Engine — CI Reference

> [!IMPORTANT]
> **MANDATORY**: Every repository MUST have a CI workflow file at `.github/workflows/ci.yml`. Without it, there is no automated quality gate on Pull Requests.

For the `dsa-confidence-engine` (Go 1.25 monorepo), the CI pipeline runs three jobs sequenced as:

```
go-lint (gofmt check)
    └── go-test (go test -v -race ./...)
    └── go-build (go build ./... + go mod tidy check)
```

**Local mirrors of CI jobs** (run these before pushing):

```bash
# Mirror go-lint:
gofmt -l .                            # Must return empty output

# Mirror go-test:
go test -v -race -count=1 ./...

# Mirror go-build:
go build -v ./...
go mod tidy && git diff --exit-code go.mod go.sum
```

**CI triggers:**
- Every `git push` to **any branch** — fast feedback loop
- Every `pull_request` targeting **`main`** — enforced quality gate before merge

**Required workflow file:** `.github/workflows/ci.yml`
See `.agents/skills/github-pr-issue-automation/SKILL.md` Section 5 for the full list of required workflows.

