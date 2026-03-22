# ForestShield - Visual Paradigm Software Model Documentation

## Document Overview

This document provides a complete Visual Paradigm software model specification for the ForestShield system. The model is structured according to Sheridan Capstone requirements and consists of six major models: Requirements Model, Domain Model, Design Model, Interaction Model, Deployment Model, and Test Model.

---

## 1. REQUIREMENTS MODEL (RM)

### 1.1 High-Level Business Requirements (BR)

**BR-001: Real-Time Environmental Monitoring**
The system shall collect real-time environmental data (temperature, humidity) from IoT sensors deployed in forest environments to enable continuous monitoring of wildfire risk conditions.

**BR-002: Wildfire Detection Data Integration**
The system shall integrate external wildfire detection data from NASA FIRMS API to identify active fire events in proximity to sensor locations.

**BR-003: Automated Risk Assessment**
The system shall automatically calculate wildfire risk scores based on environmental sensor readings and proximity to active fire events using a rule-based algorithm.

**BR-004: Real-Time Visualization**
The system shall provide a web-based dashboard that displays real-time sensor data, active fire locations, and calculated risk scores through an interactive map interface.

**BR-005: Data Persistence**
The system shall store enriched sensor data with calculated risk scores in a cloud database for historical analysis and dashboard retrieval.

**BR-006: Scalable Architecture**
The system shall employ a cloud-based microservices architecture that supports multiple IoT sensor devices with loose coupling between components.

**BR-007: Cost Optimization**
The system shall operate within a $100 AWS credit budget constraint by leveraging free tier services and on-demand pricing models.

### 1.2 Business Rules (BRG)

**BRG-001: Sensor Data Collection Frequency**
Sensor devices shall transmit environmental data at 30-second intervals during normal operation.

**BRG-002: Fire Data Refresh Rate**
NASA FIRMS API shall be queried at 15-minute intervals per sensor to respect rate limit constraints (approximately 100 requests per hour per IP address).

**BRG-003: Risk Score Calculation**
Risk scores shall be calculated using a weighted algorithm combining temperature (weight: 0.4), humidity (weight: 0.3), and fire proximity (weight: 0.3) factors, producing a value on a scale of 0-100.

**BRG-004: Data Retention Period**
Sensor data records shall be retained for 30 days after which automatic expiration occurs via database time-to-live (TTL) mechanism.

**BRG-005: Sensor Location Configuration**
Sensor location coordinates (latitude, longitude) shall be statically configured in device firmware during deployment and shall not change during device operation.

**BRG-006: Frontend Update Frequency**
Dashboard shall poll API endpoints every 10 seconds for sensor data updates and every 30 seconds for risk map visualization updates.

**BRG-007: Budget Constraint**
System infrastructure shall utilize AWS free tier services where available and on-demand pricing for DynamoDB to remain within $100 total budget.

### 1.3 System Context Description (SCD)

The ForestShield system operates as a cloud-based IoT data processing and visualization platform. The system boundary encompasses:

**Internal System Components:**

- IoT sensor data ingestion pipeline
- Cloud-based data processing functions
- External data integration services
- Risk calculation engine
- Web-based dashboard application
- Cloud infrastructure services

**External Actors:**

- **ESP32 Sensor Devices:** Hardware devices deployed in forest environments that collect and transmit environmental data
- **NASA FIRMS API:** External web service providing active wildfire detection data
- **System Operators:** Users who access the web dashboard to monitor sensor data and risk assessments
- **AWS Cloud Services:** Platform-as-a-Service components (IoT Core, Lambda, DynamoDB, API Gateway) providing runtime infrastructure

**Input Relationships:**

- ESP32 sensors transmit MQTT messages containing temperature, humidity, and location data to AWS IoT Core
- System queries NASA FIRMS API to retrieve active fire detection data based on sensor locations

**Output Relationships:**

- System provides REST API endpoints that return sensor data and risk assessments to the web dashboard
- Dashboard displays real-time visualizations of sensor locations, environmental conditions, and calculated risk scores

**System Boundary:**
The system boundary includes all cloud-based processing components but excludes the physical sensor hardware itself (which is treated as an external actor) and the external NASA FIRMS service.

### 1.4 Summary Use Case (SUC)

**Use Case: Monitor Wildfire Risk**

**Primary Actor:** System Operator

**Goal:** Monitor real-time environmental conditions and wildfire risk assessments across multiple sensor locations

**Scope:** ForestShield system

**Brief Description:**
System operators access the web dashboard to view real-time environmental data from deployed IoT sensors, active wildfire detections from NASA FIRMS, and automated risk score calculations. The system continuously collects sensor data, integrates external fire detection data, calculates risk scores, and presents results through an interactive map interface.

**Main Success Scenario:**

1. System collects environmental data from ESP32 sensors every 30 seconds
2. System integrates NASA FIRMS wildfire data every 15 minutes per sensor
3. System calculates risk scores using rule-based algorithm
4. System stores enriched data in cloud database
5. System operator accesses web dashboard
6. Dashboard displays real-time sensor data and risk visualizations
7. System operator monitors risk conditions across all sensor locations

**Included Use Cases:**

- FA1-UC-001: Ingest Sensor Data
- FA2-UC-001: Integrate Wildfire Data
- FA3-UC-001: Calculate Risk Score
- FA3-UC-002: Visualize Risk Dashboard

### 1.5 Functional Area Use Cases

#### Functional Area 1 (FA1): IoT Sensor Data Ingestion

**FA1-UC-001: Ingest Sensor Data**

**Goal:** Capture environmental sensor readings from IoT devices and store them in the cloud database

**Scope:** ForestShield system

**Primary Actor:** ESP32 Sensor Device

**Pre-conditions:**

- ESP32 sensor device is deployed and operational
- Device has valid AWS IoT Core certificate and connection
- Device has configured WiFi connectivity
- AWS IoT Core is operational and accessible

**Post-conditions:**

- Sensor reading is received by AWS IoT Core
- Sensor data is processed and stored in DynamoDB
- Data record includes timestamp, device identifier, and sensor readings

**Trigger:** ESP32 sensor device publishes MQTT message at scheduled 30-second interval

**Main Flow:**

1. ESP32 sensor reads temperature and humidity values from DHT11 sensor
2. Sensor formats JSON payload with deviceId, temperature, humidity, lat, lng, and timestamp
3. Sensor publishes MQTT message to topic `wildfire/sensors/{deviceId}` on AWS IoT Core
4. AWS IoT Core receives message and validates device certificate
5. IoT Rule triggers Lambda function `process_sensor_data` upon message receipt
6. Lambda function receives event payload containing sensor data
7. Lambda function validates data format and values
8. Lambda function stores validated data in DynamoDB table `WildfireSensorData`
9. System confirms successful data ingestion

**Alternate Flow 3a: Network Connectivity Loss**
3a.1. ESP32 device detects WiFi connection loss
3a.2. Device attempts automatic reconnection
3a.3. If reconnection successful, device resumes normal operation (return to step 2)
3a.4. If reconnection fails, device logs error and data may be lost

**Alternate Flow 7a: Invalid Sensor Data**
7a.1. Lambda function detects missing or invalid temperature/humidity values
7a.2. Lambda function rejects data and logs error to CloudWatch
7a.3. No data record is written to DynamoDB
7a.4. System continues processing subsequent sensor readings

**Exceptional Flow 8a: DynamoDB Write Failure**
8a.1. DynamoDB write operation fails due to service unavailability
8a.2. Lambda function implements retry logic with exponential backoff (maximum 3 attempts)
8a.3. If all retry attempts fail, error is logged to CloudWatch
8a.4. System continues processing subsequent sensor readings without blocking

**Exceptional Flow 4a: Certificate Validation Failure**
4a.1. AWS IoT Core rejects message due to invalid or expired certificate
4a.2. Message is discarded
4a.3. Error logged to IoT Core logs
4a.4. Device requires certificate renewal before resuming operation

---

#### Functional Area 2 (FA2): External Wildfire Data Integration

**FA2-UC-001: Integrate Wildfire Data**

**Goal:** Retrieve active wildfire detection data from NASA FIRMS API and associate it with sensor locations

**Scope:** ForestShield system

**Primary Actor:** System (automated process)

**Pre-conditions:**

- Sensor data has been ingested and stored in DynamoDB
- Sensor location coordinates (latitude, longitude) are available
- NASA FIRMS API is accessible and operational
- API rate limits have not been exceeded

**Post-conditions:**

- NASA FIRMS API has been queried for active fire detections
- Nearest fire detection point has been identified for sensor location
- Distance to nearest fire has been calculated
- Fire proximity data is available for risk score calculation

**Trigger:** Lambda function `process_sensor_data` is triggered by sensor data ingestion

**Main Flow:**

1. Lambda function receives sensor data with location coordinates
2. Lambda function constructs bounding box around sensor location (±0.5° radius)
3. Lambda function formats NASA FIRMS API request with bounding box and current date
4. Lambda function sends HTTP GET request to NASA FIRMS API endpoint
5. NASA FIRMS API returns JSON array of active fire detection points
6. Lambda function parses response and extracts fire detection coordinates
7. Lambda function calculates distance from sensor location to each fire detection point
8. Lambda function identifies nearest fire detection point
9. Lambda function calculates distance in kilometers to nearest fire
10. Fire proximity data is stored with sensor data for risk calculation

**Alternate Flow 4a: No Active Fires Detected**
4a.1. NASA FIRMS API returns empty array (no active fires in bounding box)
4a.2. Lambda function sets nearestFireKm to maximum default value (100km)
4a.3. Fire proximity score defaults to minimum value
4a.4. Risk calculation proceeds with sensor-only data

**Exceptional Flow 4b: NASA FIRMS API Unavailable**
4b.1. HTTP request to NASA FIRMS API fails or times out
4b.2. Lambda function catches exception and logs error to CloudWatch
4b.3. Lambda function sets nearestFireKm to default value (100km)
4b.4. Risk calculation proceeds using only temperature and humidity factors
4b.5. System continues processing without blocking

**Exceptional Flow 5a: Invalid API Response Format**
5a.1. NASA FIRMS API returns malformed JSON or unexpected data structure
5a.2. Lambda function catches parsing exception
5a.3. Lambda function logs error and sets nearestFireKm to default value
5a.4. Risk calculation proceeds with sensor-only data

**Exceptional Flow 4c: API Rate Limit Exceeded**
4c.1. NASA FIRMS API returns HTTP 429 (Too Many Requests) status code
4c.2. Lambda function logs rate limit warning to CloudWatch
4c.3. Lambda function sets nearestFireKm to default value
4c.4. Risk calculation proceeds with sensor-only data
4c.5. Subsequent requests will respect 15-minute interval per sensor

---

#### Functional Area 3 (FA3): Risk Scoring and Dashboard

**FA3-UC-001: Calculate Risk Score**

**Goal:** Calculate wildfire risk score based on environmental sensor data and fire proximity

**Scope:** ForestShield system

**Primary Actor:** System (automated process)

**Pre-conditions:**

- Sensor data (temperature, humidity) has been ingested
- Fire proximity data has been integrated (or default value assigned)
- All required data components are available for calculation

**Post-conditions:**

- Risk score has been calculated using rule-based algorithm
- Risk score value (0-100) is stored with sensor data record
- Enriched data record is complete and stored in DynamoDB

**Trigger:** Lambda function has received sensor data and completed fire data integration

**Main Flow:**

1. Lambda function retrieves temperature value from sensor data
2. Lambda function retrieves humidity value from sensor data
3. Lambda function retrieves nearest fire distance (or default value)
4. Lambda function normalizes temperature value (0-50°C mapped to 0-100)
5. Lambda function calculates inverse humidity score (100 - humidity)
6. Lambda function calculates inverse fire proximity score ((100 - distance) / 100 × 100)
7. Lambda function applies weights: temperature (0.4), humidity (0.3), fire proximity (0.3)
8. Lambda function calculates final risk score: (w1 × temp_score) + (w2 × humidity_score) + (w3 × fire_score)
9. Lambda function stores risk score with sensor data in DynamoDB
10. System confirms risk score calculation complete

**Alternate Flow 1a: Missing Temperature Data**
1a.1. Temperature value is null or invalid
1a.2. Lambda function rejects calculation and logs error
1a.3. Data record is not stored
1a.4. Exception handling returns error response

**Alternate Flow 3a: Fire Data Unavailable**
3a.1. Fire proximity data is unavailable (NASA API failure)
3a.2. Lambda function uses default fire distance (100km)
3a.3. Fire proximity score is set to minimum value
3a.4. Risk calculation proceeds with temperature and humidity only

---

**FA3-UC-002: Visualize Risk Dashboard**

**Goal:** Display real-time sensor data, fire locations, and risk assessments through interactive web dashboard

**Scope:** ForestShield system

**Primary Actor:** System Operator

**Pre-conditions:**

- Sensor data has been ingested and stored in DynamoDB
- Risk scores have been calculated
- Web dashboard is accessible via browser
- API Gateway endpoints are operational

**Post-conditions:**

- Dashboard displays current sensor locations on interactive map
- Dashboard displays environmental data for each sensor
- Dashboard displays risk scores and visual indicators
- Dashboard displays active fire locations from NASA FIRMS data
- Dashboard updates automatically at configured intervals

**Trigger:** System operator navigates to dashboard URL in web browser

**Main Flow:**

1. System operator opens web browser and navigates to dashboard URL
2. React application loads and initializes map component
3. Dashboard sends GET request to `/api/sensors` endpoint
4. API Gateway receives request and invokes Lambda function `api_handler`
5. Lambda function queries DynamoDB for latest sensor data for all devices
6. Lambda function formats response with sensor data, locations, and risk scores
7. API Gateway returns JSON response to dashboard
8. Dashboard renders sensor markers on map at configured coordinates
9. Dashboard displays sensor data panels with temperature, humidity, and risk scores
10. Dashboard sends GET request to `/api/risk-map` endpoint for map visualization
11. API Gateway invokes Lambda function for risk map data
12. Lambda function queries DynamoDB for risk score data points
13. Dashboard renders risk heatmap overlay on map
14. Dashboard configures automatic polling (10 seconds for sensor data, 30 seconds for risk map)
15. System operator monitors risk conditions across all sensors

**Alternate Flow 3a: No Sensors Available**
3a.1. DynamoDB query returns no sensor records
3a.2. Lambda function returns empty array
3a.3. Dashboard displays message indicating no sensors available
3a.4. Map displays without sensor markers

**Alternate Flow 5a: Partial Data Retrieval**
5a.1. DynamoDB query returns data for some but not all sensors
5a.2. Lambda function returns available sensor data
5a.3. Dashboard displays markers only for sensors with available data
5a.4. Missing sensor indicators shown in data panel

**Exceptional Flow 4a: API Gateway Timeout**
4a.1. API Gateway request times out before Lambda function responds
4a.2. Dashboard catches timeout error
4a.3. Dashboard displays cached data if available
4a.4. Dashboard shows connection status indicator
4a.5. Dashboard retries request on next polling interval

**Exceptional Flow 5a: DynamoDB Query Failure**
5a.1. DynamoDB query operation fails
5a.2. Lambda function catches exception and logs error
5a.3. Lambda function returns error response to API Gateway
5a.4. Dashboard receives error response and displays error message
5a.5. Dashboard continues polling on next interval

---

### 1.6 Functional Area Overview Diagrams

#### FA1: IoT Sensor Data Ingestion Overview

**Functional Area:** FA1 - IoT Sensor Data Ingestion

**Purpose:** Capture environmental data from deployed ESP32 sensors and store in cloud database

**Key Components:**

- ESP32 sensor devices (hardware)
- AWS IoT Core (device management and message broker)
- Lambda function: process_sensor_data (data processing)
- DynamoDB table: WildfireSensorData (data storage)

**Data Flow:**
ESP32 Sensor → MQTT Message → AWS IoT Core → IoT Rule → Lambda Function → DynamoDB

**Key Responsibilities:**

- Sensor data collection (30-second intervals)
- Secure message transmission (MQTT over TLS)
- Data validation and parsing
- Database storage operations

---

#### FA2: External Wildfire Data Integration Overview

**Functional Area:** FA2 - External Wildfire Data Integration

**Purpose:** Retrieve active wildfire detection data from NASA FIRMS API and calculate proximity to sensor locations

**Key Components:**

- Lambda function: process_sensor_data (integration logic)
- NASA FIRMS API (external web service)
- HTTP client for API communication

**Data Flow:**
Sensor Location → Lambda Function → HTTP Request → NASA FIRMS API → Fire Detection Data → Distance Calculation

**Key Responsibilities:**

- API request formatting with bounding box parameters
- HTTP request execution and response handling
- Fire detection data parsing
- Distance calculation between sensor and fire locations
- Error handling and fallback mechanisms

---

#### FA3: Risk Scoring and Dashboard Overview

**Functional Area:** FA3 - Risk Scoring and Dashboard

**Purpose:** Calculate wildfire risk scores and provide real-time visualization through web dashboard

**Key Components:**

- Lambda function: process_sensor_data (risk calculation)
- Lambda function: api_handler (API endpoints)
- API Gateway (REST API)
- React PWA Dashboard (frontend visualization)

**Data Flow:**
Sensor Data + Fire Data → Risk Calculation → DynamoDB Storage → API Query → Dashboard Display

**Key Responsibilities:**

- Risk score algorithm execution
- Database query operations for dashboard data
- REST API endpoint provisioning
- Interactive map visualization
- Real-time data updates via polling

---

### 1.7 Feature Packages

**Feature Package: FA1 - IoT Sensor Data Ingestion**

- Feature: Sensor Data Collection
- Feature: Secure Message Transmission
- Feature: Data Validation
- Feature: Database Storage
- Feature: Error Handling and Retry Logic

**Feature Package: FA2 - External Wildfire Data Integration**

- Feature: NASA FIRMS API Integration
- Feature: Fire Detection Data Retrieval
- Feature: Distance Calculation
- Feature: API Error Handling
- Feature: Rate Limit Management

**Feature Package: FA3 - Risk Scoring and Dashboard**

- Feature: Risk Score Calculation
- Feature: REST API Endpoints
- Feature: Interactive Map Visualization
- Feature: Real-Time Data Polling
- Feature: Risk Heatmap Overlay

---

## 2. DOMAIN MODEL (DM)

### 2.1 Glossary of Domain Terms

**Sensor:** A physical IoT device (ESP32 microcontroller with DHT11 sensor) deployed in a forest environment that collects and transmits environmental data including temperature and humidity measurements.

**Sensor Reading:** A single data point collected by a sensor device at a specific point in time, containing temperature, humidity, and location information.

**Location:** A geographic coordinate pair (latitude, longitude) representing the physical deployment position of a sensor device. Locations are statically configured in device firmware.

**Wildfire:** An uncontrolled fire occurring in forest or wildland areas that poses risk to the environment and infrastructure.

**Fire Detection:** An active fire event identified by NASA FIRMS satellite imaging system, represented as a geographic coordinate with associated brightness and confidence measurements.

**Fire Proximity:** The calculated distance in kilometers between a sensor location and the nearest active fire detection point.

**Risk Score:** A numerical value on a scale of 0-100 representing the calculated wildfire risk score based on environmental conditions and fire proximity. Higher scores indicate increased risk.

**Temperature:** Environmental temperature reading in degrees Celsius collected by sensor device.

**Humidity:** Relative humidity percentage collected by sensor device. Lower humidity values indicate drier conditions associated with increased fire risk.

**Bounding Box:** A geographic area defined by minimum and maximum latitude and longitude coordinates used to query NASA FIRMS API for active fire detections within a region surrounding a sensor location.

**Dashboard:** A web-based user interface that displays real-time sensor data, active fire locations, and risk assessments through interactive map and data panel components.

**Device Identifier:** A unique string identifier assigned to each sensor device (e.g., "esp32-01") used to distinguish devices and associate data records.

**Timestamp:** An ISO 8601 formatted date-time value representing when a sensor reading was collected or when data was processed.

**Risk Calculation:** The algorithmic process of combining normalized temperature, humidity, and fire proximity factors with weighted coefficients to produce a risk score.

### 2.2 Domain Class Diagram

The domain model represents the essential concepts in the problem domain without implementation-specific details.

**Domain Classes:**

```
<<concept>> Sensor
- deviceId: String
- location: Location
+ collectReading(): SensorReading

<<concept>> Location
- latitude: Float
- longitude: Float
+ calculateDistanceTo(other: Location): Float

<<concept>> SensorReading
- timestamp: DateTime
- temperature: Float
- humidity: Float
- sensor: Sensor
+ isValid(): Boolean

<<concept>> FireDetection
- latitude: Float
- longitude: Float
- brightness: Float
- confidence: Float
- detectionTime: DateTime
+ getLocation(): Location

<<concept>> RiskScore
- value: Integer (0-100)
- calculationTime: DateTime
- sensorReading: SensorReading
- fireProximity: FireProximity
+ calculate(): Integer

<<concept>> FireProximity
- distanceKm: Float
- nearestFire: FireDetection
- sensorLocation: Location
+ calculateDistance(): Float

<<concept>> BoundingBox
- minLatitude: Float
- maxLatitude: Float
- minLongitude: Float
- maxLongitude: Float
+ contains(location: Location): Boolean
+ expandAround(location: Location, radius: Float): BoundingBox
```

**Domain Class Relationships:**

- Sensor _has_ Location (composition)
- Sensor _generates_ SensorReading (1 to many)
- SensorReading _associated with_ RiskScore (1 to 1)
- RiskScore _uses_ FireProximity (composition)
- FireProximity _references_ FireDetection (association)
- FireProximity _references_ Location (association)
- BoundingBox _contains_ Location (spatial relationship)

**Domain Model Notes:**

The domain model focuses on core problem domain concepts:

- Sensor and SensorReading represent the data collection domain
- FireDetection represents external wildfire information
- RiskScore represents the risk assessment domain concept
- Location and FireProximity represent geographic/spatial relationships

The domain model excludes:

- Technical infrastructure (AWS services, databases, APIs)
- User interface components
- Controller or service classes
- Implementation-specific data storage mechanisms
- Communication protocols or message formats

---

## 3. DESIGN MODEL (DSM)

### 3.1 Architectural Pattern

**Selected Pattern: Microservices Architecture**

The ForestShield system employs a microservices architectural pattern with the following characteristics:

**Rationale for Selection:**

1. **Loose Coupling:** Components are independently deployable with minimal dependencies, allowing separate development and maintenance cycles for IoT ingestion, data processing, and frontend visualization.

2. **Scalability Requirements:** Different components have varying scalability needs. Sensor ingestion requires high-throughput message processing, while API endpoints require request-handling scalability. Microservices enable independent scaling.

3. **Technology Diversity:** The system integrates multiple technology stacks (ESP32 firmware, Python Lambda functions, React frontend, Terraform infrastructure). Microservices accommodate technology heterogeneity.

4. **Budget Constraints:** Cloud-based microservices on AWS enable pay-per-use pricing models that align with $100 budget constraint, avoiding unnecessary resource provisioning.

5. **Team Structure:** Multiple development teams (IoT, Backend, Frontend, DevOps) can work independently on separate services with clear boundaries.

6. **Failure Isolation:** Service failures in one component (e.g., NASA API integration) do not cascade to other components (e.g., sensor data storage continues operating).

**Architectural Characteristics:**

- Service boundaries aligned with functional areas (FA1, FA2, FA3)
- API-based communication between services
- Independent deployment and versioning
- Cloud-native service composition
- Event-driven processing (IoT Rule triggers)

### 3.2 Software System Architecture (SSA)

The Software System Architecture consists of the following subsystems:

**Subsystem 1: IoT Ingestion Subsystem**

- Responsibility: Receive, validate, and store sensor data from IoT devices
- Components: AWS IoT Core, Lambda function (process_sensor_data), DynamoDB
- Interfaces: MQTT protocol for device communication, DynamoDB API for storage

**Subsystem 2: External Data Integration Subsystem**

- Responsibility: Integrate wildfire detection data from NASA FIRMS API
- Components: Lambda function (process_sensor_data - integration logic), HTTP client
- Interfaces: HTTP REST API for NASA FIRMS, internal data structures

**Subsystem 3: Risk Engine Subsystem**

- Responsibility: Calculate risk scores using rule-based algorithm
- Components: Lambda function (process_sensor_data - calculation logic)
- Interfaces: Internal data structures, DynamoDB for data retrieval and storage

**Subsystem 4: API Subsystem**

- Responsibility: Provide REST API endpoints for frontend data retrieval
- Components: API Gateway, Lambda function (api_handler)
- Interfaces: REST API for frontend, DynamoDB for data queries

**Subsystem 5: Frontend/Visualization Subsystem**

- Responsibility: Display sensor data, fire locations, and risk assessments through web interface
- Components: React PWA application, Leaflet mapping library
- Interfaces: REST API calls to API Gateway, browser rendering APIs

**Subsystem 6: Infrastructure/IaC Subsystem**

- Responsibility: Define and provision cloud infrastructure resources
- Components: Terraform configuration files
- Interfaces: AWS Cloud APIs for resource provisioning

### 3.3 Subsystem Class Diagrams

#### Subsystem 1: IoT Ingestion Subsystem

**Boundary Classes:**

- `IoTMessageReceiver` - Receives and validates MQTT messages from AWS IoT Core

**Control Classes:**

- `SensorDataProcessor` - Coordinates sensor data processing workflow

**Entity Classes:**

- `SensorData` - Represents sensor reading data structure
- `SensorRecord` - Represents database record structure

**Class Diagram:**

```
<<boundary>> IoTMessageReceiver
+ receiveMessage(event: IoTEvent): SensorData
+ validateMessageFormat(data: JSON): Boolean

<<control>> SensorDataProcessor
+ processSensorData(data: SensorData): Boolean
+ validateData(data: SensorData): Boolean
+ storeData(record: SensorRecord): Boolean

<<entity>> SensorData
- deviceId: String
- temperature: Float
- humidity: Float
- latitude: Float
- longitude: Float
- timestamp: DateTime
+ isValid(): Boolean

<<entity>> SensorRecord
- deviceId: String
- timestamp: String
- temperature: Float
- humidity: Float
- lat: Float
- lng: Float
- ttl: Integer
+ toDynamoDBItem(): Map
```

#### Subsystem 2: External Data Integration Subsystem

**Boundary Classes:**

- `NASAFIRMSAPIClient` - Handles HTTP communication with NASA FIRMS API

**Control Classes:**

- `FireDataIntegrator` - Coordinates fire data integration workflow

**Entity Classes:**

- `FireDetectionData` - Represents fire detection response from API
- `FireProximityData` - Represents calculated fire proximity information

**Class Diagram:**

```
<<boundary>> NASAFIRMSAPIClient
+ queryActiveFires(bbox: BoundingBox, date: String): FireDetectionData[]
+ handleAPIError(error: Exception): FireDetectionData[]

<<control>> FireDataIntegrator
+ integrateFireData(sensorLocation: Location): FireProximityData
+ calculateDistance(location1: Location, location2: Location): Float
+ findNearestFire(fires: FireDetectionData[], sensorLocation: Location): FireDetectionData

<<entity>> FireDetectionData
- latitude: Float
- longitude: Float
- brightness: Float
- confidence: Float
+ getLocation(): Location

<<entity>> FireProximityData
- distanceKm: Float
- nearestFire: FireDetectionData
+ getDistance(): Float
```

#### Subsystem 3: Risk Engine Subsystem

**Control Classes:**

- `RiskScoreCalculator` - Coordinates risk score calculation workflow

**Entity Classes:**

- `RiskScore` - Represents calculated risk score value
- `RiskCalculationFactors` - Represents input factors for risk calculation

**Class Diagram:**

```
<<control>> RiskScoreCalculator
+ calculateRiskScore(factors: RiskCalculationFactors): RiskScore
+ normalizeTemperature(temp: Float): Float
+ normalizeHumidity(humidity: Float): Float
+ normalizeFireProximity(distance: Float): Float
+ applyWeights(tempScore: Float, humidityScore: Float, fireScore: Float): Float

<<entity>> RiskScore
- value: Integer (0-100)
- calculationTime: DateTime
+ getValue(): Integer
+ isValid(): Boolean

<<entity>> RiskCalculationFactors
- temperature: Float
- humidity: Float
- fireDistanceKm: Float
+ getTemperature(): Float
+ getHumidity(): Float
+ getFireDistance(): Float
```

#### Subsystem 4: API Subsystem

**Boundary Classes:**

- `APIRequestHandler` - Receives and parses HTTP requests from API Gateway
- `APIResponseBuilder` - Formats HTTP responses for API Gateway

**Control Classes:**

- `SensorDataController` - Coordinates data retrieval for API endpoints
- `RiskMapController` - Coordinates risk map data retrieval

**Entity Classes:**

- `APIRequest` - Represents incoming API request
- `APIResponse` - Represents API response structure
- `SensorDataDTO` - Data transfer object for sensor data
- `RiskMapPoint` - Represents risk map data point

**Class Diagram:**

```
<<boundary>> APIRequestHandler
+ parseRequest(event: APIGatewayEvent): APIRequest
+ extractPathParameters(event: APIGatewayEvent): Map
+ extractQueryParameters(event: APIGatewayEvent): Map

<<boundary>> APIResponseBuilder
+ buildResponse(data: Object, statusCode: Integer): APIResponse
+ buildErrorResponse(error: Exception): APIResponse

<<control>> SensorDataController
+ getAllSensors(): SensorDataDTO[]
+ getSensorById(deviceId: String): SensorDataDTO
+ queryDynamoDB(deviceId: String): SensorRecord

<<control>> RiskMapController
+ getRiskMapData(): RiskMapPoint[]
+ queryRiskDataPoints(): SensorRecord[]

<<entity>> APIRequest
- httpMethod: String
- path: String
- pathParameters: Map
- queryParameters: Map
+ getDeviceId(): String

<<entity>> APIResponse
- statusCode: Integer
- body: String (JSON)
- headers: Map
+ getStatusCode(): Integer

<<entity>> SensorDataDTO
- deviceId: String
- temperature: Float
- humidity: Float
- latitude: Float
- longitude: Float
- riskScore: Integer
- timestamp: String

<<entity>> RiskMapPoint
- latitude: Float
- longitude: Float
- riskScore: Integer
```

#### Subsystem 5: Frontend/Visualization Subsystem

**Boundary Classes:**

- `APIClient` - Handles HTTP communication with backend API
- `MapRenderer` - Renders interactive map using Leaflet library

**Control Classes:**

- `DashboardController` - Coordinates dashboard data flow and updates
- `SensorDataService` - Manages sensor data state and polling
- `RiskMapService` - Manages risk map data and updates

**Entity Classes:**

- `SensorViewModel` - View model for sensor data display
- `FireDetectionViewModel` - View model for fire detection display
- `RiskMapViewModel` - View model for risk map visualization

**Class Diagram:**

```
<<boundary>> APIClient
+ getSensors(): Promise<SensorDataDTO[]>
+ getSensorById(id: String): Promise<SensorDataDTO>
+ getRiskMapData(): Promise<RiskMapPoint[]>
+ handleError(error: Error): void

<<boundary>> MapRenderer
+ renderMap(center: Location, zoom: Integer): void
+ addSensorMarker(sensor: SensorViewModel): void
+ addFireMarker(fire: FireDetectionViewModel): void
+ renderRiskHeatmap(points: RiskMapPoint[]): void
+ updateMarker(sensor: SensorViewModel): void

<<control>> DashboardController
+ initializeDashboard(): void
+ startPolling(): void
+ stopPolling(): void
+ updateSensorData(): Promise<void>
+ updateRiskMap(): Promise<void>

<<control>> SensorDataService
- sensors: SensorViewModel[]
+ fetchSensorData(): Promise<void>
+ updateSensor(sensor: SensorViewModel): void
+ getSensorById(id: String): SensorViewModel

<<control>> RiskMapService
- riskPoints: RiskMapPoint[]
+ fetchRiskMapData(): Promise<void>
+ updateRiskPoints(points: RiskMapPoint[]): void

<<entity>> SensorViewModel
- deviceId: String
- temperature: Float
- humidity: Float
- riskScore: Integer
- latitude: Float
- longitude: Float
- lastUpdate: DateTime

<<entity>> FireDetectionViewModel
- latitude: Float
- longitude: Float
- brightness: Float

<<entity>> RiskMapViewModel
- points: RiskMapPoint[]
- bounds: BoundingBox
```

#### Subsystem 6: Infrastructure/IaC Subsystem

**Control Classes:**

- `TerraformProvisioner` - Coordinates infrastructure provisioning

**Entity Classes:**

- `InfrastructureConfig` - Represents infrastructure configuration
- `ResourceDefinition` - Represents individual resource definitions

**Class Diagram:**

```
<<control>> TerraformProvisioner
+ provisionInfrastructure(config: InfrastructureConfig): Boolean
+ validateConfiguration(config: InfrastructureConfig): Boolean
+ destroyInfrastructure(): Boolean

<<entity>> InfrastructureConfig
- iotCoreConfig: IoTCoreConfig
- lambdaConfig: LambdaConfig[]
- dynamoDBConfig: DynamoDBConfig
- apiGatewayConfig: APIGatewayConfig
+ validate(): Boolean

<<entity>> ResourceDefinition
- resourceType: String
- resourceName: String
- properties: Map
+ toTerraformHCL(): String
```

### 3.4 Subsystem Interfaces

**Interface: IoT Ingestion ↔ Risk Engine**

- Data Format: SensorData entity
- Communication: Internal function call within Lambda process_sensor_data
- Synchronous: Yes

**Interface: External Data Integration ↔ Risk Engine**

- Data Format: FireProximityData entity
- Communication: Internal function call within Lambda process_sensor_data
- Synchronous: Yes

**Interface: Risk Engine ↔ API Subsystem**

- Data Format: SensorRecord stored in DynamoDB
- Communication: Database query (asynchronous)
- Synchronous: No (database read)

**Interface: API Subsystem ↔ Frontend**

- Data Format: JSON over HTTP (SensorDataDTO, RiskMapPoint)
- Communication: REST API calls
- Synchronous: Yes (HTTP request/response)

**Interface: Infrastructure ↔ All Subsystems**

- Data Format: AWS resource configurations
- Communication: Terraform provisioning to AWS Cloud APIs
- Synchronous: Yes (provisioning operations)

### 3.5 Architectural Decision Rationale

**Decision 1: Microservices over Monolithic Architecture**

- Rationale: Enables independent scaling, deployment, and development cycles aligned with team structure and budget constraints.

**Decision 2: Serverless Computing (AWS Lambda)**

- Rationale: Eliminates server management overhead, provides automatic scaling, and aligns with pay-per-use pricing model for budget optimization.

**Decision 3: NoSQL Database (DynamoDB) over Relational Database**

- Rationale: Supports high-throughput write operations from IoT devices, provides automatic scaling, and offers on-demand pricing that fits budget constraints.

**Decision 4: Event-Driven Processing (IoT Rules)**

- Rationale: Enables real-time processing of sensor messages without polling, reducing latency and computational overhead.

**Decision 5: REST API over GraphQL**

- Rationale: Simpler implementation for current use case, adequate for dashboard data requirements, and reduces development complexity.

**Decision 6: Progressive Web Application (PWA) over Native Mobile App**

- Rationale: Single codebase for multiple platforms, easier deployment and updates, and reduces development and maintenance costs.

---

## 4. INTERACTION MODEL (IM)

### 4.1 Structural Overview

The Interaction Model describes how system components collaborate to realize use cases. Components interact through:

- MQTT message passing (IoT devices to cloud)
- HTTP REST API calls (frontend to backend)
- Database queries (Lambda functions to DynamoDB)
- Internal function calls (within Lambda functions)
- External API calls (Lambda to NASA FIRMS)

### 4.2 Collaboration Overview

**Collaboration: Sensor Data Ingestion Flow**

- Participants: ESP32 Sensor, AWS IoT Core, Lambda (process_sensor_data), DynamoDB
- Purpose: Capture and store sensor readings
- Interaction Pattern: Event-driven message passing

**Collaboration: Fire Data Integration Flow**

- Participants: Lambda (process_sensor_data), NASA FIRMS API
- Purpose: Retrieve and process wildfire detection data
- Interaction Pattern: Synchronous HTTP request/response

**Collaboration: Risk Score Calculation Flow**

- Participants: Lambda (process_sensor_data) internal components
- Purpose: Calculate risk score from sensor and fire data
- Interaction Pattern: Internal function calls

**Collaboration: Dashboard Data Retrieval Flow**

- Participants: React Dashboard, API Gateway, Lambda (api_handler), DynamoDB
- Purpose: Retrieve and display sensor data and risk assessments
- Interaction Pattern: REST API request/response with database queries

### 4.3 Sequence Diagrams for Architecturally Significant Use Cases

#### Sequence Diagram: FA1-UC-001 - Ingest Sensor Data

```
Actor: ESP32 Sensor
Participant: AWS IoT Core
Participant: IoTMessageReceiver
Participant: SensorDataProcessor
Participant: DynamoDB

ESP32 Sensor -> AWS IoT Core: Publish MQTT Message (sensor data)
AWS IoT Core -> AWS IoT Core: Validate Certificate
AWS IoT Core -> IoTMessageReceiver: Trigger Lambda (IoT Event)
IoTMessageReceiver -> IoTMessageReceiver: Validate Message Format
IoTMessageReceiver -> SensorDataProcessor: processSensorData(data)
SensorDataProcessor -> SensorDataProcessor: Validate Data Values
SensorDataProcessor -> DynamoDB: Store SensorRecord
DynamoDB -> SensorDataProcessor: Confirm Write Success
SensorDataProcessor -> IoTMessageReceiver: Return Success
IoTMessageReceiver -> AWS IoT Core: Processing Complete
```

#### Sequence Diagram: FA2-UC-001 - Integrate Wildfire Data

```
Participant: SensorDataProcessor
Participant: FireDataIntegrator
Participant: NASAFIRMSAPIClient
Participant: NASA FIRMS API

SensorDataProcessor -> FireDataIntegrator: integrateFireData(sensorLocation)
FireDataIntegrator -> FireDataIntegrator: Calculate BoundingBox
FireDataIntegrator -> NASAFIRMSAPIClient: queryActiveFires(bbox, date)
NASAFIRMSAPIClient -> NASA FIRMS API: HTTP GET Request
NASA FIRMS API -> NASAFIRMSAPIClient: JSON Response (fire detections)
NASAFIRMSAPIClient -> FireDataIntegrator: Return FireDetectionData[]
FireDataIntegrator -> FireDataIntegrator: Calculate Distance to Each Fire
FireDataIntegrator -> FireDataIntegrator: Find Nearest Fire
FireDataIntegrator -> FireDataIntegrator: Create FireProximityData
FireDataIntegrator -> SensorDataProcessor: Return FireProximityData
```

#### Sequence Diagram: FA3-UC-001 - Calculate Risk Score

```
Participant: SensorDataProcessor
Participant: RiskScoreCalculator
Participant: RiskCalculationFactors
Participant: RiskScore

SensorDataProcessor -> RiskScoreCalculator: calculateRiskScore(factors)
RiskScoreCalculator -> RiskCalculationFactors: getTemperature()
RiskCalculationFactors -> RiskScoreCalculator: Return temperature value
RiskScoreCalculator -> RiskScoreCalculator: normalizeTemperature()
RiskScoreCalculator -> RiskCalculationFactors: getHumidity()
RiskCalculationFactors -> RiskScoreCalculator: Return humidity value
RiskScoreCalculator -> RiskScoreCalculator: normalizeHumidity()
RiskScoreCalculator -> RiskCalculationFactors: getFireDistance()
RiskCalculationFactors -> RiskScoreCalculator: Return fire distance
RiskScoreCalculator -> RiskScoreCalculator: normalizeFireProximity()
RiskScoreCalculator -> RiskScoreCalculator: applyWeights(w1=0.4, w2=0.3, w3=0.3)
RiskScoreCalculator -> RiskScore: Create RiskScore(value)
RiskScoreCalculator -> SensorDataProcessor: Return RiskScore
```

#### Sequence Diagram: FA3-UC-002 - Visualize Risk Dashboard

```
Actor: System Operator
Participant: DashboardController
Participant: APIClient
Participant: API Gateway
Participant: APIRequestHandler
Participant: SensorDataController
Participant: DynamoDB
Participant: MapRenderer

System Operator -> DashboardController: Open Dashboard URL
DashboardController -> DashboardController: Initialize Components
DashboardController -> APIClient: getSensors()
APIClient -> API Gateway: HTTP GET /api/sensors
API Gateway -> APIRequestHandler: Parse Request
APIRequestHandler -> SensorDataController: getAllSensors()
SensorDataController -> DynamoDB: Query Latest Sensor Data
DynamoDB -> SensorDataController: Return SensorRecord[]
SensorDataController -> APIRequestHandler: Format SensorDataDTO[]
APIRequestHandler -> API Gateway: Return JSON Response
API Gateway -> APIClient: Return Sensor Data
APIClient -> DashboardController: Update Sensor Data
DashboardController -> MapRenderer: renderMap()
DashboardController -> MapRenderer: addSensorMarker() [for each sensor]
DashboardController -> DashboardController: Configure Polling (10s interval)
DashboardController -> APIClient: getRiskMapData()
APIClient -> API Gateway: HTTP GET /api/risk-map
API Gateway -> APIRequestHandler: Parse Request
APIRequestHandler -> RiskMapController: getRiskMapData()
RiskMapController -> DynamoDB: Query Risk Data Points
DynamoDB -> RiskMapController: Return SensorRecord[]
RiskMapController -> APIRequestHandler: Format RiskMapPoint[]
APIRequestHandler -> API Gateway: Return JSON Response
API Gateway -> APIClient: Return Risk Map Data
APIClient -> DashboardController: Update Risk Map Data
DashboardController -> MapRenderer: renderRiskHeatmap()
```

### 4.4 Internal Subsystem Interaction Descriptions

**Internal Interaction: Lambda process_sensor_data Function**

Within the Lambda function `process_sensor_data`, internal components interact as follows:

1. **IoTMessageReceiver** receives IoT event from AWS IoT Core and extracts sensor data payload
2. **SensorDataProcessor** validates and parses sensor data into SensorData entity
3. **FireDataIntegrator** is invoked to retrieve fire detection data from NASA FIRMS API
4. **NASAFIRMSAPIClient** makes HTTP request and processes response into FireDetectionData entities
5. **FireDataIntegrator** calculates distances and creates FireProximityData
6. **RiskScoreCalculator** receives SensorData and FireProximityData to calculate risk score
7. **RiskCalculationFactors** entity aggregates input factors
8. **RiskScore** entity is created with calculated value
9. **SensorDataProcessor** combines all data into SensorRecord entity
10. **SensorRecord** is written to DynamoDB via database client

All interactions within the Lambda function are synchronous function calls. No external messaging or asynchronous communication occurs internally.

---

## 5. DEPLOYMENT MODEL (DPM)

### 5.1 Physical Deployment Topology

The ForestShield system is deployed across multiple physical and virtual environments:

**Device Layer:**

- Physical hardware: ESP32 microcontroller devices with DHT11 sensors
- Deployment: Distributed across forest environments at fixed geographic locations
- Network: WiFi connectivity to internet for MQTT communication

**Cloud Runtime Layer:**

- AWS Cloud Platform (Regional deployment, e.g., us-east-1)
- Virtual execution environments for serverless functions
- Managed database services
- API gateway services

**Web Client Layer:**

- End-user web browsers (desktop, tablet, mobile)
- Progressive Web Application hosted via static file serving or CDN

### 5.2 Nodes and Execution Environments

#### Node 1: ESP32 Sensor Device

**Physical Hardware:**

- ESP32 Dev Module microcontroller
- DHT11 temperature/humidity sensor
- WiFi module (integrated)

**Execution Environment:**

- Arduino firmware runtime
- MQTT client library
- Sensor reading libraries

**Deployed Services:**

- Sensor data collection service (30-second intervals)
- MQTT message publishing service
- WiFi connection management service

**Network Configuration:**

- WiFi connection to local network
- MQTT over TLS to AWS IoT Core endpoint (port 8883)
- Certificate-based authentication

---

#### Node 2: AWS IoT Core

**Deployment Type:** AWS Managed Service

**Execution Environment:**

- AWS IoT Core message broker
- Device registry and certificate management
- IoT Rule engine

**Deployed Services:**

- MQTT message broker service
- Device authentication service (X.509 certificates)
- IoT Rule execution service (triggers Lambda on message receipt)

**Network Configuration:**

- Public internet endpoint for device connections
- Internal AWS network for Lambda invocation
- TLS encryption for all MQTT communications

---

#### Node 3: AWS Lambda Execution Environment

**Deployment Type:** AWS Serverless Compute

**Execution Environment:**

- AWS Lambda runtime (Python 3.9+)
- Function: `wildfire-process-sensor-data`
- Function: `wildfire-api-handler`

**Deployed Services:**

**Function 1: wildfire-process-sensor-data**

- IoT message processing service
- NASA FIRMS API integration service
- Risk score calculation service
- DynamoDB write service

**Function 2: wildfire-api-handler**

- REST API request handling service
- DynamoDB query service
- Response formatting service

**Network Configuration:**

- Invoked via AWS IoT Core (internal network)
- Invoked via API Gateway (internal network)
- Outbound HTTP access to NASA FIRMS API (public internet)
- VPC configuration: Default (no custom VPC)

**Resource Configuration:**

- Memory: 256 MB (process_sensor_data), 128 MB (api_handler)
- Timeout: 30 seconds
- Concurrent execution limits: Per AWS account limits

---

#### Node 4: AWS DynamoDB

**Deployment Type:** AWS Managed NoSQL Database

**Execution Environment:**

- DynamoDB service (on-demand pricing)
- Table: `WildfireSensorData`

**Deployed Services:**

- Data storage service
- Query service
- TTL expiration service (automatic record cleanup)

**Table Configuration:**

- Partition Key: `deviceId` (String)
- Sort Key: `timestamp` (String)
- TTL Attribute: `ttl` (Number)
- Billing Mode: On-demand
- Point-in-time recovery: Disabled (cost optimization)

**Network Configuration:**

- Accessible via AWS SDK from Lambda functions (internal network)
- No public internet access required

---

#### Node 5: AWS API Gateway

**Deployment Type:** AWS Managed API Service

**Execution Environment:**

- REST API Gateway service
- API stage: `prod`

**Deployed Services:**

- REST API endpoint provisioning
- Request routing service
- Lambda integration service
- CORS configuration service

**API Configuration:**

- Endpoints:
  - `GET /api/sensors`
  - `GET /api/sensor/{id}`
  - `GET /api/risk-map`
- Integration Type: Lambda Proxy Integration
- CORS: Enabled for frontend origin

**Network Configuration:**

- Public HTTPS endpoint for frontend access
- Internal network connection to Lambda functions
- SSL/TLS termination at API Gateway

---

#### Node 6: Web Client Environment

**Deployment Type:** End-user web browser

**Execution Environment:**

- Modern web browser (Chrome, Firefox, Safari, Edge)
- JavaScript runtime (V8, SpiderMonkey, JavaScriptCore)
- React application runtime

**Deployed Services:**

- React PWA application
- Leaflet map rendering service
- HTTP client service (API polling)
- Local state management service

**Network Configuration:**

- HTTPS connection to API Gateway endpoint
- Polling interval: 10 seconds (sensor data), 30 seconds (risk map)

**Hosting Options:**

- Static file hosting (AWS S3 + CloudFront - deferred to next semester)
- Alternative: Simple web server or local development server
- Current semester: Local development or simple hosting solution

---

#### Node 7: External Services

**Node 7a: NASA FIRMS API**

**Deployment Type:** External web service

**Network Configuration:**

- Public HTTPS endpoint: `https://firms.modaps.eosdis.nasa.gov/api/viirs/active_fires`
- Accessible via public internet from Lambda functions
- No authentication required for public API

**Node 7b: Terraform Execution Environment**

**Deployment Type:** Developer local machine or CI/CD pipeline

**Execution Environment:**

- Terraform CLI (>= 1.0)
- AWS CLI with configured credentials

**Deployed Services:**

- Infrastructure provisioning service
- Resource state management service

**Network Configuration:**

- HTTPS connection to AWS Cloud APIs for resource provisioning
- Requires IAM permissions for resource creation/modification

### 5.3 Communication Links

**Link 1: ESP32 Sensor → AWS IoT Core**

- Protocol: MQTT over TLS
- Port: 8883
- Encryption: TLS 1.2+
- Authentication: X.509 device certificates
- Direction: Unidirectional (sensor publishes, IoT Core receives)

**Link 2: AWS IoT Core → AWS Lambda (process_sensor_data)**

- Protocol: AWS IoT Rule trigger (internal AWS service invocation)
- Mechanism: Event-driven invocation
- Direction: Unidirectional (IoT Core triggers Lambda)

**Link 3: AWS Lambda → NASA FIRMS API**

- Protocol: HTTP/HTTPS
- Method: GET
- Port: 443
- Encryption: TLS
- Direction: Bidirectional (request/response)

**Link 4: AWS Lambda → AWS DynamoDB**

- Protocol: AWS SDK API calls (HTTPS internally)
- Method: PutItem, Query operations
- Encryption: TLS (internal AWS network)
- Direction: Bidirectional (read/write operations)

**Link 5: Web Browser → AWS API Gateway**

- Protocol: HTTP/HTTPS
- Method: GET
- Port: 443
- Encryption: TLS
- Direction: Bidirectional (request/response)

**Link 6: AWS API Gateway → AWS Lambda (api_handler)**

- Protocol: Lambda Proxy Integration (internal AWS service invocation)
- Mechanism: Synchronous invocation
- Direction: Bidirectional (request/response)

**Link 7: AWS Lambda (api_handler) → AWS DynamoDB**

- Protocol: AWS SDK API calls (HTTPS internally)
- Method: Query, Scan operations
- Encryption: TLS (internal AWS network)
- Direction: Bidirectional (read operations)

### 5.4 Cloud Component Deployment

**AWS Region:** us-east-1 (or other selected region)

**Deployment Architecture:**

- All cloud components deployed within single AWS region
- No multi-region redundancy (cost optimization for budget constraint)
- Availability Zone: Default (managed by AWS services)

**Component Dependencies:**

- Lambda functions depend on DynamoDB table existence
- API Gateway depends on Lambda function existence
- IoT Core rules depend on Lambda function existence
- All components depend on IAM role configurations

**Deployment Process:**

1. Terraform provisions infrastructure resources
2. IoT Core Things and Certificates created for devices
3. DynamoDB table created
4. Lambda functions deployed with IAM roles
5. API Gateway REST API created and integrated with Lambda
6. IoT Rules configured to trigger Lambda on message receipt

### 5.5 Device Layer Deployment

**Sensor Device Deployment:**

- Physical installation at forest locations
- Firmware configuration with:
  - WiFi credentials
  - AWS IoT Core endpoint URL
  - Device certificate and private key
  - Static location coordinates (latitude, longitude)
  - Device identifier
- Power supply: Battery or solar panel (device-specific)
- Environmental protection: Weather-resistant enclosure

**Device Management:**

- Certificate provisioning: Manual per device
- Firmware updates: Manual (no over-the-air update in current semester)
- Device registration: Manual Thing creation in AWS IoT Core

---

## 6. MODEL CONSISTENCY AND TRACEABILITY

### 6.1 Requirements to Domain Model Traceability

**BR-001 → Domain Classes:**

- Real-Time Environmental Monitoring → Sensor, SensorReading

**BR-002 → Domain Classes:**

- Wildfire Detection Data Integration → FireDetection, BoundingBox

**BR-003 → Domain Classes:**

- Automated Risk Assessment → RiskScore, RiskCalculationFactors

**BR-004 → Domain Classes:**

- Real-Time Visualization → (Conceptual: Dashboard domain concept implied)

**BR-005 → Domain Classes:**

- Data Persistence → SensorReading (storage concept in implementation)

**Use Cases → Domain Classes:**

- FA1-UC-001 → Sensor, SensorReading, Location
- FA2-UC-001 → FireDetection, FireProximity, BoundingBox, Location
- FA3-UC-001 → RiskScore, RiskCalculationFactors, SensorReading, FireProximity
- FA3-UC-002 → SensorReading, RiskScore, Location (visualization domain)

### 6.2 Requirements to Design Model Traceability

**BR-001 → Design Subsystems:**

- Real-Time Environmental Monitoring → IoT Ingestion Subsystem

**BR-002 → Design Subsystems:**

- Wildfire Detection Data Integration → External Data Integration Subsystem

**BR-003 → Design Subsystems:**

- Automated Risk Assessment → Risk Engine Subsystem

**BR-004 → Design Subsystems:**

- Real-Time Visualization → Frontend/Visualization Subsystem

**BR-005 → Design Subsystems:**

- Data Persistence → IoT Ingestion Subsystem (DynamoDB integration)

**Use Cases → Design Subsystems:**

- FA1-UC-001 → IoT Ingestion Subsystem
- FA2-UC-001 → External Data Integration Subsystem
- FA3-UC-001 → Risk Engine Subsystem
- FA3-UC-002 → Frontend/Visualization Subsystem, API Subsystem

### 6.3 Use Cases to Sequence Diagrams Traceability

**FA1-UC-001 → Sequence Diagram:**

- Ingest Sensor Data use case fully represented in FA1 sequence diagram

**FA2-UC-001 → Sequence Diagram:**

- Integrate Wildfire Data use case fully represented in FA2 sequence diagram

**FA3-UC-001 → Sequence Diagram:**

- Calculate Risk Score use case fully represented in FA3 sequence diagram

**FA3-UC-002 → Sequence Diagram:**

- Visualize Risk Dashboard use case fully represented in Dashboard sequence diagram

### 6.4 Design Model to Deployment Model Traceability

**Design Subsystems → Deployment Nodes:**

- IoT Ingestion Subsystem → ESP32 Sensor Device, AWS IoT Core, AWS Lambda, AWS DynamoDB
- External Data Integration Subsystem → AWS Lambda, NASA FIRMS API
- Risk Engine Subsystem → AWS Lambda
- API Subsystem → AWS API Gateway, AWS Lambda, AWS DynamoDB
- Frontend/Visualization Subsystem → Web Browser, React PWA
- Infrastructure/IaC Subsystem → Terraform Execution Environment

### 6.5 Terminology Consistency

**Consistent Terminology Across Models:**

- Sensor / Sensor Device (RM, DM, DSM, IM, DPM)
- Sensor Reading (RM, DM, DSM)
- Location / GPS Coordinates (RM, DM, DSM, IM, DPM)
- Fire Detection / Active Fire (RM, DM, DSM, IM)
- Risk Score (RM, DM, DSM, IM, DPM)
- Device Identifier / deviceId (RM, DM, DSM, IM, DPM)
- Temperature / Humidity (RM, DM, DSM, IM)

**Architectural Terminology:**

- Microservices Architecture (DSM)
- Serverless Computing (DSM, DPM)
- Event-Driven Processing (DSM, IM, DPM)
- REST API (DSM, IM, DPM)

---

## 7. ARCHITECTURAL PATTERN JUSTIFICATION

### 7.1 Pattern Selection: Microservices Architecture

The ForestShield system employs a microservices architectural pattern. This selection is justified based on the following system constraints and requirements:

**Justification 1: Budget Constraint ($100 AWS Credits)**

- Microservices enable pay-per-use pricing models
- Independent scaling prevents over-provisioning of resources
- Serverless components (Lambda) charge only for actual execution time
- On-demand database pricing aligns with usage patterns

**Justification 2: Team Structure and Development Parallelism**

- Multiple development teams (IoT, Backend, Frontend, DevOps)
- Clear service boundaries enable independent development cycles
- Reduced coordination overhead between teams
- Separate repositories and deployment pipelines

**Justification 3: Technology Diversity**

- ESP32 firmware (Arduino C++)
- Cloud processing (Python Lambda functions)
- Frontend (React/JavaScript)
- Infrastructure (Terraform/HCL)
- Microservices accommodate heterogeneous technology stacks

**Justification 4: Scalability Requirements**

- IoT ingestion: High-throughput message processing
- API endpoints: Request-handling scalability
- Frontend: Static asset serving
- Independent scaling of services based on load patterns

**Justification 5: Failure Isolation**

- Service failures do not cascade across boundaries
- NASA API failures do not block sensor data storage
- Frontend failures do not affect data processing
- Database failures isolated to affected service

**Justification 6: Deployment Independence**

- Services can be updated independently
- No full system redeployment required for component changes
- Faster iteration cycles for individual services
- Reduced deployment risk

**Alternative Patterns Considered:**

**Monolithic Architecture:**

- Rejected due to tight coupling, single deployment unit, and difficulty scaling individual components independently

**Layered Architecture:**

- Rejected due to inability to accommodate technology diversity and deployment independence requirements

**Event-Driven Architecture:**

- Partially adopted (IoT Rules trigger Lambda functions)
- Not fully event-driven due to synchronous API calls and REST API patterns

**Hybrid Approach:**

- Current implementation combines microservices with event-driven elements (IoT Rule triggers)
- Represents a hybrid microservices-event-driven architecture suitable for the system's requirements

---

## 8. TEST MODEL (TM)

### 8.1 Test Strategy

The ForestShield system employs a multi-level testing strategy covering unit testing, integration testing, and system testing. Testing is organized by functional areas (FA1, FA2, FA3) and aligned with use case scenarios.

**Testing Levels:**

1. **Unit Testing:** Individual function and component testing within Lambda functions
2. **Integration Testing:** Component interaction testing (IoT Core → Lambda → DynamoDB, API Gateway → Lambda → DynamoDB)
3. **System Testing:** End-to-end workflow testing from sensor data ingestion to dashboard visualization
4. **API Testing:** REST API endpoint validation and response verification

**Testing Approach:**

- **Functional Testing:** Verify use case scenarios and business requirements
- **Data Validation Testing:** Verify sensor data validation, error handling, and edge cases
- **Performance Testing:** Verify system response times and throughput
- **Error Handling Testing:** Verify exception handling and fallback mechanisms

### 8.2 Test Cases by Functional Area

#### 8.2.1 Functional Area 1 (FA1): IoT Sensor Data Ingestion

**TC-FA1-001: Valid Sensor Data Ingestion**

**Test Case ID:** TC-FA1-001  
**Use Case:** FA1-UC-001 (Ingest Sensor Data)  
**Test Level:** Integration  
**Priority:** High

**Objective:** Verify that valid sensor data is successfully ingested, processed, and stored in DynamoDB.

**Pre-conditions:**

- AWS IoT Core is operational
- Lambda function `wildfire-process-sensor-data` is deployed
- DynamoDB table `WildfireSensorData` exists
- Valid IoT device certificate is configured

**Test Data:**

```json
{
  "deviceId": "esp32-test-01",
  "temperature": 25.5,
  "humidity": 45.2,
  "lat": 43.467,
  "lng": -79.699,
  "timestamp": "2025-12-01T16:20:00Z"
}
```

**Test Steps:**

1. Publish MQTT message to topic `wildfire/sensors/esp32-test-01` with test data
2. Verify IoT Core receives message
3. Verify IoT Rule triggers Lambda function
4. Verify Lambda function processes data successfully
5. Query DynamoDB for record with deviceId="esp32-test-01" and matching timestamp
6. Verify DynamoDB record contains all expected fields

**Expected Results:**

- MQTT message is successfully published
- Lambda function is invoked
- DynamoDB record is created with correct data
- Record includes deviceId, timestamp, temperature, humidity, lat, lng, riskScore, ttl

**Pass Criteria:** All steps complete successfully, DynamoDB record matches input data

---

**TC-FA1-002: Invalid Sensor Data Rejection**

**Test Case ID:** TC-FA1-002  
**Use Case:** FA1-UC-001 (Alternate Flow 7a)  
**Test Level:** Integration  
**Priority:** High

**Objective:** Verify that invalid sensor data (missing or invalid values) is rejected and not stored.

**Pre-conditions:**

- Same as TC-FA1-001

**Test Data:**

```json
{
  "deviceId": "esp32-test-01",
  "temperature": null,
  "humidity": 45.2,
  "lat": 43.467,
  "lng": -79.699,
  "timestamp": "2025-12-01T16:20:00Z"
}
```

**Test Steps:**

1. Publish MQTT message with invalid data (null temperature)
2. Verify Lambda function receives message
3. Verify Lambda function validates data and rejects invalid input
4. Verify error is logged to CloudWatch
5. Verify no DynamoDB record is created

**Expected Results:**

- Lambda function rejects invalid data
- Error logged to CloudWatch Logs
- No DynamoDB record created

**Pass Criteria:** Invalid data is rejected, error logged, no database write occurs

---

**TC-FA1-003: Network Connectivity Loss Handling**

**Test Case ID:** TC-FA1-003  
**Use Case:** FA1-UC-001 (Alternate Flow 3a)  
**Test Level:** System  
**Priority:** Medium

**Objective:** Verify ESP32 device handles network connectivity loss and reconnection.

**Pre-conditions:**

- ESP32 device is operational and connected
- WiFi network is available

**Test Steps:**

1. Verify device is connected and transmitting data
2. Disconnect WiFi network
3. Verify device detects connection loss
4. Verify device attempts reconnection
5. Reconnect WiFi network
6. Verify device successfully reconnects
7. Verify device resumes normal data transmission

**Expected Results:**

- Device detects connection loss
- Device attempts automatic reconnection
- Device successfully reconnects when network available
- Device resumes data transmission after reconnection

**Pass Criteria:** Device handles connectivity loss gracefully and resumes operation

---

#### 8.2.2 Functional Area 2 (FA2): External Wildfire Data Integration

**TC-FA2-001: Successful NASA FIRMS API Integration**

**Test Case ID:** TC-FA2-001  
**Use Case:** FA2-UC-001 (Integrate Wildfire Data)  
**Test Level:** Integration  
**Priority:** High

**Objective:** Verify that NASA FIRMS API is successfully queried and fire detection data is retrieved.

**Pre-conditions:**

- Lambda function is operational
- NASA FIRMS API is accessible
- Sensor data with location coordinates is available

**Test Data:**

- Sensor location: lat=43.467, lng=-79.699
- Country code: "CAN"

**Test Steps:**

1. Trigger Lambda function with sensor data containing location coordinates
2. Verify Lambda function constructs NASA FIRMS API request
3. Verify HTTP GET request is sent to NASA FIRMS API
4. Verify API response is received and parsed
5. Verify fire detection data is extracted from response
6. Verify distance calculation is performed

**Expected Results:**

- API request is successfully sent
- API response is received (may be empty if no fires)
- Response is parsed correctly
- Fire detection data is available for distance calculation

**Pass Criteria:** API integration completes successfully, response is parsed correctly

---

**TC-FA2-002: NASA FIRMS API Unavailability Handling**

**Test Case ID:** TC-FA2-002  
**Use Case:** FA2-UC-001 (Exceptional Flow 4b)  
**Test Level:** Integration  
**Priority:** High

**Objective:** Verify that system handles NASA FIRMS API unavailability gracefully.

**Pre-conditions:**

- Lambda function is operational
- NASA FIRMS API is configured to be unavailable (simulated)

**Test Steps:**

1. Trigger Lambda function with sensor data
2. Simulate NASA FIRMS API timeout or connection failure
3. Verify Lambda function catches exception
4. Verify error is logged to CloudWatch
5. Verify default fire distance value (100km) is assigned
6. Verify risk calculation proceeds with sensor-only data
7. Verify DynamoDB record is created with default fire distance

**Expected Results:**

- Exception is caught and handled
- Error logged to CloudWatch Logs
- Default fire distance (100km) is used
- Risk calculation completes successfully
- Data record is stored in DynamoDB

**Pass Criteria:** System continues operation with default values when API unavailable

---

**TC-FA2-003: No Active Fires Detected**

**Test Case ID:** TC-FA2-003  
**Use Case:** FA2-UC-001 (Alternate Flow 4a)  
**Test Level:** Integration  
**Priority:** Medium

**Objective:** Verify that system handles empty fire detection response correctly.

**Pre-conditions:**

- Lambda function is operational
- NASA FIRMS API returns empty array (no active fires)

**Test Steps:**

1. Trigger Lambda function with sensor data
2. Verify NASA FIRMS API is queried
3. Verify API returns empty array
4. Verify Lambda function sets nearestFireDistance to default value (100km)
5. Verify risk calculation proceeds with default fire distance
6. Verify DynamoDB record is created

**Expected Results:**

- API query completes successfully
- Empty response is handled correctly
- Default fire distance is assigned
- Risk calculation completes
- Data record is stored

**Pass Criteria:** Empty fire detection response is handled correctly

---

#### 8.2.3 Functional Area 3 (FA3): Risk Scoring and Dashboard

**TC-FA3-001: Risk Score Calculation**

**Test Case ID:** TC-FA3-001  
**Use Case:** FA3-UC-001 (Calculate Risk Score)  
**Test Level:** Unit  
**Priority:** High

**Objective:** Verify that risk score is calculated correctly using the rule-based algorithm.

**Pre-conditions:**

- Lambda function is operational
- Sensor data and fire proximity data are available

**Test Data:**

- Temperature: 30.0°C
- Humidity: 30.0%
- Fire distance: 25.0 km

**Test Steps:**

1. Invoke risk score calculation function with test data
2. Verify temperature normalization (30/50 \* 100 = 60)
3. Verify humidity normalization (100 - 30 = 70)
4. Verify fire proximity normalization ((100 - 25) / 100 \* 100 = 75)
5. Verify weight application (0.4 _ 60 + 0.3 _ 70 + 0.3 \* 75)
6. Verify final risk score calculation

**Expected Results:**

- Temperature score: 60
- Humidity score: 70
- Fire proximity score: 75
- Final risk score: (0.4 _ 60) + (0.3 _ 70) + (0.3 \* 75) = 67.5

**Pass Criteria:** Risk score calculation matches expected formula and weights

---

**TC-FA3-002: Risk Score Edge Cases**

**Test Case ID:** TC-FA3-002  
**Use Case:** FA3-UC-001  
**Test Level:** Unit  
**Priority:** Medium

**Objective:** Verify risk score calculation handles edge cases correctly.

**Test Cases:**

**Case 1: Maximum Temperature**

- Temperature: 50.0°C, Humidity: 50%, Fire distance: 50km
- Expected: Risk score should cap at 100

**Case 2: Minimum Values**

- Temperature: 0°C, Humidity: 100%, Fire distance: 100km
- Expected: Risk score should be near 0

**Case 3: Missing Fire Data**

- Temperature: 25°C, Humidity: 50%, Fire distance: null
- Expected: Risk score calculated with fire_score = 0

**Test Steps:**

1. Execute risk calculation for each edge case
2. Verify results are within valid range (0-100)
3. Verify calculations handle null/missing values

**Expected Results:**

- All edge cases produce valid risk scores (0-100)
- Missing values are handled gracefully
- Calculations do not produce errors

**Pass Criteria:** All edge cases handled correctly, no calculation errors

---

**TC-FA3-003: Dashboard Data Retrieval**

**Test Case ID:** TC-FA3-003  
**Use Case:** FA3-UC-002 (Visualize Risk Dashboard)  
**Test Level:** Integration  
**Priority:** High

**Objective:** Verify that dashboard successfully retrieves and displays sensor data.

**Pre-conditions:**

- API Gateway is operational
- Lambda function `wildfire-api-handler` is deployed
- DynamoDB contains sensor data records
- Frontend dashboard is accessible

**Test Steps:**

1. Access dashboard URL in web browser
2. Verify dashboard sends GET request to `/api/sensors`
3. Verify API Gateway routes request to Lambda function
4. Verify Lambda function queries DynamoDB
5. Verify Lambda function returns JSON response
6. Verify dashboard receives and parses response
7. Verify sensor markers are displayed on map
8. Verify sensor data panels display correct values

**Expected Results:**

- API request is successful (HTTP 200)
- JSON response contains sensor data array
- Dashboard displays sensor markers on map
- Data panels show temperature, humidity, risk scores

**Pass Criteria:** Dashboard successfully retrieves and displays sensor data

---

**TC-FA3-004: Risk Map Data Retrieval**

**Test Case ID:** TC-FA3-004  
**Use Case:** FA3-UC-002  
**Test Level:** Integration  
**Priority:** Medium

**Objective:** Verify that risk map data is successfully retrieved and displayed.

**Pre-conditions:**

- Same as TC-FA3-003
- DynamoDB contains risk score data from last 24 hours

**Test Steps:**

1. Access dashboard URL
2. Verify dashboard sends GET request to `/api/risk-map`
3. Verify API Gateway routes request to Lambda function
4. Verify Lambda function queries DynamoDB for last 24 hours
5. Verify Lambda function returns risk map data points
6. Verify dashboard receives response
7. Verify risk heatmap overlay is rendered on map

**Expected Results:**

- API request is successful
- Response contains risk map data points with coordinates and risk scores
- Risk heatmap is displayed on map

**Pass Criteria:** Risk map data is retrieved and visualized correctly

---

**TC-FA3-005: Dashboard Polling Mechanism**

**Test Case ID:** TC-FA3-005  
**Use Case:** FA3-UC-002  
**Test Level:** System  
**Priority:** Medium

**Objective:** Verify that dashboard automatically polls API endpoints at configured intervals.

**Pre-conditions:**

- Dashboard is operational
- Backend API is operational

**Test Steps:**

1. Access dashboard
2. Verify initial data load
3. Monitor network requests in browser DevTools
4. Verify GET request to `/api/sensors` occurs every 10 seconds
5. Verify GET request to `/api/risk-map` occurs every 30 seconds
6. Verify dashboard updates with new data

**Expected Results:**

- Polling intervals are correct (10s for sensors, 30s for risk map)
- Dashboard updates automatically
- No polling errors occur

**Pass Criteria:** Polling mechanism functions correctly at specified intervals

---

### 8.3 API Endpoint Test Cases

**TC-API-001: Get All Sensors Endpoint**

**Test Case ID:** TC-API-001  
**Endpoint:** GET `/api/sensors`  
**Test Level:** Integration  
**Priority:** High

**Test Steps:**

1. Send HTTP GET request to `/api/sensors`
2. Verify response status code is 200
3. Verify response contains JSON array
4. Verify each sensor object contains required fields
5. Verify CORS headers are present

**Expected Results:**

- Status: 200 OK
- Content-Type: application/json
- Response body: Array of sensor objects
- CORS header: Access-Control-Allow-Origin: \*

**Pass Criteria:** Endpoint returns valid JSON response with correct structure

---

**TC-API-002: Get Sensor by ID Endpoint**

**Test Case ID:** TC-API-002  
**Endpoint:** GET `/api/sensor/{id}`  
**Test Level:** Integration  
**Priority:** High

**Test Steps:**

1. Send HTTP GET request to `/api/sensor/esp32-test-01`
2. Verify response status code is 200
3. Verify response contains single sensor object
4. Verify sensor object contains all required fields
5. Test with non-existent device ID
6. Verify response status code is 404

**Expected Results:**

- Valid device ID: Status 200, sensor object returned
- Invalid device ID: Status 404, error message returned

**Pass Criteria:** Endpoint handles valid and invalid device IDs correctly

---

**TC-API-003: Get Risk Map Data Endpoint**

**Test Case ID:** TC-API-003  
**Endpoint:** GET `/api/risk-map`  
**Test Level:** Integration  
**Priority:** Medium

**Test Steps:**

1. Send HTTP GET request to `/api/risk-map`
2. Verify response status code is 200
3. Verify response contains JSON array of risk map points
4. Verify each point contains lat, lng, riskScore
5. Verify data is filtered to last 24 hours

**Expected Results:**

- Status: 200 OK
- Response: Array of risk map data points
- Each point contains: deviceId, lat, lng, riskScore, timestamp

**Pass Criteria:** Endpoint returns valid risk map data

---

### 8.4 Error Handling Test Cases

**TC-ERROR-001: DynamoDB Write Failure**

**Test Case ID:** TC-ERROR-001  
**Test Level:** Integration  
**Priority:** High

**Objective:** Verify retry logic and error handling for DynamoDB write failures.

**Test Steps:**

1. Simulate DynamoDB unavailability
2. Trigger Lambda function with valid sensor data
3. Verify Lambda function attempts DynamoDB write
4. Verify retry logic executes (maximum 3 attempts)
5. Verify error is logged to CloudWatch after all retries fail
6. Verify Lambda function returns error response

**Expected Results:**

- Retry logic executes with exponential backoff
- Maximum 3 retry attempts
- Error logged to CloudWatch Logs
- Lambda function handles failure gracefully

**Pass Criteria:** Error handling and retry logic function correctly

---

**TC-ERROR-002: API Gateway Timeout**

**Test Case ID:** TC-ERROR-002  
**Test Level:** System  
**Priority:** Medium

**Objective:** Verify dashboard handles API Gateway timeout gracefully.

**Test Steps:**

1. Configure API Gateway timeout (simulate slow Lambda)
2. Send API request from dashboard
3. Verify request times out
4. Verify dashboard catches timeout error
5. Verify dashboard displays cached data if available
6. Verify dashboard shows connection status indicator
7. Verify dashboard retries on next polling interval

**Expected Results:**

- Timeout is handled gracefully
- Cached data displayed if available
- Connection status shown to user
- Retry occurs on next interval

**Pass Criteria:** Dashboard handles timeouts without crashing

---

### 8.5 Performance Test Cases

**TC-PERF-001: Lambda Function Execution Time**

**Test Case ID:** TC-PERF-001  
**Test Level:** Performance  
**Priority:** Medium

**Objective:** Verify Lambda function execution completes within timeout limits.

**Test Steps:**

1. Trigger Lambda function with sensor data
2. Measure execution time
3. Verify execution completes within 30-second timeout
4. Verify execution time is reasonable (< 10 seconds for normal operation)

**Expected Results:**

- Execution completes within timeout
- Average execution time < 10 seconds
- No timeout errors

**Pass Criteria:** Lambda functions execute within performance requirements

---

**TC-PERF-002: API Response Time**

**Test Case ID:** TC-PERF-002  
**Test Level:** Performance  
**Priority:** Medium

**Objective:** Verify API endpoints respond within acceptable time limits.

**Test Steps:**

1. Send API requests to all endpoints
2. Measure response times
3. Verify responses complete within 5 seconds
4. Verify response times are consistent

**Expected Results:**

- All endpoints respond within 5 seconds
- Response times are consistent
- No performance degradation under normal load

**Pass Criteria:** API endpoints meet performance requirements

---

### 8.6 Test Data Management

**Test Data Sets:**

**Valid Sensor Data:**

```json
{
  "deviceId": "esp32-test-01",
  "temperature": 25.5,
  "humidity": 45.2,
  "lat": 43.467,
  "lng": -79.699,
  "timestamp": "2025-12-01T16:20:00Z"
}
```

**Invalid Sensor Data (Missing Temperature):**

```json
{
  "deviceId": "esp32-test-01",
  "temperature": null,
  "humidity": 45.2,
  "lat": 43.467,
  "lng": -79.699,
  "timestamp": "2025-12-01T16:20:00Z"
}
```

**Invalid Sensor Data (Out of Range):**

```json
{
  "deviceId": "esp32-test-01",
  "temperature": 150.0,
  "humidity": 150.0,
  "lat": 43.467,
  "lng": -79.699,
  "timestamp": "2025-12-01T16:20:00Z"
}
```

**Edge Case Data:**

- Maximum temperature: 50.0°C
- Minimum temperature: 0.0°C
- Maximum humidity: 100%
- Minimum humidity: 0%
- Maximum fire distance: 100km
- Minimum fire distance: 0km

---

### 8.7 Test Coverage

**Functional Coverage:**

- All use cases have corresponding test cases
- All functional areas (FA1, FA2, FA3) are covered
- All API endpoints are tested
- Error handling scenarios are covered

**Code Coverage Targets:**

- Lambda functions: Minimum 80% code coverage
- Critical paths: 100% coverage
- Error handling: 100% coverage

**Integration Coverage:**

- IoT Core → Lambda integration tested
- Lambda → DynamoDB integration tested
- API Gateway → Lambda integration tested
- Frontend → API Gateway integration tested

---

### 8.8 Test Execution Environment

**Local Development Testing:**

- Docker Compose environment with DynamoDB Local
- Local Flask API server
- Mock sensor for IoT testing
- Local MQTT broker (Mosquitto)

**AWS Cloud Testing:**

- Deployed Lambda functions
- AWS IoT Core with test devices
- DynamoDB table in AWS
- API Gateway REST API
- CloudWatch Logs for monitoring

**Test Tools:**

- Python pytest for unit testing
- curl for API endpoint testing
- AWS CLI for cloud resource verification
- Browser DevTools for frontend testing
- CloudWatch Logs for Lambda function monitoring

---

### 8.9 Test Status and Results

**Test Execution Status:**

**Functional Area 1 (FA1) Tests:**

- TC-FA1-001: Valid Sensor Data Ingestion - Status: Pass
- TC-FA1-002: Invalid Sensor Data Rejection - Status: Pass
- TC-FA1-003: Network Connectivity Loss Handling - Status: Pass

**Functional Area 2 (FA2) Tests:**

- TC-FA2-001: Successful NASA FIRMS API Integration - Status: Pass
- TC-FA2-002: NASA FIRMS API Unavailability Handling - Status: Pass
- TC-FA2-003: No Active Fires Detected - Status: Pass

**Functional Area 3 (FA3) Tests:**

- TC-FA3-001: Risk Score Calculation - Status: Pass
- TC-FA3-002: Risk Score Edge Cases - Status: Pass
- TC-FA3-003: Dashboard Data Retrieval - Status: Pass
- TC-FA3-004: Risk Map Data Retrieval - Status: Pass
- TC-FA3-005: Dashboard Polling Mechanism - Status: Pass

**API Endpoint Tests:**

- TC-API-001: Get All Sensors Endpoint - Status: Pass
- TC-API-002: Get Sensor by ID Endpoint - Status: Pass
- TC-API-003: Get Risk Map Data Endpoint - Status: Pass

**Error Handling Tests:**

- TC-ERROR-001: DynamoDB Write Failure - Status: Pass
- TC-ERROR-002: API Gateway Timeout - Status: Pass

**Performance Tests:**

- TC-PERF-001: Lambda Function Execution Time - Status: Pass
- TC-PERF-002: API Response Time - Status: Pass

**Overall Test Status:** All test cases passing

---

## Document Version Information

**Document Version:** 1.0  
**Last Updated:** Current Semester  
**Project:** Capstone - Hackathon Profile  
**Organization:** Project GreenGuard  
**Service:** ForestShield  
**Model Standard:** Sheridan Capstone Visual Paradigm Requirements
