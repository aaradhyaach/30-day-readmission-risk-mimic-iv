# 30-Day Hospital Readmission Risk Analysis — MIMIC-IV

**Tools:** R · DuckDB · SQL · Quarto · tidyverse · gtsummary · pROC · survival  
**Data:** MIMIC-IV (Beth Israel Deaconess Medical Center, 2008–2022)  
**Live report:** [View on GitHub Pages](https://aaradhyaach.github.io/30-day-readmission-risk-mimic-iv/readmission_analysis.html)

---

## Why this matters

Unplanned 30-day readmissions cost the U.S. healthcare system an estimated **$26 billion annually**. Under the CMS Hospital Readmissions Reduction Program (HRRP), hospitals face Medicare payment penalties of up to 3% for excess readmissions in conditions including heart failure, pneumonia, COPD, and joint replacement. Identifying which patients are at elevated risk at the time of discharge gives clinical teams a window to intervene — targeted follow-up calls, care management referrals, or scheduled outpatient visits — before a preventable return hospitalization occurs.

This project builds and evaluates a 30-day readmission risk model using real-world EHR data from MIMIC-IV, with a focus on what administrative and demographic signals are most predictive and what a care operations team could actually do with the findings.

---

## Key findings

- **15.1% readmission rate** across 220,798 index admissions — consistent with published national benchmarks
- **Mental/Behavioral disorders** carried the highest adjusted odds of readmission (OR 3.26), followed by **Neoplasms** (OR 2.34), relative to circulatory conditions
- **Each additional hospital day** was associated with a 4% increase in readmission odds (OR 1.04), reflecting higher underlying illness severity
- **Emergency admissions** had the highest readmission risk among admission types (OR 1.49)
- **Medicare patients** had modestly elevated odds vs. Medicaid (OR 1.05); patients with private insurance had lower odds (OR 0.84)
- Baseline logistic regression using administrative variables achieved **AUC 0.632** — consistent with published literature for models without clinical lab or vital sign data

---

## What this analysis enables

At a discharge rate of ~220,000 patients per year (a realistic volume for a mid-to-large academic medical center), a model flagging the top 20% by predicted risk would capture a meaningful portion of true readmissions. The findings directly inform:

- **Discharge planning** — patients with Mental/Behavioral or Neoplasm diagnoses and long LOS should be prioritized for care management follow-up
- **Resource allocation** — Emergency-admission patients warrant closer post-discharge monitoring
- **Insurance navigation** — Medicaid and Medicare patients showed higher readmission rates, suggesting a role for social work engagement at discharge

---

## Data and cohort definition

**Source:** MIMIC-IV tables — `admissions`, `patients`, `diagnoses_icd`  
**Index admission criteria:**
- Adult patients (age ≥ 18)
- First non-elective, non-newborn hospitalization
- Survived to discharge (hospital_expire_flag = 0)

**Readmission definition:** Any return hospitalization to the same institution within 30 days of index discharge date

**Final cohort:** 220,798 index admissions | 33,342 readmissions (15.1%)

---

## Methods summary

| Step | Approach |
|---|---|
| Data querying | DuckDB SQL on compressed MIMIC-IV CSVs |
| Cohort construction | Window functions to identify first admissions; 30-day readmission join |
| ICD mapping | ICD-10 chapter ranges → 8 disease categories |
| Modeling | Multivariable logistic regression (binomial GLM) |
| Performance | AUC / ROC curve via `pROC` |
| Survival analysis | Kaplan-Meier time-to-readmission via `survival` |
| Reporting | Reproducible Quarto document |

---

## Repository structure

```
├── readmission_analysis.qmd    # Full reproducible analysis
├── readmission_analysis.html   # Rendered report (GitHub Pages)
├── mimic-iv/                   # MIMIC-IV data files (not included — requires PhysioNet access)
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
3. Open `readmission_analysis.qmd` in RStudio and render with Quarto

**R packages required:** `rprojroot`, `DBI`, `duckdb`, `tidyverse`, `tableone`, `gtsummary`, `pROC`, `survival`

---

## Limitations and next steps

This model uses only administrative and demographic variables. Incorporating clinical features — lab values, vital signs, the Charlson Comorbidity Index, or discharge medications — would substantially improve AUC. A comparison of models (logistic regression vs. random forest vs. XGBoost) and a threshold analysis showing operational trade-offs (sensitivity vs. alert burden) are planned next.

See the [full report](https://aaradhyaach.github.io/30-day-readmission-risk-mimic-iv/readmission_analysis.html) for complete methods, regression table, ROC curve, and Kaplan-Meier analysis.

---

*Data from MIMIC-IV is publicly available under a data use agreement. No identifiable patient information is included in this repository.*
