# Bias-Aware Early Warning System for Higher Education

A machine learning system for early identification of at-risk students that addresses algorithmic fairness across demographic groups.

## Key Results

| Metric | Value |
|--------|-------|
| **Model Performance (AUC)** | 0.889 |
| **Prediction Timing** | First 25% of course |
| **Fairness Target** | Maintained after mitigation |

The baseline model exhibited significant bias across protected attributes (especially region), which was successfully mitigated using IBM AIF360 techniques while preserving predictive accuracy.

## Research Questions

1. How do socioeconomic, geographic, and demographic characteristics predict at-risk students?
2. What is the extent of algorithmic bias across protected attributes in temporal EWS models?
3. How effective are different bias mitigation approaches in reducing disparities while maintaining performance?

## Project Structure

```
├── data/
│   ├── raw/                  # OULAD CSV files (not tracked - see below)
│   └── processed/            # Feature engineering outputs
├── notebooks/                # Analysis pipeline (01-10)
├── models/                   # Trained PyTorch models (.pt)
├── figures/                  # Visualizations (PNG)
└── docs/                     # Reference materials
```

## Dataset

This project uses the [Open University Learning Analytics Dataset (OULAD)](https://analyse.kmi.open.ac.uk/open_dataset).

**To obtain the data:**
1. Download from the link above
2. Extract CSV files to `data/raw/`

Required files:
- `studentInfo.csv` (32,593 records)
- `studentVle.csv` (10.6M records)
- `studentAssessment.csv`
- `studentRegistration.csv`
- `assessments.csv`
- `vle.csv`
- `courses.csv`

## Installation

```bash
# Clone the repository
git clone https://github.com/baker-jr-john/bias-aware-ews.git
cd bias-aware-ews

# Create virtual environment
python -m venv venv
source venv/bin/activate  # or `venv\Scripts\activate` on Windows

# Install dependencies
pip install -r requirements.txt
```

## Notebooks

Run in order:

| Notebook | Description |
|----------|-------------|
| `01_data_exploration.ipynb` | Dataset overview and EDA |
| `02_feature_engineering.ipynb` | Temporal and static feature creation |
| `03_lstm_baseline.ipynb` | LSTM model training (AUC: 0.889) |
| `04_fairness_analysis.ipynb` | Bias audit with AIF360 |
| `05_bias_mitigation_region.ipynb` | Region fairness mitigation |
| `06_bias_mitigation_imd.ipynb` | IMD band fairness mitigation |
| `07_bias_mitigation_disability.ipynb` | Disability fairness mitigation |
| `08_bias_mitigation_age.ipynb` | Age band fairness mitigation |
| `09_intersectional_analysis.ipynb` | Intersectional fairness validation |
| `10_final_summary_report.ipynb` | Results summary |

## Model Architecture

Dual-branch LSTM combining temporal VLE engagement with static demographic features:

- **Temporal branch**: LSTM (64 units) processing 10-week engagement sequences
- **Static branch**: Dense layer (32 units) for demographic features
- **Fusion**: Concatenation → Dense(32) → Dropout → Sigmoid
- **Parameters**: 21,793

## Fairness Analysis

**Protected attributes analyzed:** gender, region, IMD band, age band, disability

**Metrics used:**
- Statistical Parity Difference (SPD)
- Equal Opportunity Difference (EOD)
- Equalized Odds
- ABROCA (AUC disparity)

**Mitigation approaches:**
- Reweighting (pre-processing)
- Threshold Optimization (post-processing)
- Reject Option Classification (post-processing)

## Results Summary

| Attribute | Baseline Status | Best Mitigation | Post-Mitigation AUC |
|-----------|-----------------|-----------------|---------------------|
| Gender | Fair | N/A | 0.889 |
| Region | Unfair (4/4) | Threshold Optimization | 0.889 |
| IMD Band | Unfair (2/4) | Threshold Optimization | 0.889 |
| Disability | Unfair (2/4) | Reweighting | 0.887 |
| Age Band | Unfair (2/4) | Threshold Optimization | 0.889 |

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

The OULAD dataset is provided under CC-BY 4.0.

## Citation

If you use this work, please cite:

```
Kuzilek, J., Hlosta, M., & Zdrahal, Z. (2017). Open University Learning Analytics Dataset.
Scientific Data, 4, 170171. https://doi.org/10.1038/sdata.2017.171
```
