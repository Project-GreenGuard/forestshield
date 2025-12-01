# ForestShield - Project Overview

## 1.0 Project Description

Project GreenGuard: ForestShield is an AI-driven wildfire response and management system developed as a capstone project. The system collects real-time environmental data from ESP32 IoT sensors deployed in forest environments, integrates NASA FIRMS wildfire detection data, performs automated risk scoring calculations, and visualizes results through a React-based Progressive Web Application dashboard.

The architecture follows a microservices pattern with clear separation between IoT data ingestion, cloud processing, and frontend visualization layers.

## 2.0 Functional Areas

### 2.1 Current Semester Implementation

The following three functional areas are implemented during the current semester:

1. **FA1: IoT Sensor Data Ingestion** - Complete end-to-end pipeline from ESP32 sensors through AWS IoT Core, Lambda processing functions, to DynamoDB storage
2. **FA2: External Wildfire Data Integration** - NASA FIRMS API integration for active fire detection and proximity calculations
3. **FA3: Risk Scoring and Dashboard** - Real-time risk calculation algorithm and web-based visualization interface

### 2.2 Deferred to Next Semester

The following functional areas are planned for next semester implementation:

4. **FA4: Alert System** - AWS SNS notification system for high-risk conditions
5. **FA5: AI Prediction** - GCP Vertex AI integration for advanced ML-based wildfire prediction models

## 3.0 System Components

### 3.1 IoT Sensor Layer

**Hardware Platform:**

- ESP32 Dev Module microcontroller
- DHT11 temperature and humidity sensor

**Functional Responsibilities:**

- Continuous reading of environmental data (temperature, humidity)
- Static GPS coordinate assignment (hardcoded per device)
- JSON payload formatting
- MQTT message transmission over TLS

**Communication Protocol:**

- Protocol: MQTT over TLS (port 8883)
- Destination: AWS IoT Core
- Topic Pattern: `wildfire/sensors/{deviceId}`

**Location Handling:**
ESP32 devices do not include built-in GPS hardware. Sensor locations are statically configured in firmware during device provisioning. Each device is assigned fixed latitude and longitude coordinates that represent the physical deployment location. Coordinates are hardcoded per device and remain constant throughout the device lifecycle.

**Repository:** `forestshield-iot-firmware`

### 3.2 Cloud Processing Layer

**Components:**

- **AWS IoT Core**: Device management, certificate-based authentication, MQTT message broker
- **AWS Lambda**: Serverless compute for sensor data processing, NASA FIRMS API integration, risk score calculation
- **AWS DynamoDB**: NoSQL database for enriched sensor data storage with risk scores

**Data Processing Flow:**

1. IoT Core receives MQTT messages from sensors
2. IoT Rule triggers Lambda function on message receipt
3. Lambda function parses sensor payload
4. Lambda function queries NASA FIRMS API for active fire data
5. Lambda function calculates risk score using rule-based algorithm
6. Lambda function writes enriched data record to DynamoDB

**Repository:** `forestshield-backend`

### 3.3 API Layer

**Components:**

- **AWS API Gateway**: REST API endpoint provisioning with CORS configuration
- **AWS Lambda**: API handler functions for data retrieval

**API Endpoints:**

- `GET /api/sensors` - Retrieve list of all sensors with latest data
- `GET /api/sensor/{id}` - Retrieve latest data for specific sensor by device ID
- `GET /api/risk-map` - Retrieve risk map data points for visualization

**Repository:** `forestshield-backend`

### 3.4 Frontend Dashboard

**Technology Stack:**

- React Progressive Web Application (PWA)
- Leaflet mapping library for geographic visualization
- Real-time data polling mechanism

**Functional Features:**

- Real-time sensor data visualization
- Interactive map with sensor location markers
- Risk heatmap overlay visualization
- Responsive web design for multiple device types

**Repository:** `forestshield-frontend`

### 3.5 Infrastructure Layer

**Technology:**

- Terraform for Infrastructure as Code (IaC)

**Managed Resources:**

- AWS IoT Core configuration (Things, Certificates, Policies, Rules)
- AWS Lambda functions and execution roles
- DynamoDB table definitions
- API Gateway REST API configuration
- IAM roles and policies

**Repository:** `forestshield-infrastructure`

## 4.0 System Architecture and Data Flow

### 4.1 High-Level Architecture

The system follows a unidirectional data flow pattern:

```
ESP32 Sensor Hardware
    ↓ (MQTT over TLS)
AWS IoT Core
    ↓ (IoT Rule Trigger)
AWS Lambda: process_sensor_data
    ↓ (HTTP Request to External API)
NASA FIRMS API
    ↓ (Response Processing)
Risk Score Calculation
    ↓ (Database Write)
AWS DynamoDB
    ↓ (Query Interface)
AWS Lambda: api_handler
    ↓ (REST API)
AWS API Gateway
    ↓ (HTTP/HTTPS)
React Frontend Dashboard
```

### 4.2 Data Flow Sequence

1. ESP32 sensor reads environmental data (temperature, humidity) at 30-second intervals
2. Sensor publishes MQTT message to AWS IoT Core topic `wildfire/sensors/{deviceId}`
3. IoT Core rule triggers Lambda function `process_sensor_data` upon message receipt
4. Lambda function parses sensor payload and extracts location coordinates
5. Lambda function queries NASA FIRMS API for active fire detections in bounding box
6. Lambda function calculates distance to nearest fire detection point
7. Lambda function computes risk score using weighted algorithm
8. Lambda function writes enriched data record to DynamoDB table
9. Frontend dashboard polls API Gateway endpoint at 10-second intervals
10. API Gateway invokes Lambda function `api_handler`
11. Lambda function queries DynamoDB for latest sensor data
12. Frontend receives data and updates map visualization

## 5.0 Data Schema

### 5.1 DynamoDB Table Structure

**Table Name:** `WildfireSensorData`

**Key Schema:**

- **Partition Key:** `deviceId` (String) - Unique identifier for sensor device
- **Sort Key:** `timestamp` (String, ISO 8601 format) - Timestamp of sensor reading

**Item Attributes:**

- `temperature` (Number) - Temperature reading in degrees Celsius
- `humidity` (Number) - Relative humidity percentage
- `lat` (Number) - Latitude coordinate (hardcoded per device)
- `lng` (Number) - Longitude coordinate (hardcoded per device)
- `nearestFireDistance` (Number) - Distance to nearest active fire in kilometers
- `riskScore` (Number) - Calculated risk score (0-100)
- `ttl` (Number) - Time-to-live attribute for automatic record expiration (Unix timestamp)

**Example Item:**

```json
{
  "deviceId": "esp32-01",
  "timestamp": "2025-12-01T16:20:00Z",
  "temperature": 23.4,
  "humidity": 40.2,
  "lat": 43.467,
  "lng": -79.699,
  "nearestFireDistance": 12.5,
  "riskScore": 78,
  "ttl": 1733090400
}
```

**Data Retention:**

- TTL attribute automatically expires records after 30 days
- Expiration managed by DynamoDB TTL feature to optimize storage costs

## 6.0 Risk Scoring Algorithm

### 6.1 Algorithm Specification

The risk scoring algorithm employs a weighted linear combination approach:

```
riskScore = (w1 × temp_score) + (w2 × humidity_score) + (w3 × fire_score)
```

**Component Calculations:**

1. **Temperature Score (temp_score):**

   - Input Range: 0-50°C
   - Normalized Output: 0-100
   - Calculation: `temp_score = (temperature / 50) × 100`
   - Higher temperature yields higher score

2. **Humidity Score (humidity_score):**

   - Input Range: 0-100% relative humidity
   - Normalized Output: 0-100
   - Calculation: `humidity_score = 100 - humidity`
   - Lower humidity yields higher score (inverse relationship)

3. **Fire Proximity Score (fire_score):**
   - Input Range: 0-100km distance to nearest fire
   - Normalized Output: 0-100
   - Calculation: `fire_score = ((100 - distance) / 100) × 100`
   - Closer fire proximity yields higher score (inverse distance relationship)

**Weight Configuration:**

- `w1` (Temperature Weight): 0.4
- `w2` (Humidity Weight): 0.3
- `w3` (Fire Proximity Weight): 0.3
- Weight Sum: 1.0

**Output Range:**

- Final risk score: 0-100 (continuous scale)
- Higher scores indicate increased wildfire risk conditions

### 6.2 Algorithm Limitations

The current implementation uses rule-based calculations. Machine learning-based risk prediction models are deferred to next semester when GCP Vertex AI integration is implemented.

## 7.0 Repository Structure

The project is organized into five independent repositories:

```
Project-GreenGuard/
├── forestshield-iot-firmware/      # ESP32 Arduino firmware code
├── forestshield-backend/           # AWS Lambda functions and API handlers
├── forestshield-frontend/          # React Progressive Web Application
├── forestshield-infrastructure/    # Terraform Infrastructure as Code
└── forestshield/                   # Project documentation and specifications
```

Each repository maintains independent version control history and can be developed and deployed separately.

## 8.0 Project Scope

### 8.1 Current Semester Implementation

**Completed Components:**

- End-to-end IoT data ingestion pipeline (ESP32 → AWS IoT Core → Lambda → DynamoDB)
- NASA FIRMS wildfire data integration via HTTP API
- Rule-based risk scoring algorithm
- Web-based dashboard visualization with interactive map
- Infrastructure provisioning using Terraform
- Lightweight AWS architecture optimized for $100 budget constraint

### 8.2 Deferred to Next Semester

**Out-of-Scope Components:**

- GCP Vertex AI integration for machine learning models
- Advanced ML-based wildfire prediction algorithms
- AWS SNS alerting system for high-risk notifications
- Security hardening components (WAF, GuardDuty, KMS)
- AWS Cognito user authentication and authorization
- CloudFront CDN for frontend asset delivery
- Advanced monitoring and analytics services

## 9.0 Assumptions and Dependencies

### 9.1 Technical Assumptions

1. **Sensor Location Configuration:**

   - ESP32 devices use static, hardcoded GPS coordinates in firmware
   - No GPS hardware available on ESP32 platform
   - Device locations are manually configured during firmware deployment
   - Coordinates represent fixed physical deployment locations

2. **Network Connectivity:**

   - IoT devices require continuous WiFi connectivity for data transmission
   - Network interruptions may result in data loss
   - No local buffering mechanism for extended connectivity loss scenarios

3. **External API Dependencies:**

   - NASA FIRMS API availability and latency assumptions:
     - Fire detection data latency: 2-3 hours from satellite detection to API availability
     - API rate limits: approximately 100 requests per hour per IP address
     - API reliability depends on NASA service availability

4. **Risk Scoring Methodology:**

   - Currently rule-based algorithm, not machine learning-based
   - ML integration deferred to next semester with GCP Vertex AI
   - Algorithm weights are fixed and not dynamically adjusted

5. **Device Management:**
   - Manual certificate provisioning required per device
   - No automated device registration system
   - Device configuration requires firmware update for location changes

### 9.2 Budget Constraints

**AWS Credit Allocation:**

- Total project budget: $100 AWS credits
- Budget constraint drives lightweight infrastructure design decisions

**Cost Optimization Strategy:**

- Maximize AWS Free Tier usage across all services
- DynamoDB: On-demand pricing (pay per request) instead of provisioned capacity
- Lambda: Free tier provides 1 million requests per month
- IoT Core: First 250,000 messages per month free
- API Gateway: First 1 million requests per month free
- Avoid advanced services with associated costs (WAF, GuardDuty, KMS, Cognito)

**Budget Impact on Architecture:**

- Limited device count supported within budget constraints
- Data retention reduced to 30 days via TTL to minimize storage costs
- No redundant services or high-availability configurations
- Regional deployment instead of multi-region architecture

### 9.3 Data and Operational Constraints

1. **Data Retention:**

   - DynamoDB TTL set to 30 days for automatic record expiration
   - Historical data beyond 30 days is not retained
   - Retention period optimized for cost management

2. **Update Frequency Limits:**

   - Sensor transmission rate: 30-second intervals (2,880 messages per device per day)
   - NASA FIRMS API calls: 15-minute intervals per sensor (96 calls per device per day)
   - Limits driven by NASA API rate limits and AWS service quotas

3. **Scalability Limitations:**

   - Current design supports limited device count due to budget constraints
   - No auto-scaling provisions for Lambda concurrency beyond free tier
   - DynamoDB on-demand pricing scales with usage but increases costs

4. **Device Management Process:**
   - Manual certificate provisioning required for each new device
   - Firmware configuration changes require device reprogramming
   - No over-the-air update mechanism

## 10.0 Technology Stack

### 10.1 IoT Layer

- **Microcontroller:** ESP32 Dev Module
- **Sensor:** DHT11 (temperature, humidity)
- **Programming Language:** Arduino C++
- **Communication Protocol:** MQTT over TLS

### 10.2 Cloud Layer

- **Device Management:** AWS IoT Core
- **Compute:** AWS Lambda (Python 3.9+)
- **Database:** AWS DynamoDB (NoSQL)
- **API Gateway:** AWS API Gateway (REST API)

### 10.3 Frontend Layer

- **Framework:** React (PWA)
- **Mapping Library:** Leaflet
- **Styling:** TailwindCSS
- **Runtime:** Node.js 16.x+

### 10.4 Infrastructure Layer

- **Infrastructure as Code:** Terraform (>= 1.0)
- **Version Control:** Git
- **Cloud Provider:** AWS (hackathon profile)

### 10.5 Local Development

- **Container Platform:** Docker (>= 20.x)
- **Orchestration:** Docker Compose (>= 2.0)
- **Local Services:** DynamoDB Local, Mosquitto MQTT broker

## 11.0 Environmental Setup Requirements

### 11.1 Required Tools and Versions

- **AWS CLI:** Configured with `hackathon` profile and appropriate credentials
- **Terraform:** Version >= 1.0 for infrastructure provisioning
- **Node.js:** Version >= 16.x for frontend development and build processes
- **Python:** Version >= 3.9 for Lambda function development and local testing
- **Arduino IDE:** Version >= 2.0 for ESP32 firmware development and compilation
- **Docker:** Version >= 20.x for local service containerization
- **Docker Compose:** Version >= 2.0 for local development environment orchestration

### 11.2 AWS Configuration

**Profile Setup:**

```bash
aws configure --profile hackathon
```

**Required IAM Permissions:**
The configured AWS profile must have appropriate IAM permissions for:

- IoT Core device management (Things, Certificates, Policies, Rules)
- Lambda function deployment and execution
- DynamoDB table creation and management
- API Gateway REST API configuration
- CloudWatch Logs access for monitoring

## 12.0 Data Update Frequencies

### 12.1 Sensor Data Collection

- **ESP32 Transmission Interval:** 30 seconds
- **Message Rate:** Approximately 2,880 messages per device per day
- **DynamoDB Write Operations:** One write operation per sensor reading

### 12.2 External Data Integration

- **NASA FIRMS API Call Frequency:** Every 15 minutes per sensor
- **API Calls per Device:** Approximately 96 calls per device per day
- **Dataset Source:** NASA FIRMS MODIS_NRT (Moderate Resolution Imaging Spectroradiometer Near Real-Time) Active Fire Data
- **Data Latency:** 2-3 hours from satellite detection to API data availability
- **Rate Limit Compliance:** Call frequency adjusted to respect ~100 requests per hour per IP limit

### 12.3 Frontend Data Refresh

- **Dashboard Polling Interval:** Every 10 seconds for sensor data updates
- **Risk Map Update Interval:** Every 30 seconds for map visualization refresh

## 13.0 Key Features

1. **Real-Time Data Collection:** ESP32 sensors transmit environmental data at 30-second intervals with automatic reconnection handling
2. **Wildfire Data Integration:** Automated integration with NASA FIRMS API for active fire detection and proximity analysis
3. **Automated Risk Assessment:** Rule-based risk scoring algorithm combining temperature, humidity, and fire proximity factors
4. **Interactive Visualization:** Web-based dashboard with interactive map, sensor markers, and risk heatmap overlay
5. **Scalable Architecture:** Microservices architecture with loose coupling enabling independent development and deployment

## 14.0 External Integrations

### 14.1 NASA FIRMS API

**API Specification:**

- **Service:** NASA FIRMS (Fire Information for Resource Management System)
- **Dataset:** MODIS_NRT (Moderate Resolution Imaging Spectroradiometer Near Real-Time) Active Fire Data
- **Base URL:** `https://firms.modaps.eosdis.nasa.gov/api/country/csv/{country_code}/MODIS_NRT/1`
- **Response Format:** CSV

**Request Parameters:**

- `country_code`: ISO country code (e.g., "CAN" for Canada) used to retrieve fire detections for the specified country

**Example Request:**

```
GET https://firms.modaps.eosdis.nasa.gov/api/country/csv/CAN/MODIS_NRT/1
```

**Response Structure:**
CSV format containing fire detection point records with latitude, longitude, brightness measurements, confidence indicators, and detection timestamps. The Lambda function parses the CSV response and extracts fire detection coordinates for distance calculations.

**Rate Limits:**

- Public API limit: Approximately 100 requests per hour per IP address
- Implementation refresh frequency: 15 minutes per sensor to respect rate limits

### 14.2 AWS Services

- **AWS IoT Core:** Device management, certificate-based authentication, MQTT message broker
- **AWS Lambda:** Serverless compute for data processing and API request handling
- **AWS DynamoDB:** NoSQL database for sensor data storage with automatic TTL management
- **AWS API Gateway:** REST API endpoint provisioning with CORS configuration

## 15.0 Failure and Fallback Behavior

### 15.1 Error Handling Strategies

**1. NASA FIRMS API Unavailability:**

- Lambda function continues processing with sensor-only data
- Risk score calculated using only temperature and humidity factors
- `nearestFireDistance` attribute set to null or default value (100km maximum distance)
- Error condition logged to CloudWatch Logs for monitoring and troubleshooting

**2. Invalid IoT Sensor Data:**

- Data validation rejects readings with NaN or missing temperature/humidity values
- Lambda function returns error response without DynamoDB write operation
- Device identifier and error details logged for troubleshooting

**3. Lambda Function Exceptions:**

- Exceptions caught and logged to CloudWatch Logs with full stack traces
- Failed sensor readings do not block processing of other devices
- Dead-letter queue pattern identified as future enhancement

**4. DynamoDB Write Failures:**

- Retry logic implemented with exponential backoff (maximum 3 attempts)
- Failed write operations logged to CloudWatch Logs
- System continues processing subsequent sensor readings without blocking

**5. WiFi Connectivity Loss:**

- ESP32 devices may buffer readings locally if storage available (firmware-dependent)
- Automatic reconnection attempts implemented in firmware
- Data loss possible during extended network outages

**6. API Gateway Timeouts:**

- Frontend application handles timeout conditions gracefully
- Cached data displayed if available from previous successful requests
- Connection status indicator shown to users for transparency

## 16.0 Project Timeline

- **Current Semester:** Core functionality implementation including IoT ingestion, external data integration, risk scoring, and dashboard visualization
- **Next Semester:** Machine learning integration via GCP Vertex AI, advanced features including SNS alerting, and security hardening components

## 17.0 Data Privacy

**Privacy Statement:**

This system does not collect, store, or transmit personally identifiable information (PII). Sensors report only environmental metrics including temperature, humidity, and location coordinates. Location data represents sensor deployment coordinates only and does not correspond to individual user locations. No user tracking, authentication, or personal data collection mechanisms are implemented in the current system design.

---

**Project:** Capstone - Hackathon Profile  
**Organization:** Project GreenGuard  
**Service:** ForestShield
