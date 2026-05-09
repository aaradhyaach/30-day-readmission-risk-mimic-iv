# 30-Day Hospital Readmission Risk Analysis — MIMIC-IV

**Tools:** R · Python · DuckDB · SQL · Quarto · tidyverse · gtsummary · pROC · survival · XGBoost · SHAP  
**Data:** MIMIC-IV (Beth Israel Deaconess Medical Center, 2008–2022)  
**Live report:** [View on GitHub Pages](https://aaradhyaach.github.io/30-day-readmission-risk-mimic-iv/readmission_analysis.html)

---

## Why this matters

Unplanned 30-day readmissions cost the U.S. healthcare system an estimated **$26 billion annually**. Under the CMS Hospital Readmissions Reduction Program (HRRP), hospitals face Medicare payment penalties of up to 3% for excess readmissions in conditions including heart failure, pneumonia, COPD, and joint replacement. Identifying which patients are at elevated risk at the time of discharge gives clinical teams a window to intervene — targeted follow-up calls, care management referrals, or scheduled outpatient visits — before a preventable return hospitalization occurs.

This project builds and evaluates a 30-day readmission risk model using real-world EHR data from MIMIC-IV, progressing from an administrative-only logistic regression baseline through an XGBoost model with lab features and SHAP-based feature importance — with a focus on what a care operations team could actually do with the findings at the time of discharge.

---

## Key findings

- **15.1% readmission rate** across 220,798 index admissions — consistent with published national benchmarks
- **Prior admissions in the 6 months** before discharge carried the highest adjusted odds (OR 1.73) — the strongest administrative predictor in the model
- **Mental/Behavioral disorders** carried the highest adjusted readmission odds by diagnosis category (OR 3.31), followed by **Neoplasms** (OR 2.35), relative to circulatory conditions
- **Each additional hospital day** increased readmission odds by 2% (OR 1.02), reflecting cumulative illness severity
- **Emergency admissions** had the highest readmission risk among admission types (OR 1.41)
- **XGBoost with lab features** achieved **AUC 0.679**, up from 0.632 (administrative baseline) and 0.644 (logistic regression with utilization + labs)
- **SHAP analysis** identified length of stay, psychiatric diagnosis, low albumin, and low hemoglobin as the strongest individual predictors — signals available in any EHR at discharge
- **Threshold analysis:** at a 0.20 cutoff, the model flags ~120 patients/day and catches 18.3 true readmissions/day from a ~121 discharge/day volume

---

## What this analysis enables

The findings directly inform three operational decisions a hospital care team faces at discharge:

- **Discharge planning** — patients with Mental/Behavioral or Neoplasm diagnoses, extended LOS, and low albumin or hemoglobin should be prioritized for care management follow-up
- **Resource allocation** — the threshold analysis translates model output into staffing-level decisions: a hospital with two dedicated post-discharge care managers needs a different operating threshold than one with a full transition care team
- **Insurance navigation** — Medicaid and Medicare patients showed higher readmission rates, suggesting a role for social work engagement before the patient leaves the building

The value of this model is not its AUC in isolation — it is its ability to rank patients by risk at the moment of discharge, before a preventable readmission occurs.

---

## Data and cohort definition

**Source:** MIMIC-IV tables — `admissions`, `patients`, `diagnoses_icd`, `labevents`  
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
| Feature engineering | Prior utilization (6-month admit count), comorbidity burden (ICD code count), Elixhauser high-risk flag, 7 discharge lab values |
| Baseline model | Multivariable logistic regression (binomial GLM) — administrative features only |
| Extended model | XGBoost classifier — administrative + utilization + lab features |
| Model comparison | Held-out 20% test set; AUC/ROC for both models |
| Feature importance | SHAP beeswarm plot on XGBoost test set predictions |
| Threshold analysis | Sensitivity, PPV, and daily alert volume at 5 operating thresholds |
| Survival analysis | Kaplan-Meier time-to-readmission via `survival` |
| Reporting | Reproducible Quarto document |

---

## Model comparison

| Model | Features | AUC |
|---|---|---|
| Logistic regression (R) | Demographics, diagnosis, LOS, insurance | 0.632 |
| Logistic regression (Python) | + Prior admissions, comorbidity count, 8 lab values | 0.644 |
| XGBoost (Python) | Same expanded feature set | 0.679 |

---

## Repository structure
