# Soil Moisture Estimation using SAR, SMAP & Hydro-Climatic Data

Multi-layer soil moisture prediction using **Sentinel-1 SAR**, **SMAP**, rainfall, temperature, and soil properties — validated against IoT field sensors across Madhya Pradesh.

Designed for deployment in data-sparse regions where continuous IoT monitoring is not feasible.

---

## Problem Statement

Soil moisture data is critical for agricultural planning, irrigation scheduling, and drought monitoring — but IoT sensor networks in rural India are sparse and costly. This project builds a machine learning model that estimates soil moisture at multiple depths (SM_L1 to SM_L6) using freely available satellite and climate data, reducing dependence on ground sensors.

---

## Study Area

- **State:** Madhya Pradesh, India
- **Districts:** Sehore & Khargone
- **Ground truth:** ~10 IoT soil moisture monitoring sites

---

## Data Sources

| Dataset | Source | Purpose |
|---------|--------|---------|
| Sentinel-1 SAR | Google Earth Engine | Surface backscatter, short-term wetting |
| SMAP Soil Moisture | NASA | Background soil moisture state |
| Rainfall (CHIRPS) | Climate Hazards Group | Precipitation accumulation |
| Temperature (ERA5 / IMD) | ECMWF / IMD | Evapotranspiration proxy |
| Soil Properties | SoilGrids / Field Data | Water retention characteristics |
| DEM | SRTM | Elevation and slope |
| IoT Sensors | Field observations | Ground truth labels |

---

## Feature Engineering

Features were designed to capture the physical processes driving soil moisture:

**SAR (Sentinel-1) — short-term surface dynamics**
- `VV_linear`, `delVV_linear` (change in backscatter), `VV/VH_linear`

**Hydro-climatic forcing**
- `Rain_7d`, `Rain_15d` — accumulated rainfall over 7 and 15 days
- `Tavg`, `Tmax`, `Tmin` — temperature for ET estimation

**Soil and terrain**
- `clay_frac`, `silt_frac` — control water retention behavior
- `elevation`, `slope` — drainage and runoff characteristics

**Seasonal encoding** (avoids discontinuity at year boundaries)
- `sin_doy`, `cos_doy` — day-of-year encoded as sine/cosine

**State variable**
- `SM_SMAP` — acts as a soil moisture memory proxy, capturing longer-term moisture state that SAR alone cannot see

---

## Model

| Property | Detail |
|----------|--------|
| Algorithm | Random Forest Regressor |
| Validation | TimeSeriesSplit (no data leakage) |
| Tuning | GridSearchCV |
| Target | Multi-layer SM (SM_L1 – SM_L6) |

TimeSeriesSplit was used deliberately to prevent future data leaking into training — critical for any time-series soil moisture model.

---

## Results

| Metric | Value |
|--------|-------|
| Train R² | ~0.91 |
| Test R² | ~0.58 – 0.62 |
| RMSE | ~9 – 10% VWC |

The train/test R² gap reflects the genuine challenge of generalizing from ~10 sparse IoT sites to unseen spatial locations using satellite data — a known hard problem in remote sensing. The test R² of ~0.60 is consistent with published literature for SAR-based soil moisture retrieval under similar data constraints, and the predictions remain physically consistent across depths.

---

## Key Insights

- **SMAP** acts as a soil moisture memory — it captures the slow-draining deeper moisture state that SAR backscatter (which responds to surface conditions within hours) cannot see alone
- **SAR** captures rapid short-term wetting events after rainfall
- **Rainfall accumulation windows** (7d, 15d) explain hydrological forcing better than single-day values
- **Soil texture** (clay/silt fraction) controls how long moisture is retained after rain events

---

## Limitations

- ~10 IoT validation sites — limited spatial generalizability
- No vegetation index (NDVI) — canopy effects not corrected
- Scale mismatch between point IoT measurements and satellite footprint
- Irregular satellite revisit times create temporal gaps

---

## Future Work

- Integrate NDVI for vegetation correction
- Expand IoT sensor coverage across more districts
- Explore LightGBM / XGBoost for comparison
- Add prediction uncertainty estimation (conformal prediction or quantile RF)
- Build a real-time inference pipeline via GEE + FastAPI

---

## Tech Stack

| Category | Tools |
|----------|-------|
| ML | scikit-learn (Random Forest, GridSearchCV) |
| Data processing | Pandas, NumPy |
| Satellite data | Google Earth Engine |
| Geospatial | Rasterio, GeoPandas |
| Environment | Python 3, Jupyter |

---

## Author

**Nitesh Singh Bhadoriya**
AI & ML Engineer — MPSEDC, Bhopal
[LinkedIn](https://www.linkedin.com/in/nitesh-singh-bhadoriya-917411226) · [GitHub](https://github.com/nitesh-bhadoriya)

---

*Developed at MPSEDC as part of agricultural monitoring and water resource management initiatives for Madhya Pradesh.*
