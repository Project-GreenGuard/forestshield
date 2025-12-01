# ForestShield - Quick Start Guide

## Get the Application Running Locally

### Prerequisites

- Docker & Docker Compose
- Node.js >= 16
- Python 3.11+ (optional, for local testing)

### Step 1: Start Backend Services

```bash
cd forestshield-backend
docker-compose up -d
```

This starts:

- **API Server** on http://localhost:5001
- **DynamoDB Local** on http://localhost:8000
- **Mosquitto MQTT** on ports 1883, 9001

Verify services are running:

```bash
docker-compose ps
```

### Step 2: Create Frontend Environment File

```bash
cd ../forestshield-frontend
cp .env.example .env
# Edit .env if needed (default works for local development)
```

### Step 3: Install Frontend Dependencies

```bash
npm install
```

### Step 4: Start Frontend

```bash
npm start
```

Frontend will open at http://localhost:3000

---

## Cost Information

**All services use AWS Free Tier:**

- Lambda: 1M requests/month free
- DynamoDB: On-demand pricing (cost-effective for low traffic)
- IoT Core: 250K messages/month free
- API Gateway: 1M requests/month free

**Estimated monthly cost for development:** $0-5 (well within $100 budget)

---

## Testing the Setup

### Test Backend API

```bash
# Get all sensors (will be empty initially)
curl http://localhost:5001/api/sensors

# Health check
curl http://localhost:5001/health
```

### Add Test Data (Optional)

You can manually add test data to DynamoDB Local using AWS CLI:

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

---

## Stopping Services

```bash
# Stop backend
cd forestshield-backend
docker-compose down

# Stop frontend
# Press Ctrl+C in the terminal running npm start
```

---

## Next Steps

1. **Connect Seth's Sensor**: Set up AWS IoT Core and connect the ESP32 device
2. **Deploy to AWS**: Use Terraform to deploy infrastructure (when ready)
3. **Add More Sensors**: Configure additional ESP32 devices

---

## Troubleshooting

**Backend not starting?**

- Check Docker is running: `docker ps`
- Check ports 5001, 8000, 1883 are not in use
- View logs: `docker-compose logs -f`

**Frontend can't connect to API?**

- Verify backend is running: `curl http://localhost:5001/health`
- Check `.env` file exists and has correct URL
- Check browser console for CORS errors

**No data showing?**

- Backend is running but DynamoDB is empty (normal for first run)
- Add test data using AWS CLI (see above)
- Or wait for sensor data once IoT device is connected
