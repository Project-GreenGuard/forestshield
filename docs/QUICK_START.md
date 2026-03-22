# ForestShield — quick start

Run the stack locally and find pointers for Lambdas, Terraform, and devices.

## Prerequisites

- Docker & Docker Compose  
- Node.js ≥ 16  
- Python 3.11+ (optional: mock sensor, Lambda scripts)

## Run locally

### 1. Backend (Docker)

```bash
cd forestshield-backend
docker-compose up -d
docker-compose ps
```

On the host, the API is exposed as **`http://localhost:5001`** (maps to port 5000 in the container). Use **`http://localhost:5001/api`** as the frontend base (see below).

Services typically include: API, DynamoDB Local (~8000), Mosquitto MQTT (1883).

### 2. Frontend

```bash
cd forestshield-frontend
cp .env.example .env   # or: echo "REACT_APP_API_URL=http://localhost:5001/api" > .env
npm install
npm start
```

App: **http://localhost:3000**

### 3. Optional — mock sensor (no ESP32)

From **`forestshield-iot-firmware`** (or your MQTT test client), publish JSON to the topic / broker your setup expects. For a local Mosquitto path, see that repo’s README / `mock_sensor.py` if present.

## Verify the API

```bash
curl -sS "http://localhost:5001/api/sensors"
curl -sS "http://localhost:5001/api/risk-map"
```

Use **`GET /api/sensors`** as the primary health check (there may be no dedicated `/health` route).

## Optional test row (DynamoDB Local)

```bash
aws dynamodb put-item \
  --endpoint-url http://localhost:8000 \
  --table-name WildfireSensorData \
  --item '{
    "deviceId": {"S": "esp32-test-01"},
    "timestamp": {"S": "2025-12-01T16:20:00Z"},
    "temperature": {"N": "25.5"},
    "humidity": {"N": "45.2"},
    "lat": {"N": "43.467"},
    "lng": {"N": "-79.699"},
    "riskScore": {"N": "45.2"},
    "nearestFireDistance": {"N": "12.5"},
    "ttl": {"N": "1733090400"}
  }'
```

## Developer notes

### Lambda zip (deploy via Terraform)

From **`forestshield-backend`**:

```bash
cd lambda-processing
zip -r ../lambda-processing.zip . -x "*.pyc" "*__pycache__*"

cd ../api-gateway-lambda
zip -r ../api-gateway-lambda.zip . -x "*.pyc" "*__pycache__*"
```

Copy the zips into **`forestshield-infrastructure/aws/`** if your Terraform expects them there, then:

```bash
cd forestshield-infrastructure/aws
terraform init
terraform plan
# terraform apply   # when you intend to change AWS
```

### Key frontend files

- `src/App.js` — routing / data polling  
- `src/components/MapArea.js` — map, sensors, NASA FIRMS layer  
- `src/services/api.js` — API base URL  

### Key backend files

- `lambda-processing/process_sensor_data.py` — IoT → DynamoDB enrichment  
- `api-gateway-lambda/api_handler.py` — REST API  

### Production checks

See **[PRODUCTION_VERIFICATION.md](./PRODUCTION_VERIFICATION.md)** and the monorepo scripts **`scripts/verify-production.sh`** / **`scripts/check-prod-health.sh`** (set `FORESTSHIELD_API_BASE` to your `…/prod/api` URL).

## Troubleshooting

| Issue | What to check |
|-------|----------------|
| Backend won’t start | `docker ps`, ports **5001 / 8000 / 1883**, `docker-compose logs` |
| Frontend empty / errors | `.env` → `REACT_APP_API_URL=http://localhost:5001/api`, browser console |
| No map data | DynamoDB empty until IoT or test `put-item` above |

## Next steps

- **[API_DOCUMENTATION.md](./API_DOCUMENTATION.md)** — all routes and env vars  
- **`forestshield-iot-firmware/docs/DEVICE_OPS.md`** — provision ESP32 → AWS IoT  
- **[PROJECT_OVERVIEW.md](./PROJECT_OVERVIEW.md)** — architecture and scope  

---

**Last updated:** March 2026
