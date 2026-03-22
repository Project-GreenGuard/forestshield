# ForestShield — project overview

**Last updated:** March 2026.

## Purpose

Wildfire-risk awareness stack: IoT sensors → AWS → optional ML on **GCP Cloud Run** → DynamoDB → REST API → React dashboard (map, alerts, reports). **Evacuation** is not an active product track (no routes/zones); any existing copy-only UI is legacy/optional.

## Architecture (current)

```
ESP32 → AWS IoT Core → Lambda (process_sensor_data) → DynamoDB
                              ↑
                    optional: HTTPS POST to Cloud Run /predict
                              ↓
API Gateway → Lambda (api_handler) ← React app (REACT_APP_API_URL only)
```

- **Cloud Run** is **never** called from the browser. Only **`process_sensor_data`** calls it when **`CLOUD_RUN_PREDICT_URL`** is set.
- The dashboard uses **API Gateway** (or local Flask on port **5001**) with base path **`/api`**.

## What is implemented (production path)

| Layer | Notes |
|-------|--------|
| Ingestion | IoT rule → `process_sensor_data`; NASA FIRMS (area key + country CSV fallback); Cloud Run or rule-based risk + spread heuristic |
| Storage | DynamoDB table per env; TTL ~30 days on items |
| Read API | `GET /api/sensors`, `/api/sensor/{id}`, `/api/risk-map`, `/api/nasa-fires` |
| API behavior | **`riskLevel`** / **`spreadRateKmh`** preferred from DynamoDB; **`sensor_enrichment`** fills gaps for legacy rows |
| Risk map | Scan with **`Attr('timestamp')`** for last 24h (partition key is `deviceId`, not time) |
| Frontend | Polls sensors ~10s, risk map ~30s, NASA FIRMS ~3m; map toggle **NASA FIRMS** + orange hotspots from **`GET /api/nasa-fires`**; Alerts “Last updated” = **sensor reading time**, not poll time |

## Explicit non-goals (current scope)

- **No SNS / outbound alerting** — in-app Alerts only.
- **No CloudFront** in scope for this phase.
- **No GuardDuty** unless the course mandates it.
- **No “heavy” observability** — only a **light** CloudWatch bar (alarms / basic visibility); see **`REMAINING_WORK.md`**.
- **No evacuation product work** — no official routes, zones, or map integrations for evac.
- **Not** dashboard authentication by default (Cognito only if rubric requires).

## In scope (see `REMAINING_WORK.md`)

**Lambda DLQ**, **Terraform vs deployed reality**, **reports**, **device ops** (firmware/ops owner), **monorepo hygiene**, **light monitoring**.

**NASA fires on map:** Implemented in **`forestshield-frontend`** (poll **`/api/nasa-fires`**, draw **`CircleMarker`**s). Requires **`NASA_MAP_KEY`** on the **API** Lambda for live data.

## Deferred / optional (not current focus)

Cognito, KMS hardening, WAF-only work, multi-region, advanced APM/tracing suites — only if rubric or time allows.

## Vertex AI

**Not** required for this project path. Inference is **Cloud Run** (or rule-based fallback).
