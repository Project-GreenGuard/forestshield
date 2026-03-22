# Model Training Guide

Training methodology for ForestShield AI wildfire risk prediction model.

## Overview

**Model**: Gradient Boosting Regressor | **Target**: Risk score (0-100) | **Region**: Ontario, Canada  
**Duration**: ~60s | **Performance**: RMSE=5.43, R²=0.952, 93.4% accuracy

## Data

**NASA MODIS** (https://firms.modaps.eosdis.nasa.gov/): 720K Canada fires → **60,970 Ontario fires** (8.5%)  
**Ontario Bounds**: lat(41.91-56.87), lon(-95.15 to -74.32)  
**Synthetic Samples**: 20 per fire with augmentation → 100K training samples  
**Strategy**: High-risk (near fires), Medium (moderate), Low (distant) with weather noise

## Features (11 Total)

**Raw** (5): temperature, humidity, lat, lng, nearestFireDistance  
**Derived** (6): temp_normalized, humidity_inverse, fire_proximity_score, hour_sin/cos, fire_danger_index  
**Importance**: Fire proximity 99.1%, weather 0.9%

## Training

**Pipeline**: Load fires → Generate samples → Build features → Train/test split (80/20) → Train GB model → Evaluate → Save

**Run**:
```bash
python training/train.py  # ~60s
```

**Hyperparameters**:
```python
GradientBoostingRegressor(n_estimators=100, learning_rate=0.1, max_depth=5)
```

## Performance

| Metric | Value | | Scenario | Prediction |
|--------|-------|---|----------|-----------|
| RMSE | 5.43 | | Active fire (42°C, 18% hum, 0.5km) | 97.12 HIGH |
| R² | 0.952 | | Moderate (28°C, 45% hum, 15km) | 47.3 MEDIUM |
| Accuracy | 93.4% | | Safe (22°C, 65% hum, 50km) | 22.31 LOW |

## Quick Start

Training code lives in the **`forestshield-ai`** repo (not the Lambda repos). From that project root:

```bash
# 1. Install
pip install pandas numpy scikit-learn joblib

# 2. Download NASA data (2018-2024) to data/modis_YYYY_Canada.csv
# https://firms.modaps.eosdis.nasa.gov/

# 3. Train (~60s)
python training/train.py

# 4. Test (if inference/ exists in that repo)
python inference/predict.py
```

## Configuration

```python
# Adjust samples (train.py)
samples[:500000]  # More data: 100K→60s, 500K→5min, 1.2M→15min

# Disable Ontario filter (data_preparation.py)
load_nasa_data(filter_ontario=False)  # Use all 720K fires

# Tune hyperparameters (train.py)
GradientBoostingRegressor(n_estimators=200, learning_rate=0.05, max_depth=7)
```

## Troubleshooting

| Issue | Fix |
|-------|-----|
| No CSV files | Download NASA data to `data/` |
| No Ontario fires | Check bounds or set `filter_ontario=False` |
| Low R² | Increase samples or tune hyperparameters |

## Deployment & Improvements

**Deploy**: AWS Lambda (load `.pkl`) | Vertex AI (upload to GCS)  
**Improve**: Real-time weather API, temporal features (drought index), NDVI, regional testing

---

**Version**: v1.0-gradient-boost-nasa | **Region**: Ontario | **Samples**: 100K from 60,970 fires | **Updated**: March 2026
