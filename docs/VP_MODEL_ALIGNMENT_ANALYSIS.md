# ForestShield - VP Software Model Alignment Analysis

## Document Purpose

This document identifies discrepancies and missing information between the Visual Paradigm Software Model (VP_SOFTWARE_MODEL.md) and the actual implementation, ensuring documentation consistency across all project files.

**Last Updated:** Current Semester  
**Analysis Date:** Current Review

---

## 1.0 Field Name Inconsistencies

### 1.1 DynamoDB Field: Fire Distance

**Issue:** Inconsistent field naming for nearest fire distance across documentation.

**Current State:**
- **VP_SOFTWARE_MODEL.md:** Uses `nearestFireKm` in use case descriptions
- **PROJECT_OVERVIEW.md:** Uses `nearestFireKm` in data schema
- **ARCHITECTURE.md:** Uses `nearestFireDistance` in DynamoDB item example
- **API_DOCUMENTATION.md:** Uses `nearestFireDistance` in data models
- **Actual Implementation:** Uses `nearestFireDistance` in `process_sensor_data.py`

**Resolution Required:**
- Update VP_SOFTWARE_MODEL.md to use `nearestFireDistance` (matches implementation)
- Update PROJECT_OVERVIEW.md to use `nearestFireDistance` (matches implementation)

**Recommended Change:**
Replace all instances of `nearestFireKm` with `nearestFireDistance` in:
- VP_SOFTWARE_MODEL.md (lines 226, 233, 240, 246)
- PROJECT_OVERVIEW.md (lines 178, 192, 514)

---

## 2.0 NASA FIRMS API Endpoint Discrepancy

### 2.1 API Endpoint URL Mismatch

**Issue:** VP_SOFTWARE_MODEL.md and PROJECT_OVERVIEW.md document a different NASA FIRMS API endpoint than what is actually implemented.

**Documented Endpoints:**
- **VP_SOFTWARE_MODEL.md (line 1407):** `https://firms.modaps.eosdis.nasa.gov/api/viirs/active_fires`
- **PROJECT_OVERVIEW.md (line 488):** `https://firms.modaps.eosdis.nasa.gov/api/viirs/active_fires?bbox=...&date=...&format=json`

**Actual Implementation:**
- **process_sensor_data.py (line 22):** `https://firms.modaps.eosdis.nasa.gov/api/country/csv/{}/MODIS_NRT/1`
- Uses country code parameter (e.g., "CAN") instead of bounding box
- Returns CSV format, not JSON
- Uses MODIS_NRT dataset, not VIIRS

**Resolution Required:**
- Update VP_SOFTWARE_MODEL.md to reflect actual API endpoint and format
- Update PROJECT_OVERVIEW.md to reflect actual API endpoint and format
- Document the actual API integration approach (country-based CSV endpoint)

**Recommended Changes:**

1. **VP_SOFTWARE_MODEL.md Section 14.1:**
   - Update base URL to: `https://firms.modaps.eosdis.nasa.gov/api/country/csv/{country_code}/MODIS_NRT/1`
   - Update request parameters to: `country_code` (e.g., "CAN")
   - Update response format to: CSV (not JSON)
   - Update dataset to: MODIS_NRT (not VIIRS)

2. **PROJECT_OVERVIEW.md Section 14.1:**
   - Update API specification to match actual implementation
   - Update example request to show country code parameter

---

## 3.0 Lambda Function Configuration Discrepancies

### 3.1 Memory Allocation

**Issue:** VP_SOFTWARE_MODEL.md specifies different memory sizes than Terraform configuration.

**VP_SOFTWARE_MODEL.md (line 1300):**
- `wildfire-process-sensor-data`: 256 MB
- `wildfire-api-handler`: 128 MB

**Actual Terraform (lambda.tf):**
- `wildfire-process-sensor-data`: 256 MB
- `wildfire-api-handler`: 256 MB

**Resolution Required:**
- Update VP_SOFTWARE_MODEL.md to reflect 256 MB for both functions

---

## 4.0 Design Model vs Implementation Structure

### 4.1 Class-Based Design vs Functional Implementation

**Issue:** VP_SOFTWARE_MODEL.md describes the system using formal class diagrams with Boundary/Control/Entity classes, but the actual implementation uses functional/procedural Python code.

**VP_SOFTWARE_MODEL.md Describes:**
- `IoTMessageReceiver` (Boundary class)
- `SensorDataProcessor` (Control class)
- `NASAFIRMSAPIClient` (Boundary class)
- `FireDataIntegrator` (Control class)
- `RiskScoreCalculator` (Control class)
- `APIRequestHandler` (Boundary class)
- `SensorDataController` (Control class)

**Actual Implementation:**
- Functional Python code with functions:
  - `lambda_handler()` - Main entry point
  - `fetch_nasa_firms_data()` - API client function
  - `find_nearest_fire()` - Fire proximity calculation
  - `calculate_risk_score()` - Risk calculation
  - `calculate_distance()` - Distance calculation

**Resolution Required:**
- Add note in VP_SOFTWARE_MODEL.md Section 3.3 explaining that the class diagrams represent logical design structure, while implementation uses functional programming paradigm
- This is acceptable for a design model, but should be explicitly documented

**Recommended Addition to VP_SOFTWARE_MODEL.md:**

Add after Section 3.3 Subsystem Class Diagrams:

```
### 3.3.1 Design Model Implementation Note

The subsystem class diagrams represent logical design structure using Boundary-Control-Entity (BCE) pattern. The actual implementation uses functional/procedural programming paradigm in Python Lambda functions rather than object-oriented classes. The logical responsibilities described in the class diagrams are realized through function-based implementations:

- Boundary classes map to input/output handling functions
- Control classes map to workflow coordination functions
- Entity classes map to data structures (dictionaries, objects)

This design-to-implementation mapping maintains the same logical separation of concerns while using a functional programming approach suitable for serverless Lambda functions.
```

---

## 5.0 Missing Information in VP_SOFTWARE_MODEL.md

### 5.1 Lambda Function Runtime Version

**Issue:** VP_SOFTWARE_MODEL.md specifies "Python 3.9+" but Terraform uses "python3.11".

**VP_SOFTWARE_MODEL.md (line 1272):**
- Runtime: "AWS Lambda runtime (Python 3.9+)"

**Actual Terraform (lambda.tf):**
- Runtime: "python3.11"

**Resolution Required:**
- Update VP_SOFTWARE_MODEL.md to specify "Python 3.11" (matches implementation)

---

### 5.2 DynamoDB Table Name Consistency

**Verification:** All documents consistently use `WildfireSensorData` - ✓ No change needed.

---

### 5.3 API Gateway Endpoint Paths

**Verification:** All documents consistently use:
- `/api/sensors`
- `/api/sensor/{id}`
- `/api/risk-map`

✓ No change needed.

---

### 5.4 Lambda Function Names

**Verification:** All documents consistently use:
- `wildfire-process-sensor-data`
- `wildfire-api-handler`

✓ No change needed.

---

## 6.0 Sequence Diagram Accuracy

### 6.1 Sequence Diagram Participants

**Verification:** Sequence diagrams in VP_SOFTWARE_MODEL.md reference class names that represent logical design components. These align with the functional implementation responsibilities:

- `IoTMessageReceiver` → `lambda_handler()` event parsing
- `SensorDataProcessor` → `lambda_handler()` workflow
- `NASAFIRMSAPIClient` → `fetch_nasa_firms_data()` function
- `FireDataIntegrator` → `find_nearest_fire()` function
- `RiskScoreCalculator` → `calculate_risk_score()` function

**Resolution:** Add note explaining that sequence diagram participants represent logical components, not actual class names.

---

## 7.0 Data Schema Consistency

### 7.1 DynamoDB Item Structure

**VP_SOFTWARE_MODEL.md vs Implementation:**

**VP Model Describes:**
- `nearestFireKm` (should be `nearestFireDistance`)
- `nearestFireData` (JSON string)

**Implementation Uses:**
- `nearestFireDistance` (Number or -1 if no fire)
- `nearestFireData` (JSON string or None)

**Resolution Required:**
- Update VP_SOFTWARE_MODEL.md to use `nearestFireDistance`
- Document that -1 is used as default when no fire is detected

---

## 8.0 Summary of Required Updates

### 8.1 High Priority Updates

1. **Field Name Consistency:**
   - Update `nearestFireKm` → `nearestFireDistance` in VP_SOFTWARE_MODEL.md and PROJECT_OVERVIEW.md

2. **NASA FIRMS API Documentation:**
   - Update API endpoint documentation to reflect actual CSV country-based endpoint
   - Update request/response format documentation

3. **Lambda Memory Configuration:**
   - Update api_handler memory from 128 MB to 256 MB in VP_SOFTWARE_MODEL.md

4. **Python Runtime Version:**
   - Update from "Python 3.9+" to "Python 3.11" in VP_SOFTWARE_MODEL.md

### 8.2 Medium Priority Updates

5. **Design Model Implementation Note:**
   - Add explanation of class-based design model vs functional implementation

6. **Sequence Diagram Note:**
   - Add note explaining logical component names in sequence diagrams

### 8.3 Low Priority (Documentation Enhancement)

7. **Default Value Documentation:**
   - Document that `nearestFireDistance` uses -1 as default when no fire detected

---

## 9.0 Verification Checklist

After making updates, verify:

- [ ] All field names match implementation (`nearestFireDistance`)
- [ ] NASA FIRMS API endpoint matches actual code
- [ ] Lambda memory sizes match Terraform configuration
- [ ] Python runtime version matches Terraform
- [ ] Design model notes explain functional implementation approach
- [ ] Sequence diagrams have notes about logical component names
- [ ] All DynamoDB field names are consistent across all documents
- [ ] API endpoint paths are consistent
- [ ] Lambda function names are consistent

---

## 10.0 Document Cross-Reference

**Documents Reviewed:**
- VP_SOFTWARE_MODEL.md
- PROJECT_OVERVIEW.md
- ARCHITECTURE.md
- API_DOCUMENTATION.md
- DEVELOPMENT_GUIDE.md
- SETUP_GUIDE.md

**Implementation Files Reviewed:**
- `forestshield-backend/lambda-processing/process_sensor_data.py`
- `forestshield-backend/api-gateway-lambda/api_handler.py`
- `forestshield-infrastructure/lambda.tf`
- `forestshield-infrastructure/dynamodb.tf`

---

**Analysis Version:** 1.0  
**Last Updated:** Current Semester  
**Project:** Project GreenGuard - ForestShield

