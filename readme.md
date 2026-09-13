# TESS Light Curve Anomaly Detection

Unsupervised detection of astrophysically interesting anomalies in TESS light curves, using cross-sector consistency as the core validation signal to distinguish genuine stellar behavior from instrumental noise.

## Motivation 

TESS observes hundreds of thousands of stars across overlapping sectors, generating far more data than can be manually inspected. A light curve anomaly (an unusual dip, spike, or pattern) could be a genuine astrophysical event — a transit, flare, or unusual variable star — or it could simply be instrumental noise (a cosmic ray hit, a systematic pipeline artifact).

This project's central idea: **if an anomaly appears consistently across multiple independent observations (sectors) of the same star, it is far more likely to be real** than an anomaly seen only once. Random instrumental noise doesn't reliably repeat across sectors; genuine stellar behavior does.

To maximize the statistical power of this check, targets are prioritized from the TESS Continuous Viewing Zone (CVZ), where stars near the ecliptic poles can accumulate 20+ sectors of repeat observations.

## Pipeline

| Step | Description | Status |
|------|-------------|--------|
| 1. Target Selection | Query MAST for TESS targets with ≥3 sectors of observations, prioritizing CVZ stars |  Complete |
| 2. Download & Clean | Download SPOC light curves via `lightkurve`, clean (remove NaNs, normalize, sigma-clip, flatten), resample to a fixed-length grid, and cache to disk |  Complete |
| 3. Baseline Detector | Extract simple statistical features (std, peak-to-peak, roughness, percentiles, skewness, kurtosis) and fit an Isolation Forest as an honest, explainable comparison point |  Complete |
| 4. VAE-based Detector | Train a Variational Autoencoder to learn "normal" light curve shapes; use reconstruction error as a deeper anomaly signal |  In progress |
| 5. Cross-Sector Consistency Check | For each flagged anomaly, check whether the same signature recurs across the star's other sectors |  Planned |
| 6. Validation | Injection-recovery testing with synthetic anomalies; cross-reference against known TOI/variable star catalogs |  Planned |

## Repository structure

```
├── 01_multisector_targets.ipynb          # Step 1: MAST query, target selection
├── 02_lightcurve_processing.ipynb        # Step 2: download, clean, resample, cache
├── 03_baseline_isolation_forest.ipynb    # Step 3: statistical feature baseline
├── multisector_tic_targets.csv           # Output of Step 1
├── baseline_anomaly_scores.csv           # Output of Step 3
└── lightcurve_cache/                     # Cached, cleaned light curve arrays (.npy)
```

## Methodology notes

- **Gap-aware resampling:** every light curve is resampled onto a fixed 2000-point time grid so all observations are directly comparable. Points falling inside large data gaps are explicitly flagged (rather than silently trusting the interpolated guess) so downstream steps don't mistake missing-data artifacts for real signal.
- **Baseline before deep learning:** Step 3's Isolation Forest baseline exists specifically so the VAE's added value (or lack of it) can be measured honestly, rather than assumed.
- **SPOC pipeline only:** light curves are restricted to TESS's primary, most rigorously validated reduction pipeline for consistency across targets.

## Tech stack

`astroquery` · `lightkurve` · `numpy` / `pandas` · `scikit-learn` · `PyTorch` · `matplotlib`

## Status

Actively in development. Steps 1–3 complete and validated (baseline anomaly scores visually confirmed against raw light curves); Step 4 (VAE) under construction.
