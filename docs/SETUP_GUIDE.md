# ForestShield - Complete Setup Guide

## Prerequisites

### Required Software
- **AWS CLI**: Configured with hackathon profile
- **Terraform**: >= 1.0
- **Docker & Docker Compose**: For local development
- **Node.js**: >= 16 (for frontend)
- **Arduino IDE**: For ESP32 firmware
- **Git**: For version control

### AWS Setup
```bash
# Configure AWS CLI with hackathon profile
aws configure --profile hackathon

# Verify
aws sts get-caller-identity --profile hackathon
```

## Step 1: Create GitHub Repositories

Create these repositories in **Project GreenGuard** organization:

1. `forestshield-iot-firmware`
2. `forestshield-backend`
3. `forestshield-frontend`
4. `forestshield-infrastructure`
5. `forestshield` (optional - docs)

**GitHub**: https://github.com/organizations/Project-GreenGuard/repositories/new

## Step 2: Set Up Local Workspace

```bash
# Create project folder
mkdir -p ~/Documents/Projects/Project-GreenGuard
cd ~/Documents/Projects/Project-GreenGuard

# Clone repositories (after creating on GitHub)
git clone https://github.com/Project-GreenGuard/forestshield-iot-firmware.git
git clone https://github.com/Project-GreenGuard/forestshield-backend.git
git clone https://github.com/Project-GreenGuard/forestshield-frontend.git
git clone https://github.com/Project-GreenGuard/forestshield-infrastructure.git
git clone https://github.com/Project-GreenGuard/forestshield.git
```

## Step 3: Deploy Infrastructure

### 3.1 Package Lambda Functions

```bash
cd forestshield-backend

# Package process_sensor_data Lambda
cd lambda-processing
zip -r ../../forestshield-infrastructure/lambda-processing.zip . \
  -x "*.pyc" "__pycache__/*" "*.zip"

# Package api_handler Lambda
cd ../api-gateway-lambda
zip -r ../../forestshield-infrastructure/api-gateway-lambda.zip . \
  -x "*.pyc" "__pycache__/*" "*.zip"
```

### 3.2 Deploy with Terraform

```bash
cd ../forestshield-infrastructure

# Initialize Terraform
terraform init

# Review plan
terraform plan

# Deploy
terraform apply
```

**Note outputs:**
- `iot_endpoint_address` - For ESP32 configuration
- `api_endpoint` - For frontend configuration
- `dynamodb_table_name` - Table name

## Step 4: Configure IoT Firmware

### 4.1 Get IoT Core Endpoint

From Terraform output:
```bash
cd forestshield-infrastructure
terraform output iot_endpoint_address
```

### 4.2 Set Up Device in AWS IoT Core

1. Go to AWS IoT Core Console
2. Create Thing: `esp32-01`
3. Create certificate and keys
4. Download:
   - Device certificate
   - Private key
   - Root CA (Amazon Root CA 1)

### 4.3 Configure ESP32 Firmware

```bash
cd forestshield-iot-firmware
```

Edit `esp32_wildfire_sensor.ino`:

1. **WiFi Credentials:**
   ```cpp
   const char* ssid = "YOUR_WIFI_SSID";
   const char* password = "YOUR_WIFI_PASSWORD";
   ```

2. **AWS IoT Core:**
   ```cpp
   const char* aws_iot_endpoint = "YOUR_ENDPOINT.iot.us-east-1.amazonaws.com";
   ```

3. **Device Info:**
   ```cpp
   const char* deviceId = "esp32-01";  // Must match IoT Thing name
   const float sensorLat = 43.467;     // Your GPS coordinates
   const float sensorLng = -79.699;
   ```

4. **Certificates:**
   - Replace `device_cert` with your certificate
   - Replace `device_key` with your private key
   - Update `root_ca` if needed

### 4.4 Upload to ESP32

1. Open `esp32_wildfire_sensor.ino` in Arduino IDE
2. Install libraries:
   - ArduinoJson
   - DHT sensor library
   - PubSubClient
3. Select board: **ESP32 Dev Module**
4. Select port
5. Upload

## Step 5: Local Backend Development

```bash
cd forestshield-backend

# Start local services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down
```

**Services:**
- API: http://localhost:5000
- DynamoDB Local: http://localhost:8000
- MQTT: mqtt://localhost:1883

## Step 6: Frontend Development

```bash
cd forestshield-frontend

# Install dependencies
npm install

# Create .env file
echo "REACT_APP_API_URL=http://localhost:5000/api" > .env

# Start development server
npm start
```

Frontend runs at: http://localhost:3000

## Step 7: Testing

### Test IoT Sensor

Monitor serial output (115200 baud):
- WiFi connection
- AWS IoT Core connection
- Sensor readings
- MQTT publishing

### Test Backend API

```bash
# Get all sensors
curl http://localhost:5000/api/sensors

# Get specific sensor
curl http://localhost:5000/api/sensor/esp32-01

# Get risk map
curl http://localhost:5000/api/risk-map
```

### Test Frontend

1. Open http://localhost:3000
2. Verify map loads
3. Check sensor data displays
4. Test API integration

## Step 8: Production Configuration

### Frontend

Update `.env`:
```env
REACT_APP_API_URL=https://YOUR_API_GATEWAY_URL/api
```

Build:
```bash
npm run build
```

### Backend

Lambda functions are deployed via Terraform.

## Troubleshooting

### IoT Connection Issues
- Verify WiFi credentials
- Check AWS IoT Core endpoint
- Ensure certificates are correctly formatted
- Monitor serial output

### Lambda Errors
- Check CloudWatch logs
- Verify DynamoDB table exists
- Ensure IAM permissions are correct

### API Gateway Issues
- Verify Lambda function names match Terraform
- Check CORS headers
- Review API Gateway logs

### Terraform Issues
- Ensure Lambda zip files are in `forestshield-infrastructure/`
- Verify AWS credentials
- Check IAM permissions

## Next Steps

1. Set up branch protection on GitHub
2. Configure team access
3. Set up CI/CD (GitHub Actions)
4. Continue development on individual components

---

**Setup Version**: 1.0  
**Last Updated**: Current Semester

