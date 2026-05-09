---
title: "30-Day Hospital Readmission Risk Analysis"
author: "Aaradhya Acharya"
date: today
format: html
execute:
  echo: true
  warning: false
  message: false
---

## Executive Summary

Unplanned 30-day readmissions are among the most costly and measurable failures in post-acute care, costing the U.S. healthcare system an estimated $26 billion annually. Under the CMS Hospital Readmissions Reduction Program, hospitals face Medicare payment penalties of up to 3% for excess readmissions in priority conditions including heart failure, pneumonia, and COPD.

This analysis applies multivariable logistic regression to 220,798 adult index admissions from the MIMIC-IV database to identify administrative and demographic predictors of 30-day readmission. Key findings:

- **15.1% readmission rate** (33,342 of 220,798 admissions) — consistent with national benchmarks
- **Mental/Behavioral disorders** (OR 3.26) and **Neoplasms** (OR 2.34) carried the highest adjusted readmission odds among diagnosis categories
- **Each additional hospital day** increased readmission odds by 4% (OR 1.04), reflecting illness severity
- **Emergency admissions** had the highest readmission risk by admission type (OR 1.49)
- Baseline model **AUC: 0.632** — expected for administrative-only features; clinical variables (labs, vitals, comorbidity indices) would improve discrimination

**Clinical implication:** At discharge, patients with psychiatric or oncologic diagnoses, extended lengths of stay, and emergency admission status represent the highest-priority group for care management follow-up and post-discharge outreach.

---

## Background

Hospital readmissions within 30 days of discharge represent a measurable and often preventable failure in the care continuum. They account for an estimated $26 billion in annual U.S. healthcare spending, and under the CMS Hospital Readmissions Reduction Program (HRRP) — active since 2012 — hospitals face Medicare payment reductions of up to 3% for excess readmissions in six priority conditions: heart failure, acute myocardial infarction, pneumonia, COPD, hip and knee replacement, and coronary artery bypass graft surgery.

Beyond financial penalties, high readmission rates signal gaps in discharge planning, post-acute follow-up, and care transitions. Identifying which patients are at elevated risk at the time of discharge gives clinical teams an actionable window: a care management referral, a scheduled outpatient call at day 3, or a social work consult before the patient leaves the building can meaningfully reduce the probability of return.

This analysis uses MIMIC-IV, a large de-identified EHR dataset from Beth Israel Deaconess Medical Center (2008–2022), to build and evaluate a 30-day readmission risk model using administrative and demographic variables. The goal is not only to achieve the best possible discrimination, but to identify which patient characteristics are most strongly associated with readmission — information a care operations team can act on today, with data already available in any EHR at discharge.
