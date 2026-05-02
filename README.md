# 30-Day Hospital Readmission Risk Analysis

An end-to-end clinical data analysis using MIMIC-IV EHR data to identify predictors of 30-day hospital readmission.

## Methods
- Cohort construction using SQL (DuckDB) from MIMIC-IV admissions, patients, and diagnoses tables
- 220,798 adult index admissions with 15.1% 30-day readmission rate
- Logistic regression with odds ratios, Table 1, and ROC/AUC evaluation
- Fully reproducible Quarto report

## Tools
SQL, R, DuckDB, Quarto, tidyverse, gtsummary, tableone, pROC

## Data
MIMIC-IV (PhysioNet). Data not included per PhysioNet data use agreement.

## Structure
- `analysis/` — Quarto analysis file
- `R/` — helper functions (if any)
- `output/` — rendered HTML report
