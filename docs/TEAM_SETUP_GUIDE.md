# ForestShield - Team Setup Guide

## Purpose

This guide helps team members set up their local development environment and configure necessary credentials for working with the ForestShield project.

## Prerequisites

- Git installed
- Docker and Docker Compose installed
- Node.js >= 16 installed
- Python 3.11+ installed (optional, for local testing)
- AWS CLI installed (for infrastructure deployment, when ready)

## Initial Setup

### 1. Clone All Repositories

```bash
# Create a workspace directory
mkdir forestshield-workspace
cd forestshield-workspace

# Clone all repositories
git clone https://github.com/Project-GreenGuard/forestshield-iot-firmware.git
git clone https://github.com/Project-GreenGuard/forestshield-backend.git
git clone https://github.com/Project-GreenGuard/forestshield-frontend.git
git clone https://github.com/Project-GreenGuard/forestshield-infrastructure.git
```

### 2. Frontend Setup

```bash
cd forestshield-frontend

# Copy environment template
cp .env.example .env

# Edit .env file (already configured for local development)
# For local: REACT_APP_API_URL=http://localhost:5001/api
# For production: Update with API Gateway URL (provided by team lead)

# Install dependencies
npm install
```

### 3. Backend Setup

```bash
cd ../forestshield-backend

# Copy environment template
cp .env.example .env

# Edit .env file (default values work for local development)
# No changes needed unless you want to customize ports or region

# Start local services
docker-compose up -d

# Verify services are running
docker-compose ps
```

### 4. Verify Local Setup

```bash
# Test backend API
curl http://localhost:5001/health

# Start frontend
cd ../forestshield-frontend
npm start
```

Frontend should open at http://localhost:3000

## Environment Variables Reference

### Frontend (.env)

| Variable            | Description          | Local Value                 | Production Value           |
| ------------------- | -------------------- | --------------------------- | -------------------------- |
| `REACT_APP_API_URL` | Backend API endpoint | `http://localhost:5001/api` | API Gateway URL (provided) |

### Backend (.env)

| Variable                | Description            | Local Value            | Production Value   |
| ----------------------- | ---------------------- | ---------------------- | ------------------ |
| `FLASK_ENV`             | Flask environment mode | `development`          | N/A (Docker only)  |
| `AWS_ENDPOINT_URL`      | DynamoDB endpoint      | `http://dynamodb:8000` | Not set (uses AWS) |
| `AWS_ACCESS_KEY_ID`     | AWS access key         | `local`                | Not set (uses IAM) |
| `AWS_SECRET_ACCESS_KEY` | AWS secret key         | `local`                | Not set (uses IAM) |
| `AWS_REGION`            | AWS region             | `us-east-1`            | `us-east-1`        |

## Credentials and Keys

### Local Development

**No credentials required** for local development. The backend uses:

- DynamoDB Local (no authentication)
- Mock AWS credentials (`local`/`local`)
- Local MQTT broker (no authentication)

### AWS Infrastructure (When Ready)

When infrastructure is deployed, you will need:

#### 1. AWS Credentials

**Who provides:** Team lead or AWS administrator

**What you need:**

- AWS Access Key ID
- AWS Secret Access Key
- AWS Region (default: `us-east-1`)

**How to configure:**

```bash
aws configure --profile hackathon
# Enter Access Key ID
# Enter Secret Access Key
# Enter region: us-east-1
# Enter output format: json
```

#### 2. API Gateway URL

**Who provides:** Team lead (after Terraform deployment)

**What you need:**

- API Gateway endpoint URL (e.g., `https://xxxxx.execute-api.us-east-1.amazonaws.com`)

**Where to use:**

- Update `forestshield-frontend/.env`:
  ```
  REACT_APP_API_URL=https://xxxxx.execute-api.us-east-1.amazonaws.com/api
  ```

#### 3. IoT Core Endpoint (For Firmware)

**Who provides:** Team lead (after Terraform deployment)

**What you need:**

- AWS IoT Core endpoint address
- Device certificates (for ESP32)

**Where to use:**

- Update `forestshield-iot-firmware/esp32_wildfire_sensor.ino` with endpoint and certificates

### NASA FIRMS API

**No credentials required.** The NASA FIRMS API is public and does not require authentication.

## Security Best Practices

1. **Never commit `.env` files** - They are in `.gitignore`
2. **Share credentials securely** - Use secure channels (not GitHub, not email)
3. **Use AWS IAM roles** - In production, use IAM roles instead of access keys when possible
4. **Rotate credentials** - If credentials are compromised, rotate them immediately
5. **Use separate AWS accounts** - Consider separate accounts for development and production

## Troubleshooting

### Backend Services Not Starting

```bash
# Check if ports are in use
lsof -i :5001
lsof -i :8000
lsof -i :1883

# View Docker logs
cd forestshield-backend
docker-compose logs -f
```

### Frontend Can't Connect to API

1. Verify backend is running: `curl http://localhost:5001/health`
2. Check `.env` file exists and has correct URL
3. Check browser console for CORS errors
4. Verify no firewall blocking localhost connections

### Missing Environment Variables

If you see errors about missing environment variables:

1. Ensure `.env` file exists (copy from `.env.example`)
2. Restart Docker Compose: `docker-compose down && docker-compose up -d`
3. Restart frontend: Stop and run `npm start` again

## Getting Help

1. Check repository README files for component-specific documentation
2. Review `QUICK_START.md` in `/docs` folder
3. Contact team lead for AWS credentials and infrastructure details
4. Check GitHub Issues for known problems

## Next Steps

After local setup is complete:

1. **Review Documentation:**

   - Read `PROJECT_OVERVIEW.md` for system architecture
   - Read `DEVELOPMENT_GUIDE.md` for development workflow
   - Read component-specific README files

2. **Start Developing:**

   - Pick a task from the project board
   - Create a feature branch
   - Follow Git workflow guidelines

3. **Test Your Changes:**
   - Run local tests
   - Verify changes work in local environment
   - Test with sample data

---

**Document Version:** 1.0  
**Last Updated:** Current Semester  
**Project:** Project GreenGuard - ForestShield
