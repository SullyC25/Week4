Coastal Monitoring Using Satellite Altimetry

This repo documents my workflow for analyzing coastal sea level variability along Bangladesh using multi-mission satellite altimetry (Jason-3 & Sentinel-3 SRAL).
The main goals are to:

Subset and preprocess global SLA data for the Bangladesh coast.

Collocate Jason-3 and Sentinel-3 observations.

Interpolate SLA fields and validate cross-mission consistency.

Experiment with simple ML models and Gaussian Processes to map biases and coastal SLA structure.

1. Preprocessing

The starting point was the global CMEMS DUACS SLA dataset.
Steps included:

Subsetting to the region 88–93°E, 20–26°N (coastal Bangladesh).

Saving as a smaller NetCDF for repeat use.

Quick plotting of the mean SLA field to check data quality.

Why: keeps the workflow light and focused on the coast instead of the full global grid.

Output: processed_sla_2024_bangladesh.nc

2. Sentinel-3 SRAL (20 Hz) Track Extraction

I downloaded and processed raw Sentinel-3 SRAL tracks (20 Hz).
Main steps:

Subsetting the track data to the same region/time window.

Flattening the variables into a tidy table (lon, lat, time, SLA).

Saving per-track files for easier handling later.

Why: Jason-3 is already along repeat ground tracks, so I needed to put SRAL data into a comparable form for collocation.

Output: flattened track files saved under /Preprocessing.

3. Collocation (CollocationV3)

Here I matched Jason-3 and Sentinel-3 observations in space and time.

Windows: 10 km / 24 h (default) so pairs are close enough to be comparable but still leave enough data.

QC filters: removed large |Δ| and flagged outliers to avoid spurious mismatches.

Bias correction: applied a daily median correction (S3 vs J3) rather than a global mean. This handles day-to-day offsets without distorting the seasonal signal.

Outputs:

s3_j3_pairs_baseline_qc.parquet — collocated pairs after QC.

Summary CSVs with daily bias values and stats.

4. Interpolation & Analysis (InterpolationV2)

This stage interpolated Sentinel-3 bias-corrected SLA fields to a regular grid and produced the main analysis figures.

Interpolation: radial basis function (thin-plate spline), smooth enough to handle coastal gradients.

Grid: 0.125° (matches DUACS resolution).

Validation: scatter plots, RMSE vs gap, residual maps, and seasonal cycles.

Figures:

Daily bias method comparison.

Scatter S3bc vs J3.

RMSE vs collocation distance/time.

Spatial residual map.

Daily maps (interpolated vs Jason-3).

Seasonal SLA (monsoon vs rest).

stricter QC scatter for a “clean” check.

Outputs:

Gridded SLA: s3bc_interpolated_daily_rbf_baseline_qc.nc

Pair stats: daily_pair_stats_baseline_qc.csv, monthly_interp_pair_stats_baseline_qc.csv

Figures under /InterpolationV2/Figures

5. Machine Learning Experiments (ML)

I tested two lightweight approaches to model biases and SLA fields:

5.1 Ridge Regression Bias Maps

Learned residual Δ = (S3bc − J3) using polynomial spatial terms + seasonal cycle.

GroupKFold by month to avoid data leakage.

Produced a bias map for a target month (0.25° grid).

Compared skill against a trivial monthly-mean baseline.

Outputs:

Bias map parquet + PNG scatter/map.

JSON with model coefficients and settings.

Metrics CSV with CV RMSE and skill score.

5.2 Gaussian Process “GPSat-style” Maps

Tiled GP regressions (1° tiles, 0.5° overlap).

Kernel: Const × RBF([60 km, 60 km, 1.5 d]) + White noise.

Used inverse-variance blending to stitch tiles into a smooth field with uncertainty.

High-res 0.05° daily maps (SLA mean + σ).

Added a Bangladesh basemap overlay with city markers for context.

Outputs:

gpsat_style_map_<date>.parquet with mean/std/n_neighbors.

Figures: GP mean, uncertainty, and basemap overlay.

Run log JSON with kernel and parameters.
