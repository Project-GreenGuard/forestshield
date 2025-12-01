# ForestShield - Development Setup Guide

This guide covers local development setup for running the ForestShield applications on your machine.

## Prerequisites

### Required Software

- **Node.js**: >= 16.x (for frontend development)
- **Python**: >= 3.9 (for backend Lambda functions)
- **Docker & Docker Compose**: >= 2.0 (for local backend services)
- **Git**: For version control

## Local Development Setup

### 1. Backend Development

The backend uses Docker Compose to run local services including DynamoDB Local and Mosquitto MQTT broker.

#### Start Local Services

```bash
cd forestshield-backend

# Start all services (API server, DynamoDB Local, Mosquitto)
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down
```

#### Available Services

- **API Server**: http://localhost:5000
- **DynamoDB Local**: http://localhost:8000
- **MQTT Broker**: mqtt://localhost:1883

#### Test API Endpoints

```bash
# Get all sensors
curl http://localhost:5000/api/sensors

# Get specific sensor
curl http://localhost:5000/api/sensor/esp32-01

# Get risk map data
curl http://localhost:5000/api/risk-map
```

#### Local Development Environment

The local API server (`local_dashboard.py`) provides the same endpoints as the production API Gateway, but connects to DynamoDB Local instead of AWS DynamoDB.

**Environment Variables:**

- `AWS_ENDPOINT_URL=http://dynamodb:8000` - DynamoDB Local endpoint
- `AWS_ACCESS_KEY_ID=local`
- `AWS_SECRET_ACCESS_KEY=local`

### 2. Frontend Development

The frontend is a React Progressive Web Application that can run independently with mock data or connect to the local backend.

#### Setup and Run

```bash
cd forestshield-frontend

# Install dependencies (first time only)
npm install

# Create .env file for local development
echo "REACT_APP_API_URL=http://localhost:5000/api" > .env

# Start development server
npm start
```

The frontend will automatically open at: **http://localhost:3000**

#### Frontend Features

- Real-time sensor data visualization
- Interactive map with Leaflet
- Risk heatmap overlay (when connected to backend)
- Responsive design

#### Development Notes

- The frontend uses mock data by default if the backend is not available
- Hot reload is enabled - changes automatically refresh in the browser
- API polling interval: 10 seconds for sensor data, 30 seconds for risk map

### 3. IoT Firmware Development

For testing without physical hardware, use the mock sensor script.

#### Mock Sensor (Python)

```bash
cd forestshield-iot-firmware

# Run mock sensor (publishes to local MQTT broker)
python mock_sensor.py
```

The mock sensor simulates ESP32 sensor data and publishes to the local Mosquitto MQTT broker.

#### ESP32 Firmware (Arduino)

For actual hardware development:

1. Open `esp32_wildfire_sensor.ino` in Arduino IDE
2. Install required libraries:
   - ArduinoJson
   - DHT sensor library (Adafruit)
   - Adafruit Unified Sensor
   - PubSubClient
3. Configure WiFi credentials and device settings
4. Upload to ESP32 device

**Note:** For AWS IoT Core integration, see the infrastructure repository documentation.

## Development Workflow

### Typical Development Session

1. **Start Backend Services:**

   ```bash
   cd forestshield-backend
   docker-compose up -d
   ```

2. **Start Frontend:**

   ```bash
   cd forestshield-frontend
   npm start
   ```

3. **Optional - Run Mock Sensor:**

   ```bash
   cd forestshield-iot-firmware
   python mock_sensor.py
   ```

4. **Access Applications:**
   - Frontend: http://localhost:3000
   - Backend API: http://localhost:5000/api

### Testing Changes

- **Frontend:** Changes automatically reload via React hot reload
- **Backend:** Restart Docker containers to apply changes:
  ```bash
  docker-compose restart
  ```

## Troubleshooting

### Backend Issues

**Docker services won't start:**

- Ensure Docker Desktop is running
- Check if ports 5000, 8000, 1883 are already in use
- Review logs: `docker-compose logs`

**API endpoints not responding:**

- Verify services are running: `docker-compose ps`
- Check API server logs: `docker-compose logs api`
- Ensure DynamoDB Local is accessible

### Frontend Issues

**Dependencies installation fails:**

- Clear node_modules and reinstall: `rm -rf node_modules && npm install`
- Check Node.js version: `node --version` (should be >= 16)

**API connection errors:**

- Verify backend is running on port 5000
- Check `.env` file has correct API URL
- Review browser console for CORS errors

**Port 3000 already in use:**

- Stop other React apps or change port: `PORT=3001 npm start`

### Mock Sensor Issues

**Python dependencies missing:**

- Install required packages: `pip install paho-mqtt`

**MQTT connection fails:**

- Ensure Mosquitto is running: `docker-compose ps`
- Check MQTT broker is accessible on port 1883

## Project Structure

```
forestshield-backend/
├── api-gateway-lambda/     # API handler Lambda function
├── lambda-processing/      # Sensor data processing Lambda
├── docker-compose.yml      # Local services configuration
└── local_dashboard.py     # Local API server

forestshield-frontend/
├── src/
│   ├── components/         # React components
│   ├── App.js             # Main app component
│   └── index.js           # Entry point
├── public/                # Static assets
└── package.json          # Dependencies

forestshield-iot-firmware/
├── esp32_wildfire_sensor.ino  # ESP32 firmware
└── mock_sensor.py            # Mock sensor for testing
```

## Next Steps

- Review [DEVELOPMENT_GUIDE.md](./DEVELOPMENT_GUIDE.md) for detailed development workflows
- Check [API_DOCUMENTATION.md](./API_DOCUMENTATION.md) for API endpoint details
- See [ARCHITECTURE.md](./ARCHITECTURE.md) for system architecture overview

---

**Setup Version**: 1.0  
**Last Updated**: Current Semester
