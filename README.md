# ForestShield - AI-Driven Wildfire Response System

**Project GreenGuard: ForestShield**

A comprehensive system that collects real-time IoT sensor data from ESP32 devices, integrates NASA FIRMS wildfire data, performs risk scoring, and visualizes everything through a React Progressive Web Application dashboard.

## Project Overview

ForestShield implements three core functional areas:

1. **IoT Sensor Ingestion** - ESP32 + DHT11 → AWS IoT Core → Lambda → DynamoDB
2. **External Wildfire Data Integration** - NASA FIRMS API integration
3. **Risk Scoring + Dashboard** - Real-time risk calculation and visualization

## Repository Structure

This project is organized as microservices with separate repositories:

### Core Repositories

- **[forestshield-iot-firmware](https://github.com/Project-GreenGuard/forestshield-iot-firmware)** - ESP32 Arduino firmware
- **[forestshield-backend](https://github.com/Project-GreenGuard/forestshield-backend)** - AWS Lambda functions and backend services
- **[forestshield-frontend](https://github.com/Project-GreenGuard/forestshield-frontend)** - React PWA dashboard
- **[forestshield-infrastructure](https://github.com/Project-GreenGuard/forestshield-infrastructure)** - Terraform infrastructure as code

## Architecture

```
ESP32 → MQTT → AWS IoT Core → IoT Rule → Lambda (Processing)
                                              ↓
                                         NASA FIRMS API
                                              ↓
                                         Risk Calculation
                                              ↓
                                         DynamoDB
                                              ↓
API Gateway ← Lambda (API Handler) ← Frontend Dashboard
```

## Quick Start

### 1. Infrastructure Setup

```bash
cd forestshield-infrastructure
terraform init
terraform apply
```

### 2. Configure IoT Devices

```bash
cd forestshield-iot-firmware
# Update WiFi, AWS IoT endpoint, certificates
# Upload to ESP32
```

### 3. Local Backend Development

```bash
cd forestshield-backend
docker-compose up
```

### 4. Frontend Development

```bash
cd forestshield-frontend
npm install
npm start
```

## Data Flow

1. ESP32 sensors read temperature and humidity
2. MQTT messages sent to AWS IoT Core
3. IoT Rule triggers Lambda function
4. Lambda fetches NASA FIRMS data and calculates risk
5. DynamoDB stores enriched data
6. API Gateway serves data to frontend
7. React dashboard visualizes everything

## Risk Scoring

Simple rule-based algorithm:

```
riskScore = w1 * temp_score + w2 * humidity_score + w3 * fire_score
```

Where:

- `temp_score` = normalized temperature (0-50°C → 0-100)
- `humidity_score` = inverse humidity (lower = higher risk)
- `fire_score` = inverse distance to nearest fire (0-100km → 0-100)
- Weights: w1=0.4, w2=0.3, w3=0.3

## Tech Stack

- **IoT**: ESP32, DHT11, Arduino C++, MQTT
- **Cloud**: AWS IoT Core, Lambda, DynamoDB, API Gateway
- **Frontend**: React, Leaflet, TailwindCSS
- **Infrastructure**: Terraform
- **Local Dev**: Docker, DynamoDB Local, Mosquitto

## Current Semester Scope

- Real IoT ingestion pipeline
- NASA FIRMS wildfire data integration
- Risk scoring + dashboard visualization
- Lightweight AWS infrastructure ($100 budget)

## Next Semester

- GCP Vertex AI integration
- Advanced ML models
- Alerting system (SNS)
- Security hardening

## Team

Project GreenGuard - Capstone Project

## License

See LICENSE file

## Links

- [IoT Firmware Documentation](https://github.com/Project-GreenGuard/forestshield-iot-firmware)
- [Backend Documentation](https://github.com/Project-GreenGuard/forestshield-backend)
- [Frontend Documentation](https://github.com/Project-GreenGuard/forestshield-frontend)
- [Infrastructure Documentation](https://github.com/Project-GreenGuard/forestshield-infrastructure)
