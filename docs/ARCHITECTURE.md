# ForestShield — system architecture

**Last updated:** March 2026 (aligned with shipped AWS + React app).

## High-level architecture

```
┌─────────────────┐
│   ESP32 + DHT11 │  (IoT sensor)
└────────┬────────┘
         │ MQTT TLS :8883  topic wildfire/sensors/{deviceId}
         ▼
┌─────────────────┐
│  AWS IoT Core   │  Thing, cert, policy, topic rule
│  Rule → Lambda  │  Optional error action → SQS DLQ
└────────┬────────┘
         │
         ▼
┌─────────────────┐     POST JSON (optional)
│  Lambda         │────► GCP Cloud Run /predict  (CLOUD_RUN_PREDICT_URL)
│ process_sensor  │      → riskScore, riskLevel, spreadRateKmh
│ _data           │
└────────┬────────┘
         │ NASA FIRMS (HTTP) for proximity / enrichment
         │ Async failures → SQS DLQ (same queue as IoT rule errors)
         ▼
┌─────────────────┐
│   DynamoDB      │  WildfireSensorData (PK deviceId, SK timestamp, TTL on ttl)
│   + riskLevel   │  nearestFireDistance, spreadRateKmh, …
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Lambda         │  GET /api/sensors, /sensor/{id}, /risk-map, /nasa-fires
│  api_handler    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  API Gateway    │  Regional REST, stage e.g. prod, CORS on responses
│  /api/*         │
└────────┬────────┘
         │ HTTPS
         ▼
┌─────────────────┐
│  React app      │  Map (sensors + risk circles + NASA FIRMS toggle),
│  (CRA / PWA)    │  Alerts, Reports (CSV export), DataPanel, …
└─────────────────┘
```

## Component summary

### IoT sensor (ESP32)

- Publishes JSON: `deviceId`, `temperature`, `humidity`, `lat`, `lng`, `timestamp` (see firmware repo).
- Typical interval: **30 s** (configurable in sketch).

### AWS IoT Core

- Rule SQL: `SELECT * FROM 'wildfire/sensors/+'` → invoke **`wildfire-process-sensor-data`**.
- **Error action (shipped):** failed rule deliveries can be sent to **SQS** (pipeline DLQ) for inspection.

### Lambda: `wildfire-process-sensor-data`

- Enriches readings, FIRMS distance, **rule-based and/or Cloud Run** scoring.
- Env: `DYNAMODB_TABLE`, `NASA_MAP_KEY`, `CLOUD_RUN_PREDICT_URL`, `CLOUD_RUN_TIMEOUT_SEC` (see API doc).
- **Dead-letter queue:** failed async invokes → **SQS** (same operational pattern as IoT errors).

### DynamoDB

- Table name environment-specific (e.g. **`WildfireSensorData`** in prod).
- Important attributes: `riskScore`, `riskLevel`, `spreadRateKmh`, `nearestFireDistance`, `temperature`, `humidity`, `lat`, `lng`, `ttl`.

### Lambda: `wildfire-api-handler`

| Route | Role |
|--------|------|
| `GET /api/sensors` | Latest row per device (merged public fields) |
| `GET /api/sensor/{id}` | Latest row for one device |
| `GET /api/risk-map` | Scan last 24 h (filter on `timestamp`) for map |
| `GET /api/nasa-fires` | VIIRS hotspots for dashboard map (**`NASA_MAP_KEY`** on this Lambda) |

### API Gateway

- Lambda proxy integration; base URL ends with **`/{stage}/api`**.

### React frontend

- **React 19**, Leaflet / react-leaflet; polls sensors / risk map / NASA layer per `App.js`.
- **NASA FIRMS:** checkbox + orange `CircleMarker`s from `/api/nasa-fires`.

## Data flow (happy path)

1. Device publishes MQTT → IoT rule → **processing** Lambda.  
2. Lambda optionally calls **Cloud Run**, writes **DynamoDB**.  
3. Browser calls **API Gateway** → **api_handler** → DynamoDB.  
4. UI updates map, panels, reports.

## Security (summary)

- Devices: **X.509** + IoT policy.  
- Lambdas: **IAM** least-privilege; execution roles include DynamoDB and (where configured) SQS DLQ send.  
- API: **CORS** `Access-Control-Allow-Origin: *` on JSON responses (verify with **GET**, not necessarily HEAD).

**Heavy add-ons** (WAF, GuardDuty, Cognito, etc.) remain **optional** for capstone scope unless required by the course.

## Observability (shipped, light)

- **CloudWatch Logs** for both Lambdas.  
- **CloudWatch alarms** (example naming): `wildfire-process-sensor-errors`, `wildfire-api-handler-errors`, throttles, **p95 duration** — see **`PRODUCTION_VERIFICATION.md`**.  
- **SQS DLQ** for poison messages / failed rule actions.

## Cost and scale

- On-demand DynamoDB, Lambda, API Gateway, IoT: suitable for demo/low traffic; see also **PROJECT_OVERVIEW** budget notes.

## Related documents

- **`API_DOCUMENTATION.md`** — exact URLs, env vars, sensor JSON.  
- **`AI_PREDICTION_AND_TRAINING_SPEC.md`** — Cloud Run JSON contract.  
- **`PROJECT_OVERVIEW.md`** — scope, timeline, §18 as-built addendum.  
- **`PRODUCTION_VERIFICATION.md`** — how to re-run smoke tests.

---

**Document version:** 1.1  
