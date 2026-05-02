# AGENTS.md — Agent Instructions for sdrx-backend

This file tells AI coding agents (Copilot, Cursor, Claude, etc.) how this
repository is structured and how to work within it correctly.

---

## Repository Overview

Standalone Express + TypeScript backend API for the
[SDRx](https://www.travler7282.com/sdrx/) Software Defined Radio Receiver UI.
Owner: Michael Hunt (travler7282).
GitHub: https://github.com/travler7282/sdrx-backend

---

## Repository Layout

```
src/
  index.ts        # Express app — routes, middleware, SDR state, graceful shutdown
  index.test.ts   # Vitest + supertest integration tests

k8s/
  deployment-dev.yaml   # K3s Deployment + Service + Ingress + Certificate (dev)
  deployment-prod.yaml  # K3s Deployment + Service + Ingress + Certificate (prod)

Dockerfile          # Two-stage build: compile TypeScript → minimal runtime image
package.json        # Project manifest, npm scripts, engines (Node 24)
tsconfig.json       # TypeScript compiler config (ES2022, NodeNext, strict)
vitest.config.ts    # Vitest config (node environment, v8 coverage)
```

---

## Branch and Environment Mapping

| Branch | Environment | Docker tag |
|---|---|---|
| `dev` | Development — dev-api.travler7282.com | `travler7282/sdrx:<version>-dev` |
| `main` | Production — api.travler7282.com | `travler7282/sdrx:<version>` |

Feature branches are merged into `dev` first, then `dev` to `main` via PR.

---

## Tech Stack

- **Runtime**: Node 24 (enforced via `engines` in `package.json`)
- **Framework**: Express 4
- **Language**: TypeScript `~5.8.3`, strict mode
- **Build**: `tsc` → `dist/`
- **Dev server**: `tsx watch src/index.ts`
- **Testing**: Vitest + supertest
- **Container**: Docker Hub `travler7282/sdrx`, deployed to K3s on Traefik + cert-manager
- **Listening port**: 8080

---

## Common Commands

```bash
# Install dependencies
npm install

# Start dev server (hot-reload)
npm run dev

# Build TypeScript
npm run build

# Run tests
npm test

# Run tests with coverage
npm run test:coverage
```

---

## API Routes

All routes are served at both the root path and the `/sdrx/api/v1` prefix.

| Method | Path | Description |
|---|---|---|
| GET | `/healthz` | Liveness probe — returns `{ status: "ok" }` |
| GET | `/readyz` | Readiness probe — returns `{ ready: true, ... }` |
| GET | `/api/sdr/status` | Current SDR state |
| POST | `/api/sdr/tune` | Tune SDR parameters |

Middleware (required on all routes): `helmet`, `cors`, `morgan`, `express.json`.

---

## CI/CD Pipeline

Both workflows follow the same steps:

1. Checkout → Setup Node 24 → `npm ci` → `npm test`
2. Docker login (DOCKERHUB_USERNAME / DOCKERHUB_TOKEN secrets)
3. Resolve image version from `package.json`
4. Build and push `docker.io/travler7282/sdrx:<version>[-dev]` to Docker Hub

**Required GitHub Secrets** (never hardcode or log):
`DOCKERHUB_USERNAME`, `DOCKERHUB_TOKEN`

---

## K8s Deployment

Manifests in `k8s/` are applied **manually** by the owner. Agents must not
modify these files unless explicitly asked, and must never assume CI deploys them.

Ingress path prefix: `/sdrx/api/v1`
TLS: cert-manager + Let's Encrypt via Traefik

### Rollout triage order (do not skip steps)

1. `kubectl rollout status deployment/sdrx-dev` (or `sdrx-prod`)
2. `kubectl get pods -o wide`
3. `kubectl describe pod <failing-pod>`
4. `kubectl logs --previous <failing-pod>`
5. `kubectl get deployment <name> -o yaml` — confirm effective probe paths

Probes:
- Liveness: `/healthz`
- Readiness: `/readyz`

---

## Versioning Policy

Version is defined in `package.json`. Follow semantic versioning:

| Change type | Bump |
|---|---|
| Bug fixes, probe/path fixes | Patch (`x.y.Z`) |
| New endpoints, backward-compatible features | Minor (`x.Y.z`) |
| Breaking API changes, removed endpoints | Major (`X.y.z`) |

Always bump version before a release-impacting change is merged.

---

## Code Conventions

- TypeScript strict mode; `~5.8.3` pinned.
- No UI framework — this is a backend-only repo.
- `helmet` + `cors` + `morgan` middleware required on all route handlers.
- Tests live in `src/index.test.ts` using Vitest + supertest.
- Do not add new dependencies without confirming Node 24 compatibility.

---

## Git and PR Workflow

1. **Create a GitHub issue** describing the work (label: `ai-generated`).
2. **Create a feature branch** from `main` named after the issue
   (e.g., `42-fix-healthz-route`).
3. **Commit changes** to that branch.
4. **Open a PR** targeting `dev` (label: `ai-generated`).
5. The owner reviews, merges to `dev` → CI deploys the dev Docker image.
6. The owner opens a PR from `dev` → `main` when satisfied → CI deploys prod.

Agents must never commit directly to `main` or `dev`.

---

## Secret Handling Policy (hard requirement)

- Never print, echo, or commit secret values.
- Secrets used: DOCKERHUB_USERNAME, DOCKERHUB_TOKEN.
- No AWS secrets exist in this repo — AWS S3/CloudFront is not used here.

---

## What Agents Should NOT Do

- Do not commit directly to `main`. Use feature branch → `dev` → `main` flow.
- Do not hardcode or log secret values.
- Do not modify `.github/workflows/` without understanding the full CI pipeline.
- Do not change the TypeScript version without verifying compatibility.
- Do not add AWS, S3, CloudFront, or frontend-build steps to the workflows —
  this repo only builds and publishes a Docker image.
