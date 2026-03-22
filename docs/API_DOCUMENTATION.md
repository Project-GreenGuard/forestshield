# ForestShield — API documentation

**Last updated:** March 2026.

## Base URLs

| Environment | Base URL pattern |
|-------------|------------------|
| Local (Docker) | `http://localhost:5001/api` — map `REACT_APP_API_URL` to this in Create React App |
| AWS (API Gateway) | `https://{api-id}.execute-api.{region}.amazonaws.com/{stage}/api` — e.g. production **`…/prod/api`** |

Frontend code appends paths such as `/sensors` to the base (no duplicate `/api`).

## REST endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/sensors` | Latest row per `deviceId` (merged public fields) |
| GET | `/sensor/{id}` | Latest row for one device |
| GET | `/risk-map` | Points from last 24h for map visualization |
| GET | `/nasa-fires` | VIIRS-based hotspots (needs **`NASA_MAP_KEY`** on API Lambda). **Dashboard map** polls this and draws orange **`CircleMarker`**s (toggle “NASA FIRMS”). |
| POST | `/ingest` | **Local Flask only** — runs `process_sensor_data` with JSON body (simulates IoT) |

## Sensor JSON shape (typical)

Field names match DynamoDB / API responses:

| Field | Type | Notes |
|-------|------|--------|
| `deviceId` | string | Partition key |
| `timestamp` | string | ISO 8601, range key |
| `temperature`, `humidity` | number | |
| `lat`, `lng` | number | |
| `riskScore` | number | 0–100 scale |
| `riskLevel` | string | `LOW` \| `MEDIUM` \| `HIGH` |
| `spreadRateKmh` | number | From Cloud Run or heuristic |
| `nearestFireDistance` | number | km; **`-1`** if unknown |

**Naming:** Use **`nearestFireDistance`** (not `nearestFireKm`) end-to-end.

## Processing Lambda (not HTTP from browser)

- Triggered by **IoT Core** (and test invokes). Body is JSON with at least `deviceId`, `temperature`, `humidity`, `lat`, `lng`, optional `timestamp`.

## Environment variables (summary)

**Processing Lambda (`process_sensor_data`):**

- `DYNAMODB_TABLE` — required in AWS
- `NASA_MAP_KEY` — optional; enables FIRMS area CSV (Ontario bbox), else country CSV
- `CLOUD_RUN_PREDICT_URL` — optional; full URL to `POST /predict`
- `CLOUD_RUN_TIMEOUT_SEC` — optional; default short; increase if Cloud Run is slow

**API Lambda (`api_handler`):**

- `DYNAMODB_TABLE`
- `NASA_MAP_KEY` — for `/api/nasa-fires`

**Local Docker (`docker-compose`):** see `forestshield-backend/.env.example`.

## GitHub Actions (frontend build)

`REACT_APP_API_URL` is set from repository secret **`PRODUCTION_API_URL`** (or staging equivalent). Update the secret and **re-run deploy** after changing the API base.
