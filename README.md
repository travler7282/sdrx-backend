# sdrx-backend

Express + TypeScript backend API for the SDRx (Software Defined Radio Receiver) Angular UI.

## Features

- Health endpoints for K3s probes: `GET /healthz`, `GET /readyz`
- SDR status endpoint: `GET /api/sdr/status`
- SDR tune endpoint: `POST /api/sdr/tune`
- All routes also available under the `/sdrx/api/v1` prefix
- Docker-ready build for K3s deployment

## Local development

```bash
npm install
npm run dev
```

Server defaults:

| Variable | Default | Description |
|---|---|---|
| `PORT` | `8080` | Listening port |
| `HOST` | `0.0.0.0` | Listening host |
| `CORS_ORIGIN` | `*` | Allowed CORS origin |

## Scripts

| Command | Description |
|---|---|
| `npm run dev` | Start dev server with hot-reload (`tsx watch`) |
| `npm run build` | Compile TypeScript to `dist/` |
| `npm start` | Run the compiled build |
| `npm test` | Run tests with Vitest |
| `npm run test:coverage` | Run tests with coverage report |

## API quick test

```bash
curl http://localhost:8080/healthz
curl http://localhost:8080/api/sdr/status
curl -X POST http://localhost:8080/api/sdr/tune \
  -H "Content-Type: application/json" \
  -d '{"frequencyHz":14200000,"bandwidthHz":2800,"mode":"USB"}'
```

## Build container image

```bash
docker build -t sdrx-backend:local .
```

## K3s deployment

Update the image tag in `k8s/deployment-dev.yaml` or `k8s/deployment-prod.yaml`, then apply manually:

```bash
kubectl apply -f k8s/deployment-dev.yaml
kubectl rollout status deployment/sdrx-dev
```

## Endpoints

All endpoints are available at both the root path and under the `/sdrx/api/v1` prefix.

### GET /healthz

- Liveness check
- Response: `{ "status": "ok" }`

### GET /readyz

- Readiness check
- Response: `{ "ready": true, "service": "sdr-express-app" }`

### GET /api/sdr/status

- Returns current SDR state
- Response: JSON object with `frequencyHz`, `bandwidthHz`, `gainDb`, `sampleRateHz`, `mode`, `connected`, `updatedAt`

### POST /api/sdr/tune

- Tunes SDR parameters (all fields optional)
- Request body: `{ "frequencyHz": number, "bandwidthHz": number, "gainDb": number, "sampleRateHz": number, "mode": "AM"|"FM"|"USB"|"LSB"|"CW" }`
- Response: updated SDR state or `400` error with validation message

## Branch and environment mapping

| Branch | Environment | Docker tag suffix |
|---|---|---|
| `dev` | Development | `-dev` |
| `main` | Production | (none) |

Feature branches are merged into `dev` first, then `dev` to `main` via PR.
