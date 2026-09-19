---
name: repo-transfer-automation-system
scope: codebase-repo-transfer-automation-system
description: >-
  System architecture, microservice topology, GitHub REST API rate limits, exponential backoff transfer protocols, and Go + Templ frontend runbooks for repo-transfer-automation-system.
---

# Repo Transfer Automation System — Architecture & Developer Guide

> [!IMPORTANT]
> **CODEBASE-SPECIFIC SCOPE**: This skill is strictly specific to the **Repo Transfer Automation System** (`repo-transfer-automation-system`). It details concrete service topology, transfer retry pipelines, GitHub REST API rate limits, and Go + Templ UI execution runbooks.

---

## 1. System Topology

```text
┌─────────────────────────────────────────────────────────────┐
│                 Go + Templ Web Dashboard                   │
│         (Compiled Type-Safe SSR Frontend /web)              │
└───────────────┬─────────────────────────────┬───────────────┘
                │ Form Actions / Fetch        │ Live Status Polling
                ▼                             ▼
┌─────────────────────────────────────────────────────────────┐
│                  Go Backend REST Server                     │
│    (Fiber / Standard HTTP · Auth, Repos, Job Dispatch)     │
└───────────────┬─────────────────────────────────────────────┘
                │ In-Memory / Channel Dispatch
                ▼
┌─────────────────────────────────────────────────────────────┐
│                 Transfer Worker Engine                      │
│   (Exponential Backoff · Rate Limiter · GitHub REST API)    │
└───────────────┬─────────────────────────────────────────────┘
                │ HTTPS (Personal Access Token / OAuth)
                ▼
┌─────────────────────────────────────────────────────────────┐
│                    GitHub REST API v3                       │
│        (POST /repos/{owner}/{repo}/transfer)                │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Microservice Topology & Boundaries

### Backend API (`backend/`)
- **Port**: `8080` (configurable via `PORT`).
- **Authentication**: GitHub OAuth 2.0 Web Application Flow and PAT Bearer Token support.
- **Key Endpoints**:
  - `GET /` — Serves the compiled `templ` Web Dashboard.
  - `GET /health` — Service readiness & liveness probe.
  - `GET /api/auth/login` — Initiates GitHub OAuth authentication.
  - `GET /api/auth/callback` — Exchanges OAuth code for GitHub user access token.
  - `GET /api/repos` — Retrieves user & organization repositories.
  - `POST /api/transfer` — Enqueues repository transfer jobs.
  - `GET /api/transfer/status/:id` — Real-time execution status & telemetry.
  - `GET /api/transfer/history` — Audit trail of completed and pending transfers.

### Transfer Worker Engine (`transfer-worker/`)
- Concurrently processes transfer queue with exponential backoff (`2s` initial wait, factor `2.0`, max `10s` cap with jitter).
- Intercepts GitHub `403 Forbidden` / `429 Too Many Requests` secondary rate limits and pauses the transfer pipeline until `x-ratelimit-reset`.
- Preserves audit logs for every repository transfer outcome (target user acceptance required or instant transfer).

### Templ Web Frontend (`web/`)
- Minimalist, high-performance server-rendered UI written in Go Templ (`.templ`).
- Zero Node.js runtime footprint in production.
- Responsive dark-mode layout matching GitHub enterprise tooling standards.

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
- No distribution, decompilation, reproduction, or public exposure permitted.
