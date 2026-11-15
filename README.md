# Vitality Readmission Predictor

A Python-based predictive model for patient risk stratification and hospital readmission forecasting, developed for Vitality Health Network (ITS 2122 Group Project).

This consolidated pipeline (`app.ipynb`) combines four analytical phases into a single end-to-end workflow that ingests, cleans, enriches, explores, and scores diabetic patient encounter data to predict the likelihood of hospital readmission.

## Project Overview

The **Vitality Readmission Predictor** analyzes a clinical diabetes dataset (101,766 encounters × 50 features) and engineers a **Vitality Complexity Index (VCI)** — a LACE-inspired risk score that stratifies patients into Low, Medium, and High readmission risk categories prior to discharge.

### Pipeline Phases

| Phase | Description | Source Notebook |
|-------|-------------|-----------------|
| **Phase 1** | Clinical Data Ingestion & Cleaning | `phase_o1.ipynb` |
| **Phase 2** | Data Enrichment via Web Scraping (ICD-9 descriptions) | `phase_o1.ipynb` |
| **Phase 3** | Exploratory Data Analysis & Clinical Visualizations | `phase03.ipynb` |
| **Phase 4** | Feature Engineering — Vitality Complexity Index (VCI) | `phase_04.ipynb` |

## Repository Structure

```
Vitality-Readmission-Predictor/
├── app.ipynb                 # Consolidated pipeline notebook (all 4 phases)
├── README.md                 # Project documentation
└── data_files/               # Dataset directory
    ├── diabetic_data.csv          # Raw clinical dataset (101,766 × 50)
    ├── diabetic_data_enriched.csv # Enriched with ICD-9 diagnosis descriptions
    ├── diabetic_data_vci.csv      # Enriched with VCI scores & risk strata
    └── IDs_mapping.csv            # Encounter/patient ID mapping reference
```

## Dataset

The raw dataset (`diabetic_data.csv`) contains **101,766 hospital encounters** with **50 features**, including:

- **Identifiers:** `encounter_id`, `patient_nbr`
- **Demographics:** `race`, `gender`, `age`, `weight`
- **Admission info:** `admission_type_id`, `discharge_disposition_id`, `admission_source_id`, `time_in_hospital`
- **Utilization:** `num_lab_procedures`, `num_procedures`, `num_medications`, `number_outpatient`, `number_emergency`, `number_inpatient`, `number_diagnoses`
- **Diagnoses:** `diag_1`, `diag_2`, `diag_3` (ICD-9 codes)
- **Glucose indicators:** `max_glu_serum`, `A1Cresult`
- **Medications:** 22 drug columns (e.g., `metformin`, `insulin`, `glyburide-metformin`, etc.)
- **Target:** `readmitted` — `{NO, >30, <30}` (whether the patient was readmitted within 30 days)

Missing values are encoded as `"?"` in the raw CSV and handled during cleaning.

## Getting Started

### Prerequisites

Python 3.8+ with the following packages:

```bash
pip install pandas numpy matplotlib seaborn requests beautifulsoup4
```

### Running the Pipeline

1. Clone or open this repository.
2. Ensure the `data_files/diabetic_data.csv` dataset is present.
3. Open `app.ipynb` in Jupyter Notebook, JupyterLab, or VS Code.
4. Run all cells top-to-bottom to execute the full four-phase pipeline.

Outputs are written back to `data_files/`:
- `diabetic_data_enriched.csv` — after Phase 2 (ICD-9 enrichment)
- `diabetic_data_vci.csv` — after Phase 4 (VCI scores & risk stratification)

## Phase Details

### Phase 1: Clinical Data Ingestion & Cleaning
- Loads `diabetic_data.csv` using `pandas.read_csv()` with `"?"` treated as missing values.
- Performs initial dataset assessment (shape, dtypes, descriptive statistics).
- Identifies and converts incorrect data types (identifier columns to categorical).
- Replaces `"?"` placeholders with `NaN`.
- Removes deceased patients (expired discharge disposition IDs).
- Removes exact duplicate records.
- Final cleaned dataset: **100,114 encounters**.

### Phase 2: Data Enrichment via Web Scraping
- Identifies the top 20 most frequent ICD-9 diagnosis codes in `diag_1`.
- Scrapes human-readable clinical descriptions from [icd9.chrisendres.com](http://icd9.chrisendres.com) using `requests` and `BeautifulSoup`.
- Includes a fallback dictionary for offline reliability.
- Adds a `Primary_Diagnosis_Desc` column; non-top-20 codes are labeled `"Other"`.

### Phase 3: Exploratory Data Analysis & Clinical Visualizations
- Statistical and visual exploration of the cleaned, enriched dataset.
- Clinical visualizations to understand readmission patterns and risk factors.

### Phase 4: Feature Engineering — Vitality Complexity Index (VCI)
The VCI is a LACE Index-inspired risk score computed per encounter as:

**VCI_Score = L + A + C + E**

| Component | Metric | Scoring |
|-----------|--------|---------|
| **L** — Length of Stay | `time_in_hospital` (days) | 1–4 days: 1 pt; 5–13 days: 4 pts; 14+: 7 pts |
| **A** — Acuity | `admission_type_id` | Emergency/urgent (1, 7): 3 pts; else: 0 pts |
| **C** — Comorbidities | `number_diagnoses` | <4: 0 pts; 4–7: 3 pts; 8+: 5 pts |
| **E** — Emergency Visits | `number_emergency` | 0: 0 pts; 1–4: 3 pts; 5+: 5 pts |

Patients are stratified into:
- **Low Risk**
- **Medium Risk**
- **High Risk**

The model is validated by comparing 30-day readmission rates (`readmitted == "<30"`) across each risk stratum.

## Key Technologies

- **Python 3**
- **pandas** — data manipulation and analysis
- **NumPy** — numerical computing
- **matplotlib / seaborn** — data visualization
- **requests / BeautifulSoup** — web scraping of ICD-9 descriptions
- **pathlib** — filesystem path handling

## License

This project was developed for academic purposes as part of the ITS 2122 course (Vitality Health Network group project).
