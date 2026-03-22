# ForestShield - System Architecture

## High-Level Architecture

```
┌─────────────────┐
│   ESP32 + DHT11 │  (IoT Sensor)
│   (Hardware)    │
└────────┬────────┘
         │ MQTT over TLS
         │ Topic: wildfire/sensors/{deviceId}
         ▼
┌─────────────────┐
│  AWS IoT Core   │  (Device Management)
│  - Thing        │
│  - Certificate  │
│  - Policy       │
│  - Rule         │
└────────┬────────┘
         │ Triggers on: wildfire/sensors/+
         ▼
┌─────────────────┐
│  Lambda Function│  (process_sensor_data)
│  - Parse payload│
│  - Fetch FIRMS  │
│  - Calculate    │
│    risk score   │
└────────┬────────┘
         │
         ├──→ NASA FIRMS API (External)
         │
         ▼
┌─────────────────┐
│   DynamoDB      │  (Data Storage)
│   Table:        │
│   WildfireSensor│
│   Data          │
│   - deviceId (PK)│
│   - timestamp (SK)│
│   - TTL: 30 days│
└────────┬────────┘
         │
         │ Queried by
         ▼
┌─────────────────┐
│  Lambda Function│  (api_handler)
│  - GET /sensors │
│  - GET /sensor/ │
│    {id}         │
│  - GET /risk-map│
└────────┬────────┘
         │
         │ REST API
         ▼
┌─────────────────┐
│  API Gateway    │  (REST API)
│  - /api/sensors │
│  - /api/sensor/ │
│    {id}         │
│  - /api/risk-map│
└────────┬────────┘
         │ HTTP/HTTPS
         │ CORS enabled
         ▼
┌─────────────────┐
│  React PWA      │  (Frontend)
│  - Map (Leaflet)│
│  - Sensor data  │
│  - Risk heatmap │
└─────────────────┘
```

## Component Details

### 1. IoT Sensor (ESP32)

**Hardware:**
- ESP32 Dev Module
- DHT11 sensor (temperature, humidity)
- WiFi connectivity

**Firmware Responsibilities:**
- Read sensor data every 30 seconds
- Format JSON payload
- Connect to AWS IoT Core via MQTT
- Publish to topic: `wildfire/sensors/{deviceId}`

**Payload Format:**
```json
{
  "deviceId": "esp32-01",
  "temperature": 23.4,
  "humidity": 40.2,
  "lat": 43.467,
  "lng": -79.699,
  "timestamp": "2025-12-01T16:20:00Z"
}
```

### 2. AWS IoT Core

**Components:**
- **IoT Thing**: Device registration
- **Certificate**: Device authentication
- **Policy**: Device permissions (connect, publish, subscribe)
- **IoT Rule**: Triggers Lambda on `wildfire/sensors/+` topic

**Configuration:**
- Endpoint: Regional (e.g., `xxxxx.iot.us-east-1.amazonaws.com`)
- Protocol: MQTT over TLS (port 8883)
- Security: X.509 certificates

### 3. Lambda: Process Sensor Data

**Function**: `wildfire-process-sensor-data`

**Responsibilities:**
1. Receive IoT payload from IoT Core
2. Parse sensor data (temperature, humidity, GPS)
3. Fetch NASA FIRMS wildfire data (HTTP GET)
4. Find nearest fire to sensor location
5. Calculate risk score
6. Store enriched data in DynamoDB

**Input:**
```json
{
  "deviceId": "esp32-01",
  "temperature": 23.4,
  "humidity": 40.2,
  "lat": 43.467,
  "lng": -79.699,
  "timestamp": "2025-12-01T16:20:00Z"
}
```

**Output (DynamoDB Item):**
```json
{
  "deviceId": "esp32-01",
  "timestamp": "2025-12-01T16:20:00Z",
  "temperature": 23.4,
  "humidity": 40.2,
  "lat": 43.467,
  "lng": -79.699,
  "riskScore": 45.2,
  "nearestFireDistance": 12.5,
  "nearestFireData": "{...}",
  "ttl": 1733097600
}
```

### 4. DynamoDB

**Table**: `WildfireSensorData`

**Schema:**
- **Partition Key**: `deviceId` (String)
- **Sort Key**: `timestamp` (String, ISO 8601)
- **TTL**: `ttl` (Number, Unix timestamp)

**Attributes:**
- `temperature` (Number)
- `humidity` (Number)
- `lat` (Number)
- `lng` (Number)
- `riskScore` (Number)
- `nearestFireDistance` (Number)
- `nearestFireData` (String, JSON)

**Billing**: On-demand (pay per request)

### 5. Lambda: API Handler

**Function**: `wildfire-api-handler`

**Endpoints:**

**GET /api/sensors**
- Returns: List of all sensors with latest data
- Response: Array of sensor objects

**GET /api/sensor/{id}**
- Returns: Latest data for specific sensor
- Response: Single sensor object

**GET /api/risk-map**
- Returns: Risk map data (last 24 hours)
- Response: Array of data points with coordinates and risk scores

### 6. API Gateway

**Type**: REST API (Regional)

**Configuration:**
- CORS enabled
- Integration: Lambda Proxy
- Stage: `prod`

**Endpoints:**
- `GET /api/sensors`
- `GET /api/sensor/{id}`
- `GET /api/risk-map`

### 7. React Frontend

**Framework**: React 19 (PWA)

**Components:**
- `MapArea`: Leaflet map with sensor markers
- `DataPanel`: Sensor data display
- `Sidebar`: Navigation
- `Topbar`: Header

**Features:**
- Real-time data updates
- Interactive map
- Risk heatmap overlay (planned)
- Responsive design

## Data Flow Sequence

1. **ESP32** reads sensor data
2. **ESP32** publishes MQTT message to AWS IoT Core
3. **IoT Rule** triggers Lambda function
4. **Lambda** fetches NASA FIRMS data
5. **Lambda** calculates risk score
6. **Lambda** writes to DynamoDB
7. **Frontend** requests data via API Gateway
8. **API Gateway** invokes API handler Lambda
9. **Lambda** queries DynamoDB
10. **Frontend** displays data on map

## Security Architecture

### Current (This Semester)
- AWS IoT Core: X.509 certificates for device authentication
- IAM roles: Minimal permissions for Lambda functions
- API Gateway: CORS configuration

### Future (Next Semester)
- WAF (Web Application Firewall)
- GuardDuty (threat detection)
- KMS (key management)
- Cognito (user authentication)

## Scalability Considerations

- **DynamoDB**: On-demand scaling
- **Lambda**: Auto-scaling (concurrent executions)
- **IoT Core**: Handles millions of devices
- **API Gateway**: Handles high request volumes

## Cost Optimization

- DynamoDB: On-demand pricing (no provisioned capacity)
- Lambda: Free tier (1M requests/month)
- IoT Core: First 250K messages/month free
- API Gateway: First 1M requests/month free

## Monitoring & Logging

- **CloudWatch Logs**: Lambda function logs
- **CloudWatch Metrics**: Lambda invocations, errors
- **DynamoDB Metrics**: Read/write capacity, throttles
- **IoT Core Metrics**: Messages published, rules triggered

---

**Architecture Version**: 1.0  
**Last Updated**: Current Semester

