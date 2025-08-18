# Coastal Monitoring Using Satellite Altimetry  

This repository contains my workflow for analyzing coastal sea level variability along Bangladesh using multi-mission satellite altimetry (Jason-3 & Sentinel-3 SRAL).  

The main goals are to:  
- Subset and preprocess global SLA data for the Bangladesh coast.  
- Collocate Jason-3 and Sentinel-3 observations.  
- Interpolate SLA fields and validate cross-mission consistency.  
- Experiment with simple ML models and Gaussian Processes to map biases and coastal SLA structure.  

---

## 1. Preprocessing  

The starting point was the global CMEMS DUACS SLA dataset.  

- Subset to **88–93°E, 20–26°N** (coastal Bangladesh).  
- Saved as a smaller NetCDF for repeat use.  
- Quick mean-SLA plot to confirm data coverage.  

**Why:** keeps the workflow light and focused on the coast instead of the full global grid.  

**Output:**  
- `processed_sla_2024_bangladesh.nc`  

---

## 2. Sentinel-3 SRAL (20 Hz) Track Extraction  

Steps:  
- Subset raw SRAL data to the same region/time window.  
- Flattened into tidy table (`lon, lat, time, SLA`).  
- Saved per-track files for later use.  

**Why:** Jason-3 follows fixed ground tracks, so SRAL data needed to be reshaped to align for collocation.  

**Output:**  
- Flattened track files in `/Preprocessing`  

---

## 3. Collocation (CollocationV3)  

Matched Jason-3 and Sentinel-3 observations in space/time.  

- **Windows:** 10 km / 24 h.  
- **QC filters:** removed outliers and large |Δ| mismatches.  
- **Bias correction:** applied **daily-median correction** (S3 vs J3).  

**Outputs:**  
- `s3_j3_pairs_baseline_qc.parquet`  
- Daily/monthly bias CSVs  

---

## 4. Interpolation & Analysis (InterpolationV2)  

Interpolated Sentinel-3 (bias-corrected) SLA to a regular grid and validated against Jason-3.  

- **Interpolation:** radial basis function (thin-plate spline).  
- **Grid:** 0.125° (matches DUACS resolution).  
- **Validation:** scatter plots, RMSE vs gap, residual maps, seasonal cycles.  

**Figures produced:**  
1. Daily bias method comparison  
2. Scatter (S3bc vs J3)  
3. RMSE vs collocation gap  
4. Spatial residual map  
5. Daily interpolated maps  
6. Seasonal SLA cycle (monsoon vs rest)  
+ stricter QC scatter  

**Outputs:**  
- `s3bc_interpolated_daily_rbf_baseline_qc.nc`  
- `daily_pair_stats_baseline_qc.csv`  
- `monthly_interp_pair_stats_baseline_qc.csv`  
- Figures under `/InterpolationV2/Figures`  

---

## 5. Machine Learning Experiments (ML)  

Two experimental approaches to model SLA biases and coastal fields:  

### 5.1 Ridge Regression Bias Maps  
- Learned Δ = (S3bc − J3) using polynomial spatial terms + seasonal cycle.  
- GroupKFold CV by month (avoids leakage).  
- Produced bias map for a target month on 0.25° grid.  
- Compared against a monthly-mean baseline.  

**Outputs:**  
- Bias map parquet + scatter/map PNGs  
- Model JSON with coefficients/settings  
- Metrics CSV  

### 5.2 Gaussian Process “GPSat-style” Maps  
- Tiled GP regressions (1° tiles, 0.5° overlap).  
- Kernel: `Const × RBF([60 km, 60 km, 1.5 d]) + White noise`.  
- Inverse-variance blending to stitch tiles with uncertainty.  
- High-res 0.05° daily SLA maps.  
- Bangladesh basemap overlay with city markers.  

**Outputs:**  
- `gpsat_style_map_<date>.parquet` (mean/std/n_neighbors)  
- GP mean + uncertainty figures  
- Basemap overlay PNG  
- Run log JSON  

---
