# ForestShield — production verification

**Purpose:** Repeatable checks that production matches what the **React app** and **API Lambda** expect, plus optional AWS health for the pipeline and monitoring you have configured.

**Last full run:** March 22, 2026 — **all automated checks passed** (see table below).

---

## What we validate

| Layer | Checks |
|-------|--------|
| **Public API** | `GET /sensors`, `GET /sensor/{id}`, `GET /risk-map`, `GET /nasa-fires` — HTTP 200, JSON shape (see [API_DOCUMENTATION.md](./API_DOCUMENTATION.md)). |
| **Browser integration** | `Access-Control-Allow-Origin` on **GET** `/sensors` (required for the dashboard; do not rely on **HEAD**, which API Gateway may not map). |
| **AWS (optional)** | Processing Lambda **DLQ** configured; IoT rule **error action**; at least four **`wildfire-*`** CloudWatch alarms (full stack has six: errors, throttles, p95 × two Lambdas). |
| **Local repo** | `forestshield-frontend` **`npm run build`**; `forestshield-infrastructure/aws` **`terraform validate`**. |

**Not covered here:** ESP32 hardware, Cloud Run availability (only indirectly via enriched sensor rows), SNS/email on alarms, full `terraform plan` vs remote state.

---

## Latest recorded results (March 22, 2026)

| Check | Result | Notes |
|-------|--------|--------|
| GET `/sensors` | PASS | 200; non-empty list; `deviceId`, `timestamp`, `riskScore` present |
| GET `/sensor/{id}` | PASS | 200 for live `deviceId` from `/sensors` |
| GET `/risk-map` | PASS | 200; points include `deviceId`, `lat`, `lng` |
| GET `/nasa-fires` | PASS | 200; `fires[]` with `latitude` / `longitude`; count aligned with FIRMS |
| CORS on GET `/sensors` | PASS | `Access-Control-Allow-Origin: *` |
| Lambda DLQ (`process_sensor_data`) | PASS | Target SQS `wildfire-sensor-pipeline-dlq` |
| IoT rule error action | PASS | SQS DLQ + IAM role |
| CloudWatch alarms | PASS | 6× `wildfire-*` |
| Frontend build | PASS | Production build OK |
| Terraform validate | PASS | `forestshield-infrastructure/aws` |

**Example production API base used:** `https://l2o3nlqui2.execute-api.us-east-1.amazonaws.com/prod/api` (API ID and stage may differ per account; always use your deployed stage URL.)

---

## How to re-run

### Option A — full script (recommended)

From the **monorepo / POC root** that contains `scripts/verify-production.sh` (e.g. `ai-disaster-response-system` / `forestshield-poc`):

```bash
export FORESTSHIELD_API_BASE="https://YOUR_API_ID.execute-api.us-east-1.amazonaws.com/prod/api"
export AWS_PROFILE=GreenGuard   # optional, for Lambda / IoT / CloudWatch checks
export AWS_REGION=us-east-1
./scripts/verify-production.sh
```

### Option B — lightweight poll only

Smaller smoke test (two routes only):

```bash
export FORESTSHIELD_API_BASE="https://YOUR_API_ID.execute-api.us-east-1.amazonaws.com/prod/api"
./scripts/check-prod-health.sh
```

---

## Manual UI checks (same release as API)

After deploy, in the browser (with `REACT_APP_API_URL` pointing at the same `/api` base):

1. **Map:** sensors + risk visualization; **NASA FIRMS** toggle and hotspots.
2. **Reports:** summary stats; **Export CSV** downloads a snapshot.
3. **Alerts / data panel:** `riskLevel`, `spreadRateKmh`, last reading time.

---

## If something fails

| Symptom | Where to look |
|---------|----------------|
| 403 on HEAD / curl oddities | Use **GET** for API Gateway; confirm path is `.../prod/api/sensors` not missing `/api`. |
| `/nasa-fires` empty or errors | **`NASA_MAP_KEY`** on **API Lambda**; CloudWatch logs for `wildfire-api-handler`. |
| Empty `/sensors` | IoT / processing Lambda / DynamoDB ingest; DLQ for poison payloads. |
| CORS errors in browser | API Lambda must return `Access-Control-Allow-Origin` (already in handler). |

---

_Update this file’s **Last full run** line and the results table whenever you run `./scripts/verify-production.sh` for a demo or release. After material infra or API changes, re-run the script and align **API_DOCUMENTATION.md** and **ARCHITECTURE.md** if needed._
