# ForestShield — how to demonstrate your work

Use this for a **live demo**, a **recorded video**, or a **marker walkthrough**. Each block maps to something you personally shipped or integrated.

---

## Before you start (2 minutes)

| Check | Why |
|-------|-----|
| Production dashboard loads with **`REACT_APP_API_URL`** = your real **`…/prod/api`** | Proves deployed frontend talks to real API Gateway |
| Know your **API base** (e.g. `https://{id}.execute-api.us-east-1.amazonaws.com/prod/api`) | You will use it in the browser and in `curl` |
| Optional: Terminal with **`FORESTSHIELD_API_BASE`** set and **`./scripts/verify-production.sh`** (from monorepo) | Opens with “we automated prod checks” |
| Optional: AWS Console logged in (IoT, Lambda, SQS, CloudWatch) | For “under the hood” 30 seconds |

If the ESP32 is not available, say clearly: **“Pipeline is the same; today I’m showing live cloud + API + UI; device was validated earlier.”**

---

## Suggested flow (~8–12 minutes)

Tell a **single story**: *sensor reading → cloud processing → storage → API → dashboard + external fire data + reliability.*

### 1. One-sentence positioning (15 s)

> “ForestShield ingests wildfire-relevant sensor data through AWS, enriches it with NASA FIRMS and optional ML on Cloud Run, stores it in DynamoDB, and serves a React dashboard over API Gateway.”

Point at **`ARCHITECTURE.md`** or **`PROJECT_OVERVIEW.md`** §4 / §18 if slides are allowed.

### 2. End-to-end data path (2–3 min) — **your core “functional area”**

**UI**

- Open **Map**: show **sensor marker(s)**, **risk** colouring, **risk circles** from recent data.
- Toggle **NASA FIRMS**: orange hotspots → *“These are VIIRS thermal anomalies, not legal fire polygons.”*
- Open **Data panel** / **Alerts**: call out **`riskLevel`**, **`spreadRateKmh`**, **timestamp** = last **reading**, not “last page load.”

**API (proves backend is yours)**

- In a terminal (replace URL):

```bash
curl -sS "https://YOUR_API_ID.execute-api.us-east-1.amazonaws.com/prod/api/sensors" | head -c 800
curl -sS "https://YOUR_API_ID.execute-api.us-east-1.amazonaws.com/prod/api/nasa-fires" | head -c 400
```

Say: *“Same JSON the UI uses; no mock.”*

### 3. ML / scoring (1–2 min) — **functional area: AI + rules**

> “Processing Lambda calls **Cloud Run** with a JSON payload when **`CLOUD_RUN_PREDICT_URL`** is set; if it fails or is unset, **rule-based** scoring still runs and we still persist **`riskLevel`** and **`spreadRateKmh`**.”

Show **`AI_PREDICTION_AND_TRAINING_SPEC.md`** (contract) and optionally **`MODEL_TRAINING.md`** (how the offline model was trained in **`forestshield-ai`**).

### 4. Reliability and ops (1–2 min) — **DLQ + monitoring**

**Narration**

> “Failed processing doesn’t silently disappear: the processing Lambda has a **dead-letter queue**, and the IoT rule can send errors to the **same SQS queue** so we can inspect bad payloads.”

**Console (optional)**

- **SQS** → `wildfire-sensor-pipeline-dlq` (message count often **0** in demo — that’s good).
- **CloudWatch** → Alarms → filter **`wildfire-`** → **errors / throttles / duration p95** on **both** Lambdas.

> “This is intentional **light** observability for a capstone, not full APM.”

### 5. Reports (30 s)

- **Reports** page → summary stats → **Export CSV** → open file: *“Snapshot export for markers or incident review.”*

### 6. Infrastructure and repos (1 min)

- “Infra is **Terraform** in **`forestshield-infrastructure`**; Lambdas and API live in **`forestshield-backend`**; UI in **`forestshield-frontend`**; device firmware in **`forestshield-iot-firmware`** (`DEVICE_OPS.md` for provisioning).”
- If asked about **GCP**: “**Cloud Run** hosts **`/predict`**; AWS hosts the rest.”

### 7. Documentation + VP (30 s)

- **`DOCUMENTATION_INDEX.md`** → list: API, architecture, prod verification, VP model, team setup.
- **`VP_SOFTWARE_MODEL.md`** → *“Formal requirements; the as-built addendum at the top ties it to what we shipped.”*

---

## Map every “functional area” to evidence

| Area | What to show |
|------|----------------|
| **IoT ingestion** | Live MQTT path: device **or** explain rule **`wildfire/sensors/+`** → Lambda; **`TEAM_SETUP_GUIDE`** / firmware **`DEVICE_OPS`** |
| **External data (NASA)** | Map toggle + **`/api/nasa-fires`** `curl` + note **`NASA_MAP_KEY`** on Lambda |
| **Processing / risk** | DynamoDB fields on screen + Cloud Run spec doc + rule fallback sentence |
| **Storage** | `deviceId` + `timestamp` keys; TTL mention from **ARCHITECTURE** |
| **API** | Three or four **GET** routes in **API_DOCUMENTATION** + live `curl` |
| **Dashboard** | Map, alerts, reports, CSV |
| **Reliability** | DLQ queue + CloudWatch **wildfire-** alarms |
| **IaC** | Terraform repo + “plan before apply” line |
| **Docs / governance** | Index + VP + **PRODUCTION_VERIFICATION** |

---

## If something breaks during the demo

| Problem | Fallback |
|---------|-----------|
| Map empty | Show **`curl /api/sensors`** still 200; explain cold device or empty table; show **DynamoDB** or yesterday’s screenshot |
| NASA layer empty | **0 fires** can be valid; show **`/api/nasa-fires`** JSON and **NASA_MAP_KEY** on API Lambda in console |
| Cloud Run down | Say **rule-based path** is production-safe; show **PROJECT_OVERVIEW** §18 |

---

## After the demo

- Run **`verify-production.sh`** once and update the date table in **`PRODUCTION_VERIFICATION.md`** if a formal submission asks for “evidence of working prod.”

---

**Last updated:** March 2026
