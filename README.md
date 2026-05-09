# 30-Day Hospital Readmission Risk Analysis — MIMIC-IV

**Tools:** R · Python · DuckDB · SQL · Quarto · tidyverse · gtsummary · pROC · survival · scikit-learn · XGBoost · SHAP  
**Data:** MIMIC-IV (Beth Israel Deaconess Medical Center, 2008–2022)  
**Live report:** [View on GitHub Pages](https://aaradhyaach.github.io/30-day-readmission-risk-mimic-iv/readmission_analysis.html)

---

## Why this matters

Unplanned 30-day readmissions cost the U.S. healthcare system an estimated **$26 billion annually**. Under the CMS Hospital Readmissions Reduction Program (HRRP), hospitals face Medicare payment penalties of up to 3% for excess readmissions in conditions including heart failure, pneumonia, COPD, and joint replacement. Identifying which patients are at elevated risk at the time of discharge gives clinical teams a window to intervene — targeted follow-up calls, care management referrals, or scheduled outpatient visits — before a preventable return hospitalization occurs.

This project builds and evaluates a 30-day readmission risk model using real-world EHR data from MIMIC-IV, with a focus on what administrative, demographic, and clinical signals are most predictive — and what a care operations team could actually do with the findings.

---

## Key findings

- **15.1% readmission rate** across 220,798 index admissions — consistent with published national benchmarks
- **Prior admissions in the 6 months** before discharge carried the highest adjusted odds (OR 1.73) — the strongest single administrative predictor
- **Mental/Behavioral disorders** carried the highest adjusted odds among diagnosis categories (OR 3.31), followed by **Neoplasms** (OR 2.35), relative to circulatory conditions
- **Each additional hospital day** increased readmission odds by 2% (OR 1.02); each additional ICD code increased odds by 2% (OR 1.02)
- **Direct Emergency admissions** had the highest readmission risk among admission types (OR 1.41)
- **XGBoost with lab features achieved AUC 0.679**, up from 0.632 (administrative baseline) and 0.644 (logistic regression with utilization + labs)
- **SHAP analysis** identified length of stay, psychiatric diagnosis, low albumin, and low hemoglobin as the strongest individual-level predictors — signals available in any EHR at discharge
- **Threshold analysis:** at a 0.20 probability cutoff, the model flags ~120 patients/day and catches 18.3 true readmissions/day on a test set of 44,168 admissions

---

## What this analysis enables

The findings directly inform three operational decisions a care team faces at discharge:

- **Discharge planning** — patients with Mental/Behavioral or Neoplasm diagnoses, extended LOS, or low albumin/hemoglobin should be prioritized for care management follow-up
- **Resource allocation** — the threshold analysis translates model output into daily alert volumes, letting hospitals match the operating threshold to their actual care management capacity
- **Early intervention signals** — prior utilization, lab values, and diagnosis category are all available before the patient leaves the building, making real-time risk scoring feasible with existing EHR data

---

## Model comparison

| Model | Features | AUC |
|---|---|---|
| Logistic regression (admin only) | Demographics, diagnosis, LOS, insurance | 0.632 |
| Logistic regression (+ utilization + labs) | + Prior admissions, comorbidity count, 8 lab values | 0.644 |
| XGBoost (+ utilization + labs) | Same expanded feature set | 0.679 |

---

## Operational threshold analysis

Based on the held-out test set (n = 44,168 admissions, ~121 discharges/day):

| Threshold | Sensitivity | PPV | Patients flagged/day | Readmissions caught/day |
|---|---|---|---|---|
| 0.15 | 100.0% | 15.1% | 120.9 | 18.3 |
| 0.20 | 99.9% | 15.2% | 120.0 | 18.3 |
| 0.25 | 99.1% | 15.5% | 116.6 | 18.1 |
| 0.30 | 96.9% | 16.3% | 108.5 | 17.7 |
| 0.40 | 83.5% | 19.3% | 79.2 | 15.3 |

The appropriate operating threshold depends on available care management capacity — a hospital with two dedicated post-discharge care managers needs a very different cutoff than one with a full transition care team.

---

## Data and cohort definition

**Source:** MIMIC-IV tables — `admissions`, `patients`, `diagnoses_icd`  
**Index admission criteria:**
- Adult patients (age ≥ 18)
- First non-elective, non-newborn hospitalization
- Survived to discharge (`hospital_expire_flag = 0`)

**Readmission definition:** Any return hospitalization to the same institution within 30 days of index discharge date

**Final cohort:** 220,798 index admissions | 33,342 readmissions (15.1%)

---

## Methods summary

| Step | Approach |
|---|---|
| Data querying | DuckDB SQL on compressed MIMIC-IV CSVs |
| Cohort construction | Window functions to identify first admissions; 30-day readmission join |
| ICD mapping | ICD-10 chapter ranges → 8 disease categories |
| Feature engineering | Prior utilization (6-month admit count), comorbidity burden (ICD code count), Elixhauser high-risk flag, 8 discharge lab values |
| Baseline model | Multivariable logistic regression (binomial GLM, R) |
| Extended model | Logistic regression + XGBoost (Python, scikit-learn) on expanded feature set |
| Interpretability | SHAP beeswarm plot (top 15 features, test set) |
| Performance | AUC / ROC curve comparison across models |
| Threshold analysis | Sensitivity, PPV, and daily alert volume at 5 operating thresholds |
| Survival analysis | Kaplan-Meier time-to-readmission via `survival` |
| Reporting | Reproducible Quarto document |

---

## Repository structure

```
├── readmission_analysis.qmd     # Full reproducible R analysis
├── readmission_analysis.html    # Rendered report (GitHub Pages)
├── readmission_xgboost.ipynb    # Python model comparison + SHAP analysis
├── mimic-iv/                    # MIMIC-IV data files (not included — requires PhysioNet access)
│   └── hosp/
│       ├── admissions.csv.gz
│       ├── patients.csv.gz
│       └── diagnoses_icd.csv.gz
└── README.md
```

---

## How to reproduce

1. Apply for MIMIC-IV access at [physionet.org](https://physionet.org/content/mimiciv/) (free, requires CITI training)
2. Download and place the `hosp/` files in `mimic-iv/hosp/`
3. For the R analysis: open `readmission_analysis.qmd` in RStudio and render with Quarto
4. For the Python model: open `readmission_xgboost.ipynb` in Jupyter and run all cells

**R packages required:** `rprojroot`, `DBI`, `duckdb`, `tidyverse`, `tableone`, `gtsummary`, `pROC`, `survival`  
**Python packages required:** `pandas`, `scikit-learn`, `xgboost`, `shap`, `matplotlib`, `duckdb`

---

## Limitations

This analysis has several limitations worth noting for anyone extending the work:

- **Single-center data:** MIMIC-IV represents one academic medical center in Boston; findings may not generalize to community hospitals or other patient populations
- **ICD-9 admissions:** 71% of admissions were coded under ICD-9 or lacked a mappable primary ICD-10 diagnosis and were grouped as "Other," reducing diagnostic granularity
- **LOS as severity proxy:** Length of stay is used as an indirect measure of illness severity rather than a validated clinical severity score (e.g., APACHE, SOFA)
- **Same-institution readmissions only:** The 30-day readmission definition captures only returns to Beth Israel Deaconess; transfers to other facilities are not counted, which likely underestimates true readmission rates

---

## Next steps

- **Shiny risk calculator** — interactive discharge-time tool that takes patient inputs and returns a predicted readmission probability
- **Subgroup analysis** — test whether top predictors hold within HRRP penalty conditions (CHF, AMI, pneumonia, COPD) specifically
- **Calibration evaluation** — complement AUC with a calibration curve to assess whether predicted probabilities are reliable at the individual patient level

---

*Data from MIMIC-IV is publicly available under a data use agreement. No identifiable patient information is included in this repository.*
