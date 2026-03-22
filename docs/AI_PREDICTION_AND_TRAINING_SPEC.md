# AI prediction — integration spec (Cloud Run)

**Last updated:** March 2026.

## Role

**`process_sensor_data`** (AWS Lambda) may call a **GCP Cloud Run** HTTPS endpoint to obtain ML-derived **risk** and **spread**. If the call fails or **`CLOUD_RUN_PREDICT_URL`** is unset, the handler uses **rule-based** scoring and a local spread heuristic. Optional **`CLOUD_RUN_TIMEOUT_SEC`** controls the HTTP client timeout (see **`API_DOCUMENTATION.md`**).

## Request (POST)

`Content-Type: application/json`

```json
{
  "temperature": 37.5,
  "humidity": 22.0,
  "lat": 43.55,
  "lng": -79.65,
  "nearestFireDistance": 12.3,
  "timestamp": "2026-03-22T19:13:23Z"
}
```

- **`nearestFireDistance`:** use a numeric km value when known; Lambda may send **`100.0`** when missing (implementation detail).

## Response (200 JSON)

The Lambda accepts **either** snake_case **or** camelCase:

| Logical field | Accepted keys |
|---------------|----------------|
| Score | `risk_score` or `riskScore` |
| Level | `risk_level` or `riskLevel` — `LOW` / `MEDIUM` / `HIGH` (normalized) |
| Spread km/h | `spread_rate` or `spreadRateKmh` |

If level is missing or invalid, Lambda derives level from score thresholds.

## Persistence

Mapped into DynamoDB as **`riskScore`**, **`riskLevel`**, **`spreadRateKmh`** (Decimals). The **API Lambda** reads these fields and merges with **`sensor_enrichment`** for legacy items.

## Training / model lifecycle

Owned by the **Cloud Run service** repository/image (e.g. `forestshield-ai`). This doc does not prescribe framework; only the **HTTP JSON contract** above must stay stable for Lambda compatibility.
