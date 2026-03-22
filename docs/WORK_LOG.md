# ForestShield — work log (for demos / ownership narrative)

**Purpose:** Short, honest list of engineering work you can speak to in standups, reports, or a **demonstration video**. Update this file when you ship something meaningful.

**Last updated:** March 22, 2026.

---

## Shipped — DLQ, monitoring, reports export, device ops, hygiene (March 22, 2026)

- **Production (AWS CLI):** **`wildfire-process-sensor-data`** async **DLQ** → SQS **`wildfire-sensor-pipeline-dlq`**; IAM **`WildfireLambdaDLQSend`** on **`wildfire-lambda-role`**; IoT rule **`wildfire_sensor_data_rule`** **error action** → same queue via role **`wildfire-iot-rule-error-sqs`**.
- **CloudWatch alarms (prod):** errors + throttles + **p95 duration** (25s threshold) for **`wildfire-process-sensor-data`** and **`wildfire-api-handler`** (names prefixed **`wildfire-`**).
- **Terraform:** Extended **`dlq_and_monitoring.tf`** with API throttles + duration p95 alarms to match the above; run **`terraform plan`** before apply — live resources created via CLI may need **import** or a one-time reconcile.
- **Frontend:** **Reports** page **Export CSV** (current sensor snapshot columns).
- **Firmware:** **`docs/DEVICE_OPS.md`** (provisioning, buffers, ops); MQTT **`setBufferSize(512)`** for JSON headroom.
- **Monorepo:** **`scripts/check-prod-health.sh`** polls **`/sensors`** and **`/nasa-fires`**; **`scripts/verify-production.sh`** runs a **full** prod + repo check (API schema, CORS on GET, optional AWS, `npm run build`, **`terraform validate`**). Documented in **`docs/PRODUCTION_VERIFICATION.md`**; root **`README.md`** updated.

---

## Session / milestone summary (March 2026)

### Backend & data pipeline (`forestshield-backend` → `main`)

- **Aligned processing Lambda** with production behavior: **`CLOUD_RUN_PREDICT_URL`** + **`CLOUD_RUN_TIMEOUT_SEC`**, NASA FIRMS **area API** with key plus **country CSV fallback**, rule-based fallback if Cloud Run fails.
- **Persisted** **`riskLevel`** and **`spreadRateKmh`** to DynamoDB with the rest of the sensor record.
- **API Lambda:** returns those fields; **`sensor_enrichment.merge_sensor_public_fields`** prefers DynamoDB values and backfills older rows from score/distance heuristics.
- **Risk map:** fixed DynamoDB scan filter to use **`Attr('timestamp')`** (correct for a non-key attribute).
- **Routes:** **`GET /api/nasa-fires`**, local **`POST /api/ingest`** for pipeline testing.
- **Hygiene:** removed committed secrets from compose patterns; **slim** Lambda deps (no unnecessary sklearn stack in the shipping path); dropped dev shell scripts and local pytest from the repo when trimming to “prod-merge” shape.
- **Git / review:** teammate **PR #9** reviewed and **closed without merge**; authoritative line is **`Project-GreenGuard/forestshield-backend` `main`** (your push).

### AWS operations (console / CLI — not all in git)

- **Removed duplicate** API Gateway REST APIs; kept **one** prod API + staging as appropriate.
- **Rotated / set** **`NASA_MAP_KEY`** on the processing Lambda; verified **CloudWatch** (no FIRMS area failure warning; `0` fire rows is normal when there are no hotspots in the bbox).
- **End-to-end prod check:** Lambda invoke → DynamoDB → **`GET …/api/sensors`** returns enriched JSON.

### Infrastructure as code

- **Deduplicated** duplicate Terraform **`resource`** blocks in **`forestshield-infrastructure/aws/`** (`dynamodb`, `iam`, `frontend`, `iot-core`) so **`terraform validate`** passes — **still run `terraform plan`** before apply to reconcile with live AWS.

### Frontend & CI

- Documented / set **`PRODUCTION_API_URL`** (GitHub secret) to the prod API base (**`…/prod/api`**) so **build-time** **`REACT_APP_API_URL`** matches API Gateway.
- **NASA FIRMS on map (this branch / change):** **`getNasaFires()`** → **`GET /api/nasa-fires`**, map shows up to **400** orange **`CircleMarker`**s, **toggle** “NASA FIRMS”, popups with date/confidence/FRP, **3-minute** refresh.

### Documentation & scope

- Added **`forestshield/docs/`** (overview, API, AI contract, index).
- Rewrote **`REMAINING_WORK.md`** to match **descoped** items (no SNS, no CloudFront, no heavy GuardDuty push, no evacuation product work) and **in-scope** backlog (DLQ, Terraform drift, light monitoring, reports, device ops, monorepo hygiene).
- Expanded **root `README.md`** to point at docs and repos.

---

## What you still own (in-scope — say this in the video roadmap)

| Area | Talking point |
|------|----------------|
| **Lambda DLQ** | “Failed invokes and rule errors land in **SQS** — we can inspect payloads.” |
| **Terraform vs reality** | “`terraform plan` before apply; import or align anything created in the console/CLI first.” |
| **Light monitoring** | “CloudWatch **errors / throttles / p95 duration** on both Lambdas — no enterprise APM.” |
| **Reports** | “**CSV export** of the live sensor snapshot from the Reports page.” |
| **Device ops** | “**DEVICE_OPS.md** + larger MQTT buffer; provisioning checklist in-repo.” |
| **Monorepo hygiene** | “Health script + README pointers + this work log.” |

---

## Demo video checklist (suggested order)

1. **Map:** sensors + risk circles + **NASA FIRMS** toggle (explain VIIRS = thermal anomalies, not legal fire lines).
2. **Alerts / data panel:** `riskLevel`, `spreadRateKmh`, “Last updated” = last **reading**, not last poll.
3. **API:** browser or `curl` **`/api/sensors`** and optionally **`/api/nasa-fires`**.
4. **Architecture one slide:** IoT → processing Lambda → (optional Cloud Run) → DynamoDB → API Gateway → React (no Cloud Run in browser).
5. **What you personally shipped:** point at this **WORK_LOG** + commits on **`main`** / feature branch.

---

_Use dated sections above for new milestones._
