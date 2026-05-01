# sdr-express-app

Express + TypeScript backend for the SDR Angular UI.

## Features

- Health endpoints for k3s probes: `/healthz`, `/readyz`
- SDR status endpoint: `GET /api/sdr/status`
- SDR tune endpoint: `POST /api/sdr/tune`
- Docker-ready build for k3s deployment

## Local development

```bash
npm run dev --workspace sdr-express-app
```

Server defaults:

- Host: `0.0.0.0`
- Port: `8080`
- CORS origin: `*` (configure in prod)

## API quick test

```bash
curl http://localhost:8080/api/sdr/status
curl -X POST http://localhost:8080/api/sdr/tune \
  -H "Content-Type: application/json" \
  -d '{"frequencyHz":14200000,"bandwidthHz":2800,"mode":"USB"}'
```

## Build container image

```bash
docker build -t sdr-express-app:local ./backends/sdr-express-app
```

## k3s deployment

Update image in `k8s/deployment.yaml`, then:

```bash
kubectl apply -f backends/sdr-express-app/k8s/deployment.yaml
kubectl rollout status deployment/sdr-express-app
```

## Endpoints

All endpoints are available at both the root and under `/srdx/api/v1`.

### GET /healthz
- Health check
- Response: `{ "status": "ok" }`

### GET /readyz
- Readiness check
- Response: `{ "ready": true, "service": "sdr-express-app" }`

### GET /api/sdr/status
- Returns current SDR state
- Response: JSON object with frequency, bandwidth, gain, etc.

### POST /api/sdr/tune
- Tunes SDR to new parameters
- Request body: `{ "frequencyHz": number, "bandwidthHz": number, "gainDb": number, "sampleRateHz": number, "mode": "AM"|"FM"|... }`
- Response: updated SDR state or error
# sdrx-backend
