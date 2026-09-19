---
name: github-repo-metadata-rewriter
scope: codebase-github-repo-metadata-rewriter
description: >-
  System architecture, microservice topology, git history rewrite pipelines, AI timestamp distribution engine, sandboxed workspaces, and Go + Templ frontend runbooks for github-repo-metadata-rewriter.
---

# GitHub Repo Metadata Rewriter — Architecture & Developer Guide

> [!IMPORTANT]
> **CODEBASE-SPECIFIC SCOPE**: This skill is strictly specific to the **GitHub Repo Metadata Rewriter** (`github-repo-metadata-rewriter`). It details concrete service topology, sandboxed Git workspaces, commit graph manipulation pipelines, AI timestamp models, and Go + Templ UI runbooks.

---

## 1. System Topology

```text
┌─────────────────────────────────────────────────────────────┐
│                 Go + Templ Web Dashboard                   │
│         (Compiled Type-Safe SSR Frontend /web)              │
└───────────────┬─────────────────────────────┬───────────────┘
                │ Form Submission / Job Trigger│ Live Telemetry Polling
                ▼                             ▼
┌─────────────────────────────────────────────────────────────┐
│                  Go Backend REST Server                     │
│        (HTTP API · Job Store · Preview Diff Service)        │
└───────────────┬─────────────────────────────────────────────┘
                │ Work Queue / Context Dispatch
                ▼
┌─────────────────────────────────────────────────────────────┐
│                 Rewriter Worker Engine                      │
│   (Sandboxed Git Executor · Graph Parser · LLM Client)      │
└───────┬──────────────────────┬──────────────────────┬───────┘
        │ Bare Clone / Fetch   │ AI Distribution API  │ Filter-Branch
        ▼                      ▼                      ▼
┌──────────────────┐   ┌──────────────────┐   ┌──────────────────┐
│ Sandboxed /tmp   │   │ OpenRouter /     │   │ Target Git       │
│ Workspace        │   │ Gemini APIs      │   │ Remote (SSH)     │
└──────────────────┘   └──────────────────┘   └──────────────────┘
```

---

## 2. Microservice Topology & Pipeline Stages

### Backend API (`backend/`)
- **Port**: `8080` (configurable via `PORT`).
- **Key Endpoints**:
  - `GET /` — Serves the compiled `templ` Git Telemetry Dashboard.
  - `GET /health` — Service readiness & liveness probe.
  - `POST /api/jobs` — Submits a new metadata rewriting job.
  - `GET /api/jobs` — Lists active and archived rewriting jobs.
  - `GET /api/jobs/:id` — Real-time telemetry, stage logs, and status.
  - `POST /api/jobs/:id/preview` — Generates a diff of rewritten commits before remote push.
  - `POST /api/jobs/:id/push` — Authorizes remote push with force-with-lease safety.

### Rewriter Worker Engine (`rewriter-worker/`)
- **Stage 1 (Sandboxed Clone)**: Clones target repository into isolated directory `/tmp/rewriter-workspaces/<job-id>` with reflog backup.
- **Stage 2 (Graph Extraction)**: Traverses commit topology (`git log --reverse --format="%H|%an|%ae|%ad|%s"`).
- **Stage 3 (Timestamp Computation)**: Computes realistic commit timestamp distributions across specified start/end window via AI or deterministic spacing.
- **Stage 4 (Git History Rewrite)**: Executes automated `git filter-branch` / `git-filter-repo` re-anchoring `GIT_AUTHOR_DATE` and `GIT_COMMITTER_DATE`.
- **Stage 5 (Verification & Integrity Check)**: Executes `git fsck --full` to guarantee zero object graph corruption.
- **Stage 6 (Remote Push)**: Pushes rewritten branches via SSH remote alias.

### Templ Web Frontend (`web/`)
- Server-rendered HTML powered by Go Templ (`.templ`).
- Features commit timeline visualization, interactive job configuration form, stage progress indicators, and audit logging.

---

## 3. Development Runbook

```bash
# Generate Go code from Templ files
make templ

# Build backend and worker binaries
make build

# Run application locally
make run

# Run Docker Compose stack
docker compose up -d --build
```

---

## 4. Proprietary Licensing & Closed-Source Governance

This codebase is **PROPRIETARY AND CLOSED SOURCE** owned by **Shardendu Mishra**.
- All rights reserved under the 2026 Proprietary Software License.
- No reproduction, distribution, unauthorized reverse engineering, or external exposure permitted.
