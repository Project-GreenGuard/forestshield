# ForestShield - Documentation Formatting Guide

## Purpose

This guide establishes the formatting standards for all documentation within the ForestShield project. All documentation must adhere to these standards to maintain consistency, professionalism, and academic rigor appropriate for capstone project deliverables.

## General Principles

### Tone and Style

- **Professional and Academic**: Use formal, technical language appropriate for engineering documentation
- **Clear and Concise**: Communicate information directly without unnecessary verbosity
- **Consistent Terminology**: Use consistent technical terms throughout all documents
- **Objective Language**: Avoid subjective statements and marketing language

### Prohibited Elements

- **No Emojis**: Do not use emojis, icons, or decorative symbols in documentation
- **No Informal Language**: Avoid casual expressions, slang, or conversational tone
- **No Marketing Language**: Avoid promotional or sales-oriented language
- **No Exaggeration**: Use accurate, measured descriptions

## Document Structure

### Section Numbering

Use hierarchical section numbering for formal documentation:

- Major sections: `1.0`, `2.0`, `3.0`
- Subsections: `1.1`, `1.2`, `2.1`, `2.2`
- Sub-subsections: `1.1.1`, `1.1.2`, `2.1.1`

**Example:**

```markdown
## 1.0 Project Description

## 2.0 Functional Areas

### 2.1 Current Semester Implementation

### 2.2 Deferred to Next Semester
```

### Section Headers

Use descriptive, action-oriented section headers:

- ✅ **Good**: "System Architecture", "Data Flow Sequence", "Risk Score Calculation"
- ❌ **Avoid**: "Stuff", "More Info", "Things to Know"

### Lists and Bullet Points

Use consistent list formatting:

- Start with capital letters
- Use parallel structure across items
- End items without punctuation (unless complete sentences)
- Use numbered lists for sequential steps

**Example:**

```markdown
The system includes the following components:

- AWS IoT Core for device management
- Lambda functions for data processing
- DynamoDB for data storage
- API Gateway for REST endpoints
```

### Code Blocks

Use appropriate code fence languages:

```python
# Python code example
def calculate_risk_score(temperature, humidity, fire_distance):
    return result
```

```bash
# Shell commands
cd forestshield-backend
npm install
```

```json
{
  "deviceId": "esp32-01",
  "temperature": 23.4
}
```

## Technical Writing Standards

### Terminology

**Consistent Terms:**

- Use "IoT sensor device" or "ESP32 sensor" (not "IoT thing" or "device thing")
- Use "risk score" (not "risk level" or "risk value")
- Use "sensor reading" (not "sensor data point" or "measurement")
- Use "functional area" (FA1, FA2, FA3) consistently

**Technical Abbreviations:**

- Spell out on first use: "Infrastructure as Code (IaC)"
- Use consistent capitalization: "AWS IoT Core", "API Gateway", "DynamoDB"
- Use consistent acronyms: "MQTT", "HTTP", "HTTPS", "TLS"

### Descriptions

**System Components:**

- State what the component does
- Specify its role in the architecture
- Include relevant technical specifications
- Reference related components

**Example:**

```markdown
AWS Lambda function `process_sensor_data` receives MQTT messages from AWS IoT Core via IoT Rule triggers. The function parses sensor payload data, queries NASA FIRMS API for active fire detections, calculates risk scores using a rule-based algorithm, and stores enriched data records in DynamoDB. The function executes in Python 3.9+ runtime environment with 256 MB memory allocation and 30-second timeout.
```

### Specifications

**Be Specific:**

- Include version numbers where relevant
- Specify configuration values
- Include time intervals and frequencies
- Reference exact file paths or endpoint URLs

**Example:**

```markdown
Sensor devices transmit data at 30-second intervals. The MQTT topic pattern is `wildfire/sensors/{deviceId}`. Messages use TLS encryption on port 8883. Data is stored in DynamoDB table `WildfireSensorData` with 30-day TTL retention.
```

## Formatting Standards

### Headers

- Use `#` for document title only
- Use `##` for major sections
- Use `###` for subsections
- Use `####` sparingly for sub-subsections

### Emphasis

- **Bold** for key terms, component names, and important concepts on first mention
- _Italic_ for emphasis within sentences (use sparingly)
- `Code formatting` for file names, function names, variables, commands, and technical identifiers

### Links

- Use descriptive link text
- Include full URLs for external references
- Use relative paths for internal documentation references

**Example:**

```markdown
See [PROJECT_OVERVIEW.md](./PROJECT_OVERVIEW.md) for detailed system description.

NASA FIRMS API documentation: https://firms.modaps.eosdis.nasa.gov/api/
```

### Tables

Use tables for structured data:

| Component       | Technology    | Purpose                       |
| --------------- | ------------- | ----------------------------- |
| IoT Devices     | ESP32 + DHT11 | Environmental data collection |
| Message Broker  | AWS IoT Core  | MQTT message handling         |
| Data Processing | AWS Lambda    | Sensor data enrichment        |

### Diagrams

Use code blocks for ASCII diagrams:

```
ESP32 Sensor
    ↓ (MQTT)
AWS IoT Core
    ↓ (IoT Rule)
Lambda Function
    ↓
DynamoDB
```

## Content Organization

### Document Introduction

Every document should begin with:

- Clear document title
- Brief purpose statement (1-2 sentences)
- Overview of document contents (optional)

### Logical Flow

Organize content in logical sequence:

1. Overview and context
2. Detailed specifications
3. Implementation details
4. Examples and references

### Cross-References

Reference related documentation:

- Link to other documentation files when relevant
- Reference sections within the same document
- Maintain consistent naming for cross-references

## Specific Document Types

### README Files

**Structure:**

1. Title and brief description
2. Overview section
3. Quick start or getting started
4. Architecture or components
5. Configuration instructions
6. Usage examples
7. Related repositories or links

**Keep README files:**

- Focused on the specific repository/component
- Practical and actionable
- Free of project-wide details (reference main docs instead)

### Technical Specifications

**Include:**

- Detailed technical descriptions
- Configuration parameters
- API specifications
- Data schemas
- Architecture diagrams

**Maintain:**

- Section numbering
- Formal language
- Complete technical details
- References to standards or specifications

### Setup and Configuration Guides

**Structure:**

1. Prerequisites
2. Step-by-step instructions
3. Configuration details
4. Verification steps
5. Troubleshooting

**Format:**

- Numbered steps for sequential procedures
- Code blocks for commands
- Clear separation between sections

## Quality Checklist

Before finalizing any documentation:

- [ ] No emojis or decorative symbols
- [ ] Professional, academic tone throughout
- [ ] Consistent terminology used
- [ ] Section numbering present (for formal docs)
- [ ] Technical specifications complete and accurate
- [ ] Code examples are functional and tested
- [ ] Links are valid and descriptive
- [ ] Cross-references are accurate
- [ ] Grammar and spelling checked
- [ ] Formatting is consistent

## Examples

### Good Documentation

```markdown
## 3.0 Risk Scoring Algorithm

### 3.1 Algorithm Specification

The risk scoring algorithm employs a weighted linear combination approach combining three normalized factors: temperature, humidity, and fire proximity.

The algorithm calculates risk scores using the following formula:
```

riskScore = (w1 × temp_score) + (w2 × humidity_score) + (w3 × fire_score)

```

**Weight Configuration:**
- Temperature weight (w1): 0.4
- Humidity weight (w2): 0.3
- Fire proximity weight (w3): 0.3

Temperature values are normalized from the range 0-50°C to 0-100 scale. Humidity values use inverse normalization where lower humidity yields higher risk scores. Fire proximity is calculated as inverse distance, with closer fires contributing to higher risk scores.

### 3.2 Implementation Details

The risk scoring algorithm is implemented in the Lambda function `process_sensor_data` located in `forestshield-backend/lambda-processing/process_sensor_data.py`. The function receives sensor data and fire proximity information as inputs and returns a risk score value between 0 and 100.
```

### Poor Documentation (Avoid)

```markdown
## 🔧 Risk Scoring

Super cool algorithm! 🎯

Just multiply some stuff together:

- temp stuff: 0.4
- humidity stuff: 0.3
- fire stuff: 0.3

It's really awesome and works great! ⚡
```

## Maintenance

- Review documentation formatting periodically
- Update this guide as standards evolve
- Ensure all team members follow these standards
- Reference this guide when creating new documentation

---

**Document Version:** 1.0  
**Last Updated:** Current Semester  
**Project:** Project GreenGuard - ForestShield
