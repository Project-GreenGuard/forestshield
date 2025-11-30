# ForestShield - API Documentation

## Base URL

**Local Development:**
```
http://localhost:5000/api
```

**Production (API Gateway):**
```
https://YOUR_API_GATEWAY_URL/api
```

## Endpoints

### 1. Get All Sensors

**GET** `/api/sensors`

Returns a list of all sensors with their latest data.

**Response:**
```json
[
  {
    "deviceId": "esp32-01",
    "temperature": 23.4,
    "humidity": 40.2,
    "lat": 43.467,
    "lng": -79.699,
    "riskScore": 45.2,
    "timestamp": "2025-12-01T16:20:00Z"
  },
  {
    "deviceId": "esp32-02",
    "temperature": 25.1,
    "humidity": 38.5,
    "lat": 43.470,
    "lng": -79.702,
    "riskScore": 52.3,
    "timestamp": "2025-12-01T16:21:00Z"
  }
]
```

**Status Codes:**
- `200 OK` - Success
- `500 Internal Server Error` - Server error

---

### 2. Get Sensor by ID

**GET** `/api/sensor/{id}`

Returns the latest data for a specific sensor.

**Parameters:**
- `id` (path parameter) - Device ID (e.g., `esp32-01`)

**Example:**
```
GET /api/sensor/esp32-01
```

**Response:**
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
  "nearestFireData": "{\"latitude\":\"43.480\",\"longitude\":\"-79.710\",...}",
  "ttl": 1733097600
}
```

**Status Codes:**
- `200 OK` - Success
- `404 Not Found` - Sensor not found
- `500 Internal Server Error` - Server error

---

### 3. Get Risk Map Data

**GET** `/api/risk-map`

Returns risk map data for visualization (last 24 hours).

**Response:**
```json
[
  {
    "deviceId": "esp32-01",
    "lat": 43.467,
    "lng": -79.699,
    "riskScore": 45.2,
    "temperature": 23.4,
    "humidity": 40.2,
    "timestamp": "2025-12-01T16:20:00Z"
  },
  {
    "deviceId": "esp32-01",
    "lat": 43.467,
    "lng": -79.699,
    "riskScore": 46.1,
    "temperature": 23.6,
    "humidity": 39.8,
    "timestamp": "2025-12-01T16:50:00Z"
  }
]
```

**Status Codes:**
- `200 OK` - Success
- `500 Internal Server Error` - Server error

---

## Data Models

### Sensor Data Object

```typescript
interface SensorData {
  deviceId: string;           // Device identifier (e.g., "esp32-01")
  timestamp: string;          // ISO 8601 timestamp (e.g., "2025-12-01T16:20:00Z")
  temperature: number;        // Temperature in Celsius
  humidity: number;           // Humidity percentage (0-100)
  lat: number;                // Latitude (GPS coordinate)
  lng: number;                // Longitude (GPS coordinate)
  riskScore: number;          // Calculated risk score (0-100)
  nearestFireDistance?: number; // Distance to nearest fire in km (optional)
  nearestFireData?: string;   // JSON string of nearest fire data (optional)
  ttl?: number;               // Time-to-live (Unix timestamp, optional)
}
```

### Risk Score Calculation

The risk score is calculated using:

```
riskScore = w1 * temp_score + w2 * humidity_score + w3 * fire_score

Where:
- temp_score = min(temperature / 50.0, 1.0) * 100
- humidity_score = (1.0 - min(humidity / 100.0, 1.0)) * 100
- fire_score = max(0, (100 - min(fire_distance, 100)) / 100.0) * 100
- w1 = 0.4, w2 = 0.3, w3 = 0.3
```

**Risk Score Ranges:**
- `0-30`: Low risk
- `31-60`: Medium risk
- `61-100`: High risk

## IoT Payload Format

Sensors publish to AWS IoT Core with this format:

**Topic:** `wildfire/sensors/{deviceId}`

**Payload:**
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

## CORS

All endpoints support CORS with:
- **Access-Control-Allow-Origin**: `*`
- **Access-Control-Allow-Methods**: `GET, OPTIONS`
- **Access-Control-Allow-Headers**: `Content-Type`

## Error Responses

### Standard Error Format

```json
{
  "error": "Error message description"
}
```

### Common Error Codes

- `400 Bad Request` - Invalid request
- `404 Not Found` - Resource not found
- `500 Internal Server Error` - Server error

## Rate Limiting

Currently no rate limiting implemented. Consider adding for production.

## Authentication

Currently no authentication required. Consider adding API keys or Cognito for production.

---

**API Version**: 1.0  
**Last Updated**: Current Semester

