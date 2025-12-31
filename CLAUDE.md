# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **bias-aware Early Warning System (EWS)** research project for higher education. The goal is to predict at-risk students while addressing algorithmic fairness using the Open University Learning Analytics Dataset (OULAD).

**Primary Research Question:** How do socioeconomic, geographic, and demographic characteristics predict at-risk students, and what's the effectiveness of different bias mitigation approaches in reducing prediction disparities while maintaining strong predictive performance (AUC ROC > 0.80)?

## Technical Requirements

- **Model:** LSTM (Long Short-Term Memory) for temporal predictions
- **Prediction Timing:** Within first 25% of course completion
- **Target Performance:** AUC ROC > 0.80
- **Fairness Metrics:** ABROCA, Equalized Odds, Statistical Parity
- **Fairness Toolkit:** IBM AI Fairness 360 (AIF360)

## Project Structure

```
Capstone_Project/
├── data/
│   ├── raw/                  # OULAD CSV files
│   └── processed/            # Feature engineering outputs
│       ├── features_temporal.npy    # Weekly VLE sequences (32593 x 10 x 5)
│       ├── features_static.csv      # Static features + protected attrs
│       ├── feature_metadata.json    # Encodings and preprocessing info
│       ├── predictions_baseline.csv # Test set predictions
│       ├── fairness_results.json    # Fairness audit results
│       └── mitigation_results_region.json
├── notebooks/                # Jupyter notebooks
│   ├── 01_data_exploration.ipynb
│   ├── 02_feature_engineering.ipynb
│   ├── 03_lstm_baseline.ipynb       # Model training (AUC: 0.8889)
│   ├── 04_fairness_analysis.ipynb   # Bias audit with AIF360
│   └── 05_bias_mitigation_region.ipynb
├── models/
│   ├── lstm_baseline.pt             # Baseline model checkpoint
│   └── lstm_reweighted_region.pt    # Reweighted model for region
├── figures/                  # Visualizations (PNG)
├── docs/                     # Reference materials
├── src/                      # Reusable Python modules (future)
└── CLAUDE.md
```

## Dataset Structure

The OULAD dataset is in `data/raw/` with these tables:

| File | Records | Key Columns |
|------|---------|-------------|
| `studentInfo.csv` | 32,593 | id_student, gender, region, imd_band (deprivation), age_band, disability, final_result |
| `studentVle.csv` | 10.6M | id_student, id_site, date, sum_click (VLE interactions - primary temporal feature) |
| `studentAssessment.csv` | 173,912 | id_student, id_assessment, date_submitted, score |
| `studentRegistration.csv` | 32,593 | id_student, date_registration, date_unregistration |
| `assessments.csv` | 206 | id_assessment, assessment_type, date, weight |
| `vle.csv` | 6,364 | id_site, activity_type, week_from, week_to |
| `courses.csv` | 22 | code_module, code_presentation, length |

**Key relationships:** Tables link via `code_module`, `code_presentation`, and `id_student`. See `docs/oulad_database_schema.png` for visual reference.

**Protected attributes for fairness analysis:** gender, region, imd_band (socioeconomic), age_band, disability

## Research Objectives

1. **Audit** temporal EWS models for socioeconomic, geographic, and demographic bias
2. **Implement and compare** AIF360 mitigation techniques (pre-processing, in-processing, post-processing)
3. **Evaluate** fairness-performance trade-offs

## Key Implementation Considerations

- B presentations (February start) and J presentations (October start) may have different structures; analyze separately
- `date` fields are relative to module start (day 0); negative values indicate pre-course activity
- `final_result` values: Pass, Fail, Withdrawn, Distinction
- Score < 40 is considered Fail for assessments
- For early prediction, filter `studentVle` interactions to first 25% of course `length`

## Model Architecture

The baseline LSTM model uses a dual-branch architecture:
- **Temporal branch**: LSTM (64 hidden units) processing 10-week VLE engagement sequences
- **Static branch**: Dense layer (32 units) for demographic/academic features
- **Fusion**: Concatenation followed by Dense(32) + Dropout(0.3) + Dense(1) + Sigmoid
- **Total parameters**: 21,793

Training configuration:
- **Data split**: 70% train / 15% val / 15% test (course-stratified)
- **Batch size**: 256
- **Optimizer**: Adam (lr=0.001)
- **Early stopping**: Patience=5 based on validation AUC
- **Random seed**: 42 (for reproducibility)

## Fairness Analysis Results

Baseline model fairness audit (using thresholds: |SPD| < 0.10, |EOD| < 0.10, EqOdds < 0.10, ABROCA < 0.03):

| Attribute | SPD | EOD | EqOdds | ABROCA | Status |
|-----------|-----|-----|--------|--------|--------|
| gender | +0.061 | +0.062 | 0.077 | 0.018 | FAIR |
| **region** | **+0.242** | **+0.183** | **0.216** | **0.109** | **UNFAIR (4/4)** |
| imd_band | +0.159 | +0.048 | 0.079 | 0.055 | UNFAIR (2/4) |
| age_band | +0.016 | +0.022 | 0.107 | 0.052 | UNFAIR (2/4) |
| disability | +0.124 | +0.053 | 0.126 | 0.015 | UNFAIR (2/4) |

**Priority for mitigation**: region > imd_band > disability > age_band

## Bias Mitigation Approaches

For region attribute, Threshold Optimization was selected as the best approach:
- Maintains AUC at 0.8889 (no performance loss)
- Reduces SPD from +0.079 to +0.060
- Reduces EOD from +0.019 to -0.004 (near zero)
- Group-specific thresholds: Privileged=0.48, Unprivileged=0.51

Region grouping for fairness:
- **Privileged** (base_rate < 50%): Ireland, North Region, South East Region, South Region
- **Unprivileged** (base_rate >= 50%): All other regions
