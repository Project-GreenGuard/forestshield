# ForestShield AI Prediction & Training Spec (Draft)

> This document is intentionally open-ended so Samira (and future contributors) can propose and iterate on the AI design while staying aligned with the existing system.

## 1. Purpose

- Define how AI-based wildfire risk prediction should *conceptually* work in ForestShield.
- Describe the inputs/outputs that the backend and Lambdas will eventually rely on.
- Capture an initial training strategy and leave space for iteration once we see how it fits with our deployment and budget.

## 2. Model Responsibility

**Goal:** Given a single sensor reading (plus context), output a wildfire risk assessment that is compatible with the existing rule-based `riskScore` (0–100) and risk levels.

High-level contract:

- **Input:** One sensor data point (what processing Lambda sees after IoT + NASA FIRMS).
- **Output:**
  - `risk_score` – float, 0–100
  - `risk_level` – one of `LOW`, `MEDIUM`, `HIGH`
  - `model_version` – string identifier (for debugging / A/B later)

The rest of the system should not care whether this comes from a rule-based model, a local ML model, or Vertex AI — only that the contract above is honoured.

## 3. Planned Inputs (Features)

These are *candidate* features based on the current architecture; they are not final. Samira can modify/refine this list as she designs the model.

Core fields already flowing through the system:

- `temperature` (float, °C)
- `humidity` (float, %)
- `lat` (float, degrees)
- `lng` (float, degrees)
- `nearestFireDistance` (float, km; can default to a max value like 100 when unknown)
- `timestamp` (ISO8601 string)

Possible derived features (to be validated later):

- Normalised temperature, e.g. `temp_normalized = temperature / 50.0`
- Inverse humidity, e.g. `humidity_inverse = 1 - humidity / 100.0`
- Fire proximity score, e.g. scaled version of `nearestFireDistance`
- Time-based features:
  - hour-of-day (0–23, or sin/cos encoding)
  - day-of-week or month-of-year
- Simple location buckets (e.g. cluster lat/lng into regions)

**Open question for Samira:**  
- Which of these features make sense scientifically, and which should be dropped or extended (e.g. vegetation index, weather history, etc.)?

## 4. Output Semantics

- `risk_score`:
  - Continuous 0–100 scale.
  - Should roughly align with the current rule-based score so the dashboard doesn’t need to be redesigned.
- `risk_level`:
  - Suggested thresholds (can be revisited):
    - `LOW`: 0–30
    - `MEDIUM`: >30–60
    - `HIGH`: >60–100
- `model_version`:
  - Free-form string (e.g. `v1-rule-approx`, `v2-vertex-linear`, etc.) for logging and debugging.

## 5. Initial Training Strategy (for Samira)

**Phase 1 – Cheap and pragmatic:**

- **Labels:**
  - Use the existing rule-based `riskScore` as the target (`y`) so we can train a model that approximates the current behaviour.
  - Later, if we ever get real incident/outcome data, we can retrain with better labels.
- **Data source:**
  - Historical data exported from DynamoDB or collected into CSVs with the fields listed above.
  - It’s fine to start with a relatively small dataset; this is more about wiring and deployment than deep science.
- **Model type:**
  - Simple regression model (e.g. RandomForestRegressor, Gradient Boosted Trees, or even linear regression as a baseline).
- **Goal:**
  - Get a model that is:
    - Stable (no crazy predictions),
    - Roughly aligned with the current rule,
    - Easy to export for Vertex AI.

**Phase 2 – Vertex AI integration:**

- Export the trained model from the `forestshield-ai` repo in a format Vertex AI can serve (joblib, pickle, or container depending on the approach we settle on).
- Deploy to a single Vertex AI endpoint in `us-central1` with minimal replicas for cost control.
- The processing Lambda (AWS) will call this endpoint over HTTP and receive `risk_score` / `risk_level` back.

## 6. Deployment & Cost Considerations (High-Level)

- Training can initially run:
  - Locally (developer machine),
  - Or in a one-off cloud job (Colab/Cloud Build) triggered manually.
- Serving should be:
  - Via Vertex AI Online Prediction (single small endpoint),
  - With low or zero minimum replicas to avoid idle cost.
- No always-on custom servers are required for the first iteration.

## 7. Open Items for Samira to Propose

Samira can propose and document (in comments on this file or a follow-up doc):

- Final feature list (which inputs/derived features to actually use).
- Specific model choice (algorithm, hyperparameters).
- How to handle missing or noisy sensor data.
- How to gradually migrate from the rule-based risk to the ML-based risk (e.g. shadow mode, logging only).

Once we see how this design behaves with our deployment structure and budget, we can tighten the spec and update the training + inference code in `forestshield-ai` accordingly.

