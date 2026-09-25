---
name: precommit-workflow-management
scope: generic
description: >-
  Rules, architecture, and runbooks for configuring, updating, and operating the intelligent Git pre-commit workflow.
---

# Pre-Commit Workflow Management Skill

This skill explains how to maintain, configure, and execute the intelligent pre-commit hook workflow across polyglot repositories.

---

## 1. Local Branch-First Development

> [!IMPORTANT]
> **CREATE A LOCAL BRANCH FIRST**: Always start by creating a dedicated local branch from `main`:
> ```bash
> git switch -c <developer-or-agent>/main/<feature-name>
> ```
> Never make changes directly on `main`.

---

## 2. Pre-Commit and Pre-Push Architecture

The hooks live in `.githooks/` and are tracked directly in version control. Commits must stay fast, so the work is split:

- **`pre-commit`**: fast checks only (secret scan, formatting, lint, static analysis, unit tests of the changed areas).
- **`pre-push`**: re-runs the pre-commit checks for the commits being pushed, then the slow builds (binaries, frontend production build, container images when container files change).

```text
Staged Changes Detected (git diff --cached)          [pre-commit]
  │
  ├── Only Go files changed ───────► Formatting, vet, tests of the changed packages
  ├── Only Python files changed ───► Pyright/Ruff, test suites
  ├── Only Frontend files changed ─► Biome/Linter, tsc --noEmit
  ├── Only Documentation changed ──► Fast-path bypass (<0.2s)
  └── Global / Config / Manual ────► Every fast check

Pushed Commits Detected (stdin refs from git push)   [pre-push]
  │
  ├── Re-run the pre-commit checks for <remote sha>..<local sha>
  ├── Code changed ────────────────► Compile binaries / frontend production build
  └── Container files changed ─────► Build the affected images (cache only)
```

---

## 3. Key Features

1. **Selective Staged-File Execution**: Avoids unnecessary work by only running checks relevant to the files staged for the commit.
2. **Fast-Path for Documentation**: Commits containing only markdown (`*.md`), text (`*.txt`), or asset files skip every check.
3. **Builds at Push Time**: Compilation and image builds run once per push instead of on every commit; the pre-push hook reads the pushed ref range from stdin and falls back to a full run when it cannot determine it.
4. **Subshell Directory Isolation**: Every check runs in a subshell `(cd "${REPO_ROOT}" && ...)` to prevent current working directory drift.
5. **Fail-Fast with Remediation**: Aborts immediately on failure and outputs exact commands needed to fix the problem.
6. **Zero External Runtime Dependencies**: Implemented in portable bash, requiring no heavy wrapper packages at the repository root. Keep it compatible with the bash 3.2 macOS ships: no associative arrays, `mapfile`/`readarray` or `${var,,}`, and no empty arrays under `set -u`.

---

## 4. Operations & Maintenance Runbook

### Activating the Hook Locally
```bash
make hooks-install
# Or: ./scripts/install-hooks.sh
```

### Manually Running the Gates
```bash
make pre-commit   # every fast check
make pre-push     # every check plus every build, without pushing
```

### Bypass Flags (Emergency Only)
```bash
# Skip tests for quick drafts
SKIP_TESTS=1 git commit -s -S -m "chore: draft"

# Skip the push-time builds (or only the container image builds)
SKIP_BUILDS=1 git push
SKIP_DOCKER=1 git push

# Force full validation across all subsystems
PRECOMMIT_ALL=1 git commit -s -S -m "feat: core change"
PREPUSH_ALL=1 git push

# Skip a hook entirely
SKIP_HOOKS=1 git commit -s -S -m "wip: draft"   # or PRECOMMIT_BYPASS=1
PREPUSH_BYPASS=1 git push
```
