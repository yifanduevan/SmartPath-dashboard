# Smart Path Dashboard

## Authors

- Kelly Deng, M.Eng., Electrical and Computer Engineering, University of Waterloo, `y99deng@uwaterloo.ca`
- Evan Du, M.Eng., Electrical and Computer Engineering, University of Waterloo, `y242du@uwaterloo.ca`

Smart Path Dashboard is a full-stack monitoring and control UI for the SmartPath adaptive routing system. It provides a web dashboard for backend health, Sysdig metrics, routing weights, UCB scores, and workload execution, with a small Node/Express service that can either proxy a live SmartPath deployment or serve mock data for local development.

## Overview

The dashboard sits alongside the main SmartPath system and gives you a clearer operational view of how the adaptive gateway is behaving.

It includes:

- A React + Vite frontend for live status visualization
- An Express backend that proxies SmartPath analyzer/gateway endpoints
- Mock mode for local UI development without a live cluster
- Workload triggering through Locust
- Views for backend status, routing weights, UCB values, and Sysdig metrics

## Architecture

```text
SmartPath Dashboard Frontend
  |
  v
Dashboard Backend (Express)
  |----> SmartPath Managing System API
  |        /status
  |        /sysdig
  |        /ucb
  |        /weight
  |
  |----> Graph metrics endpoint
  |
  |----> Locust workload runner
```

### Project roles

- `frontend/`: React dashboard UI with polling, charts, backend cards, and workload controls.
- `backend/`: Express API layer that proxies live SmartPath endpoints or returns local mock data.
- `k8s/`: Kubernetes/OpenShift deployment manifests for the dashboard frontend and backend.
- `locustfile.py`: Load-generation script used by the backend workload runner.

## What the dashboard shows

The current implementation surfaces:

- Backend pool status and health
- Backend request totals, successes, and derived failure rate
- Sysdig CPU, memory, and latency metrics
- Routing-weight distribution
- UCB values and best-backend selection
- Workload job status and Locust execution results

## How data flows

The frontend polls the dashboard backend every few seconds. The backend then:

- serves mock responses when no live SmartPath URL is configured
- proxies `/status`, `/sysdig`, `/ucb`, and `/weight` to the managing system when `MANAGING_BASE_URL` is set
- proxies `/graph-metrics` when `GRAPH_METRICS_BASE_URL` is configured
- starts and tracks Locust workloads through `/workload`

## Repository structure

```text
backend/                Express proxy/mock API and workload runner
frontend/               React + Vite dashboard
k8s/                    Deployment manifests
locustfile.py           Locust scenario used by workload execution
README.md               Project overview
```

## Local development

### Prerequisites

- Node.js and npm
- Python and Locust if you want workload execution to work locally

### Install dependencies

From the repo root:

```bash
npm install
npm run install:all
```

### Run frontend and backend together

```bash
npm run dev
```

This starts:

- frontend on `http://localhost:5173`
- backend on `http://localhost:4000`

## Environment variables

### Root scripts

- No required variables for basic local startup

### Backend

- `PORT`: backend port, default `4000`
- `MANAGING_BASE_URL`: SmartPath managing-system base URL for `/status`, `/sysdig`, `/ucb`, and `/weight`
- `GRAPH_METRICS_BASE_URL`: base URL for graph metrics proxying
- `WORKLOAD_API_KEY`: optional API key protecting workload endpoints
- `WORKLOAD_ALLOWED_HOSTS`: comma-separated allowlist for workload target hosts
- `LOCUST_BIN`: Locust executable path, default `locust`
- `LOCUST_FILE_PATH`: path to the Locust scenario file
- `WORKLOAD_OUTPUT_DIR`: directory for generated Locust reports
- `ALLOWED_ORIGINS`: comma-separated CORS allowlist

### Frontend

- `VITE_BACKEND_BASE_URL`: dashboard backend base URL, default `http://localhost:4000`
- `VITE_WORKLOAD_API_KEY`: optional API key sent to protected workload routes
- `VITE_DEFAULT_WORKLOAD_HOST`: default target host shown in the workload form

## API surface

### Dashboard backend endpoints

- `GET /`: backend health summary and endpoint list
- `GET /health`: simple health check
- `GET /status`: backend status from mock data or the managing system
- `GET /sysdig`: Sysdig metrics from mock data or the managing system
- `GET /ucb`: UCB metrics from the managing system
- `GET /weight`: routing weights from the managing system
- `GET /graph-metrics`: graph metrics proxy endpoint
- `POST /workload`: start a Locust workload
- `GET /workload/:jobId`: inspect workload execution state
- `GET /workload/:jobId/report/:kind`: fetch workload artifacts such as HTML or CSV reports

## Mock mode vs live mode

### Mock mode

If `MANAGING_BASE_URL` is not set, the backend serves local mock status and metrics so the UI can be developed without the main SmartPath system running.

### Live mode

If `MANAGING_BASE_URL` is set, the backend proxies SmartPath analyzer/gateway data and the dashboard reflects live backend behavior.

## Deployment

The repo includes Kubernetes/OpenShift manifests for both services under `k8s/`.

These cover:

- frontend deployment
- backend deployment
- backend config map
- backend secret

Adjust image names, routes, and environment values to match your cluster before applying them.

## Notes

- Workload execution depends on Locust being installed and reachable from the backend runtime.
- The dashboard backend is intentionally thin; most system logic still lives in the main SmartPath services.
- If you publish this repo, review any deployment secrets and cluster-specific URLs before pushing.
