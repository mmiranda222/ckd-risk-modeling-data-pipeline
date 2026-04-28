# Methodology

## 1. Project Objective

The objective of this project was to construct a clinically coherent and temporally consistent patient-level dataset to support predictive modeling for chronic kidney disease (CKD) progression and related outcomes.

Rather than immediately developing predictive models, the focus was placed on ensuring that the underlying data structure was valid for time-dependent analysis, particularly for survival modeling and risk prediction at fixed horizons.

---

## 2. Data Sources (Abstracted)

The dataset was built from multiple heterogeneous clinical data sources, including:

- Longitudinal patient records  
- Clinical encounter data (consultations)  
- Clinical history records  
- Laboratory results  
- Pre-constructed outcome datasets  

These sources were not originally designed for analytical use, requiring significant transformation to produce a consistent patient-level dataset.

---

## 3. Cohort Construction

### 3.1 Unit of Analysis

The dataset was structured at the **patient level**, with one row per patient.

Given that the raw data contained multiple records per patient (e.g., multiple consultations and history entries), aggregation was required.

---

### 3.2 Patient Identification and Aggregation

Patients were uniquely identified using a consistent patient identifier across tables.

The cohort construction process involved:

- Linking clinical history records to consultations  
- Linking consultations to patients  
- Aggregating multiple records into a single patient-level representation  

---

### 3.3 Inclusion Criteria

Patients were included if:

- At least one valid clinical timestamp was available  
- A temporal sequence (index → event or censoring) could be constructed  

Patients with no valid temporal information were excluded.

---

### 3.4 Final Cohort

- **33,364 patients**
- **1 row per patient**
- Fully aligned across all integrated data sources

---

## 4. Temporal Framework

The temporal design of the dataset was the central component of the methodology.

---

### 4.1 Index Date Definition

Due to the absence of a standardized “start of follow-up” variable (e.g., first eGFR measurement or program enrollment), the index date was defined as:

> **The earliest valid clinical timestamp available per patient**

This was constructed using a hierarchical fallback strategy:

1. Clinical result date (when available)  
2. Clinical history start date  
3. Consultation date  

Formally:
```
index_date = MIN(COALESCE(result_date, history_start_date, consultation_date))
```
This ensured maximum coverage across patients while maintaining temporal consistency.

---

### 4.2 Censoring Date Definition

The censoring date was defined as:

> **The latest available clinical timestamp per patient**

This was derived from:

- The last observed clinical encounter  
- Or the last available record in the system  

This allows estimation of follow-up time for patients without observed events.

---

### 4.3 Temporal Consistency Checks

The following constraints were enforced:

- `censoring_date ≥ index_date` for all patients  
- Patients violating temporal ordering were removed or corrected  
- Missing index or censoring dates were minimized (<0.01%)

---

## 5. Outcome Definition

### 5.1 Outcomes Considered

The following outcomes were included:

- Renal loss  
- Cardiovascular event  
- Renal death  
- General mortality  
- Cardiovascular mortality  
- Hospitalization  

---

### 5.2 Event Consolidation

For each outcome:

- Patients could have multiple recorded events  
- Only the **earliest valid event date** was retained  

This ensures compatibility with time-to-event modeling.

---

### 5.3 Event Encoding

Each outcome was represented as:

- Binary indicator (`event = 1 or 0`)  
- Event date (if applicable)

---

## 6. Time-to-Event Construction

For each outcome, a time-to-event variable was computed:

- If `event = 1`:

  `t = event_date − index_date`

- If `event = 0`:

  `t = censoring_date − index_date`

Time was expressed in **years**.

---

### 6.1 Data Cleaning

During computation:

- Negative time intervals were identified and set to missing  
- Events without valid time were flagged and investigated  
- Final dataset ensured alignment between event indicators and time values  

---

## 7. Data Validation and Diagnostics

A series of validation steps were performed:

- Verification of event–time consistency (`event = 1` → valid `t`)  
- Detection of missing or inconsistent timestamps  
- Distribution analysis of time-to-event variables  
- Cross-checking cohort size after each transformation  

---

## 8. Washout Analysis (Critical Finding)

A major finding was the presence of a large proportion of events occurring at:

> **t = 0 (same day as index date)**

This was particularly pronounced for:

- Hospitalization (~86%)  
- Cardiovascular events (~50%)  
- Renal death (~28%)  

---

### 8.1 Interpretation

These events likely represent:

- Pre-existing conditions  
- Retrospectively recorded diagnoses  
- Administrative artifacts  

Rather than true incident events occurring after baseline.

---

### 8.2 Implication

This creates a fundamental ambiguity:

- Without adjustment, models would learn baseline status instead of future risk  
- Applying a washout period reduces bias but significantly decreases event counts  

---

### 8.3 Decision

No washout was applied at this stage.

The dataset was **intentionally frozen** pending clinical validation.

---

## 9. Outcome Viability Assessment

Outcomes were evaluated based on event frequency:

| Outcome | Events |
|--------|--------|
| Renal loss | 4211 |
| Cardiovascular event | 1872 |
| Renal death | 152 |
| General mortality | 165 |
| Cardiovascular mortality | 24 |
| Hospitalization | 45 |

---

### 9.1 Key Limitation

- Cardiovascular mortality (24 events) is not statistically viable for modeling  
- Hospitalization is also limited due to both low frequency and washout issues  

---

## 10. Variable Coverage and Sparsity

Although the dataset includes **102 variables**, many exhibit:

- High missingness  
- Inconsistent recording  
- Limited overlap across patients  

This limits their usability in predictive modeling.

---

## 11. Final Dataset Structure

- **33,364 patients**
- **102 variables**
- Multiple clinical domains
- Time-to-event variables for all outcomes
- Fully validated temporal structure

---

## 12. Decision to Halt Modeling

Model development was intentionally not pursued due to:

- Temporal ambiguity (washout problem)  
- Insufficient event counts for key outcomes  
- Risk of producing clinically misleading predictions  

> The decision prioritizes methodological rigor over premature modeling.

---

## 13. Next Steps

To enable modeling, the following are required:

- Clinical validation of washout definition  
- Access to additional datasets with outcome coverage  
- Potential redefinition of outcomes or composite endpoints  
- Feature selection based on variable coverage  

---

## 14. Summary

This project demonstrates the construction of a real-world clinical data pipeline under significant constraints, emphasizing:

- Temporal coherence  
- Outcome validity  
- Data quality assessment  
- Critical decision-making prior to modeling


