# ForestShield - Development Guide

## Development Workflow

### Repository Structure

Each repository is independent and can be developed separately:

```
Project-GreenGuard/
├── forestshield-iot-firmware/      # ESP32 development
├── forestshield-backend/          # Lambda/API development
├── forestshield-frontend/          # React development
└── forestshield-infrastructure/     # Terraform/IaC
```

## IoT Firmware Development

### Setup

```bash
cd forestshield-iot-firmware
```

### Required Libraries

Install via Arduino Library Manager:
- **ArduinoJson** (v6.x+)
- **DHT sensor library** (Adafruit)
- **Adafruit Unified Sensor** (dependency)
- **PubSubClient** (v2.8.x+)

### Development Process

1. **Edit firmware** (`esp32_wildfire_sensor.ino`)
2. **Upload to ESP32** via Arduino IDE
3. **Monitor serial output** (115200 baud)
4. **Test MQTT publishing** to AWS IoT Core

### Testing Without Hardware

Use mock sensor:
```bash
python mock_sensor.py
```

### Key Files
- `esp32_wildfire_sensor.ino` - Main firmware
- `mock_sensor.py` - Python mock for testing

## Backend Development

### Local Development

```bash
cd forestshield-backend

# Start Docker services
docker-compose up -d

# View logs
docker-compose logs -f api
```

### Lambda Functions

#### Process Sensor Data

**Location:** `lambda-processing/process_sensor_data.py`

**Local Testing:**
```python
# Test payload
event = {
    "deviceId": "esp32-01",
    "temperature": 23.4,
    "humidity": 40.2,
    "lat": 43.467,
    "lng": -79.699,
    "timestamp": "2025-12-01T16:20:00Z"
}

# Run locally (requires boto3, requests)
python process_sensor_data.py
```

**Deployment:**
```bash
cd lambda-processing
zip -r ../lambda-processing.zip . -x "*.pyc" "__pycache__/*"
# Then deploy via Terraform
```

#### API Handler

**Location:** `api-gateway-lambda/api_handler.py`

**Local Testing:**
```bash
# Use local Flask server
cd api-gateway-lambda
python local_dashboard.py

# Test endpoints
curl http://localhost:5000/api/sensors
```

**Deployment:**
```bash
cd api-gateway-lambda
zip -r ../api-gateway-lambda.zip . -x "*.pyc" "__pycache__/*"
# Then deploy via Terraform
```

### Environment Variables

**Local:**
```env
AWS_ENDPOINT_URL=http://dynamodb:8000
AWS_ACCESS_KEY_ID=local
AWS_SECRET_ACCESS_KEY=local
```

**AWS (via Terraform):**
- `DYNAMODB_TABLE` - Set automatically

## Frontend Development

### Setup

```bash
cd forestshield-frontend

# Install dependencies
npm install

# Create .env file
echo "REACT_APP_API_URL=http://localhost:5000/api" > .env

# Start development server
npm start
```

### Development Server

- **URL**: http://localhost:3000
- **Hot Reload**: Enabled
- **API Proxy**: Configure via `.env`

### Key Components

- `src/components/MapArea.js` - Leaflet map
- `src/components/DataPanel.js` - Sensor data display
- `src/components/Sidebar.js` - Navigation
- `src/components/Topbar.js` - Header

### Adding Features

1. **New Component:**
   ```bash
   # Create component file
   touch src/components/NewComponent.js
   ```

2. **Update API calls:**
   - Edit `src/App.js` or component files
   - Use `fetch()` or `axios` for API calls

3. **Styling:**
   - Currently using CSS modules
   - TailwindCSS to be added

### Building for Production

```bash
npm run build
```

Output: `build/` directory

## Infrastructure Development

### Terraform Workflow

```bash
cd forestshield-infrastructure

# Initialize
terraform init

# Plan changes
terraform plan

# Apply changes
terraform apply

# Destroy (careful!)
terraform destroy
```

### Adding New Resources

1. **Create Terraform file:**
   ```bash
   touch new-resource.tf
   ```

2. **Define resource:**
   ```hcl
   resource "aws_service" "example" {
     # configuration
   }
   ```

3. **Plan and apply:**
   ```bash
   terraform plan
   terraform apply
   ```

### Updating Lambda Functions

1. **Package Lambda:**
   ```bash
   cd ../forestshield-backend/lambda-processing
   zip -r ../../forestshield-infrastructure/lambda-processing.zip .
   ```

2. **Update Terraform:**
   ```bash
   cd ../forestshield-infrastructure
   terraform apply
   ```

## Testing

### Unit Tests

**Backend (Python):**
```bash
cd forestshield-backend
python -m pytest tests/
```

**Frontend (Jest):**
```bash
cd forestshield-frontend
npm test
```

### Integration Tests

**API Testing:**
```bash
# Test local API
curl http://localhost:5000/api/sensors

# Test production API
curl https://YOUR_API_GATEWAY_URL/api/sensors
```

**IoT Testing:**
- Monitor serial output
- Check AWS IoT Core console
- Verify DynamoDB entries

## Git Workflow

### Branch Strategy

- `main` - Production-ready code
- `develop` - Development branch (optional)
- `feature/feature-name` - Feature branches

### Commit Messages

Follow conventional commits:
```
feat: add new sensor endpoint
fix: resolve DynamoDB query issue
docs: update API documentation
```

### Pull Requests

1. Create feature branch
2. Make changes
3. Push to GitHub
4. Create PR
5. Review and merge

## Code Style

### Python

- Follow PEP 8
- Use type hints where possible
- Docstrings for functions

### JavaScript/React

- Use ES6+ syntax
- Functional components
- Hooks for state management

### Terraform

- Use consistent formatting
- Add comments for complex resources
- Use variables for configuration

## Debugging

### Lambda Functions

**CloudWatch Logs:**
```bash
aws logs tail /aws/lambda/wildfire-process-sensor-data --follow --profile hackathon
```

### Frontend

**Browser DevTools:**
- Console for errors
- Network tab for API calls
- React DevTools extension

### IoT

**Serial Monitor:**
- Arduino IDE Serial Monitor
- Baud rate: 115200
- Check for connection errors

## Performance Optimization

### Lambda

- Minimize dependencies
- Use connection pooling
- Optimize DynamoDB queries

### Frontend

- Code splitting
- Lazy loading
- Optimize images
- Use React.memo for expensive components

### DynamoDB

- Use appropriate indexes
- Batch operations where possible
- Set TTL for automatic cleanup

## Security Best Practices

### Current

- AWS IoT Core certificates
- IAM roles with minimal permissions
- CORS configuration

### Future

- API authentication (API keys, Cognito)
- WAF rules
- Encryption at rest
- Secrets management (AWS Secrets Manager)

---

**Development Guide Version**: 1.0  
**Last Updated**: Current Semester

