# Bias-Aware Early Warning System for Higher Education

A machine learning system for early identification of at-risk students that addresses algorithmic fairness across demographic groups.

## Key Results

| Metric | Target | Achieved |
|--------|--------|----------|
| **Model Performance (AUC)** | > 0.80 | **0.889** |
| **Early Prediction** | First 25% of course | **10 weeks** (~26% of 33–38 week courses) |
| **Bias Mitigation** | Reduce disparities | **All 4 attributes improved** |
| **Intersectional Fairness** | No critical issues | **Validated across 16 subgroups** |

The baseline model exhibited significant bias across protected attributes (especially region, with 4/4 fairness metrics violated), which was successfully mitigated using IBM AIF360 techniques while preserving predictive accuracy.

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

This project uses the [Open University Learning Analytics Dataset (OULAD)](https://analyse.kmi.open.ac.uk/open-dataset).

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

## Requirements

- Python 3.12.6+
- See `requirements.txt` for package dependencies

## Installation

```bash
# Clone the repository
git clone https://github.com/baker-jr-john/bias-aware-ews.git
cd bias-aware-ews

# Create virtual environment (requires Python 3.12+)
python3.12 -m venv venv
source venv/bin/activate  # or `venv\Scripts\activate` on Windows

# Install dependencies
pip install -r requirements.txt
```

> **Note:** If using [pyenv](https://github.com/pyenv/pyenv), the `.python-version` file will automatically select the correct Python version.

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
| `10_final_summary_report.ipynb` | Comprehensive report with methodology, results, and limitations |

## Model Architecture

Dual-branch LSTM combining temporal VLE engagement with static demographic features:

- **Temporal branch**: LSTM (64 units) processing 10-week engagement sequences
- **Static branch**: Dense layer (32 units) for demographic features
- **Fusion**: Concatenation → Dense(32) → Dropout → Sigmoid
- **Parameters**: 21,793

**Why 10 weeks?** This represents ~25% of course completion (courses are 33–38 weeks), balancing early intervention capability with sufficient behavioral signal. Students flagged at week 10 still have 75% of the course remaining to improve.

## Fairness Analysis

**Protected attributes analyzed:** gender, region, IMD band, age band, disability

**Metrics used:**
- Statistical Parity Difference (SPD)
- Equal Opportunity Difference (EOD)
- Equalized Odds
- ABROCA (AUC disparity)

**Mitigation approaches:**
- Reweighting (pre-processing) — best for underrepresented groups (disability: 9% of data)
- Threshold Optimization (post-processing) — best for well-represented groups (region, IMD, age)
- Reject Option Classification (post-processing)

## Results Summary

| Attribute | Baseline Status | Best Mitigation | Post-Mitigation AUC |
|-----------|-----------------|-----------------|---------------------|
| Gender | Fair | N/A | 0.889 |
| Region | Unfair (4/4) | Threshold Optimization | 0.889 |
| IMD Band | Unfair (2/4) | Threshold Optimization | 0.889 |
| Disability | Unfair (2/4) | Reweighting | 0.887 |
| Age Band | Unfair (2/4) | Threshold Optimization | 0.889 |

## Key Findings

- **Regional bias was most severe:** Students in Scotland, Wales, and London were flagged at higher rates than equally-at-risk students in Ireland and Southern England
- **Mitigation approach depends on group size:** Threshold optimization works for well-represented groups; reweighting is needed for minorities
- **Intersectional disparities reflect real risk:** The 0.266 range in selection rates across subgroups mirrors actual at-risk rate variation, not discrimination
- **All mitigations preserved performance:** AUC remained ≥0.887 after bias reduction

## Limitations

- **Single institution:** Results from UK distance learning may not generalize to traditional universities
- **Historical data:** OULAD covers 2013–2014; student behaviors have evolved
- **Unmeasured factors:** Employment, health, and family circumstances affect outcomes but aren't captured
- **Prediction window trade-off:** Earlier predictions have less data; later predictions reduce intervention time

See the [final summary report](notebooks/10_final_summary_report.ipynb) for detailed discussion.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

The OULAD dataset is provided under CC-BY 4.0.

## Citation

If you use this work, please cite:

```
Kuzilek, J., Hlosta, M., & Zdrahal, Z. (2017). Open University Learning Analytics Dataset.
Scientific Data, 4, 170171. https://doi.org/10.1038/sdata.2017.171
```
