# AGENTS-IMPROVEMENT-SPEC.md

Audit of agent-guidance quality for `sdrx-backend`.
Date: 2026-04-24
Last updated: 2026-05-01

---

## Audit Summary

### What Was Found

| Artifact | Status |
|---|---|
| `AGENTS.md` | ✅ Present, updated for standalone repo |
| `README.md` | ✅ Present, updated for standalone repo |
| `.env.example` | ✅ Present, trimmed to Docker Hub secrets only |
| `.github/pull_request_template.md` | ✅ Present, updated build command |
| `k8s/deployment-dev.yaml` | ✅ Correct ingress path `/sdrx/api/v1` |
| `k8s/deployment-prod.yaml` | ✅ Correct ingress path `/sdrx/api/v1` |
| `Dockerfile` | ✅ Standalone two-stage build |
| `.github/workflows/deploy_dev.yml` | ✅ CI-only, no frontend/Python/AWS steps |
| `.github/workflows/deploy_prod.yml` | ✅ CI-only, no frontend/Python/AWS steps |

---

## Repository History

This repo was extracted from the `www-travler7282-com` monorepo
(`backends/sdrx/`) and published as a standalone GitHub repository
(`travler7282/sdrx-backend`) in May 2026.

Migration work completed:
1. Rewrote `Dockerfile` for standalone build context (removed `backends/sdrx/` workspace paths).
2. Stripped monorepo CI steps from both workflow files (removed Java setup, Python setup, roboarm/wxstation/devops-assistant image builds, frontend build, S3 deploy, CloudFront invalidation).
3. Fixed pervasive `/srdx/` typo → `/sdrx/` in `src/index.ts` routes and `k8s/` ingress paths.
4. Rewrote `README.md` for standalone repo context.
5. Rewrote `AGENTS.md` to reflect single-service structure (no monorepo).
6. Trimmed `.env.example` to only Docker Hub secrets (removed all AWS_* entries).
7. Fixed `.github/pull_request_template.md` build command (`npm run build:all` → `npm run build`).
8. Renamed default branch `master` → `main`.

---

## What's Good

1. **Single-service focus.** This repo contains exactly one service — clean and easy for agents to reason about.
2. **TypeScript strict mode.** Enforced in `tsconfig.json`.
3. **Node version enforced.** Via `engines` in `package.json`.
4. **Tests present.** Vitest + supertest integration tests in `src/index.test.ts`.
5. **CI runs tests before build.** Both workflows run `npm test` before pushing the Docker image.
6. **Probe paths correct.** Liveness (`/healthz`) and readiness (`/readyz`) match the K8s manifests.

---

## Remaining Gaps

### 1. No unit-level tests for route validation logic
`src/index.test.ts` covers happy-path integration tests. Edge cases for tune
validation (e.g., out-of-range frequencyHz, invalid mode string) are not tested.

### 2. No `.devcontainer` for this standalone repo
The monorepo had a `devcontainer.json`. This standalone repo does not. Adding
one with Node 24 image and forwarded port 8080 would improve agent dev experience.

---

## Multi-Agent File Recommendation

Use `AGENTS.md` as the single source of truth for this repository.

Thin adapter files (`copilot-instructions.md`, `.claude/CLAUDE.md`) reference
`AGENTS.md` as canonical. Update `AGENTS.md` first; adapters in same commit.

---

## Improvement Spec

### P0 — Correctness (complete)

All P0 items resolved during monorepo extraction.

### P1 — Reliability

#### Add edge-case tests for POST /api/sdr/tune
- Out-of-range `frequencyHz`
- Invalid `mode` string
- Missing `Content-Type` header

### P2 — Guidance

#### Add `.devcontainer/devcontainer.json`
Node 24 image, forwarded port 8080, `postCreateCommand: npm install`.

### P3 — Long-term quality (lower urgency)

- Validate thin adapter files (Copilot, Claude, Cursor) load correctly in
  active agent sessions and reference `AGENTS.md` successfully.
