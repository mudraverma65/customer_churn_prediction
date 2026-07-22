## Overview
This repository contains my solution for the case study on **2-year customer churn prediction**.

The objective was to:
1. Build an optimized, scalable modeling pipeline on a large dataset (1M rows),
2. Use a robust validation strategy,
3. Select a **business-optimal decision threshold** based on asymmetric error costs:
   - False Negative (missed churner): **$5,000**
   - False Positive (unnecessary retention offer): **$200**
4. Provide a scalable GCP architecture approach for future growth.

---

## Repository Structure

```text
customer-churn-prediction/
├── README.md
├── requirements.txt
├── .gitignore
│
├── data/
│   └── README.md
│
├── notebooks/
│   └── churn_model.ipynb
│
├── outputs/
│   ├── metrics/
│   │   ├── model_probability_metrics.csv
│   │   ├── threshold_cost_table.csv
│   │   └── business_summary.csv
│   └── plots/
│       ├── roc_curve.png
│       ├── pr_curve.png
│       └── cost_vs_threshold.png
│
└── slides/
    └── Executive_Slide.pptx
```

> Note: The dataset is not committed to GitHub due to size and source terms.  
> Place the downloaded CSV in `data/` locally.

---

## Dataset
Source (Kaggle):  
[Customer Churn Prediction Dataset (1M)](https://www.kaggle.com/datasets/isandeep06/customer-churn-prediction-dataset-1m/data)

Expected local path:
- `data/customer_churn_1m.csv`

---

## Environment Setup

```bash
python -m venv .venv
source .venv/bin/activate   # macOS/Linux
# .venv\Scripts\activate    # Windows

pip install -r requirements.txt
```

Launch notebook:
```bash
jupyter notebook
```
Then open:
- `notebooks/bell_churn_model.ipynb`

---

## Method Summary

### 1) Data Pipeline & Optimization
- Vectorized preprocessing (no row-wise loops)
- Missing value handling:
  - Numeric: median imputation
  - Categorical: most frequent imputation
- Outlier treatment: vectorized IQR capping
- Feature engineering from `signup_date`

### 2) Validation Strategy
- **Time-based split** (train on earlier period, validate on later period)
- Chosen to reflect production forecasting and reduce temporal leakage risk

### 3) Model + Business Thresholding
- Baseline and tuned sparse linear models
- Selected final model: **Calibrated Logistic Regression**
- Threshold selected by minimizing:
  \[
  \text{Total Cost} = 5000 \cdot FN + 200 \cdot FP
  \]

---

## Final Results (Validation)
- **Winner:** `CALIBRATED(logit_C=0.5_cw=balanced)`
- ROC-AUC: **0.6826**
- PR-AUC: **0.2043**
- Optimal threshold: **0.05**

### Business Impact
- Cost at default threshold (0.50): **$6,585,000**
- Cost at optimized threshold (0.05): **$2,302,200**
- **Estimated savings: $4,282,800 (~65%)**

---

## Deliverables Included
- ✅ Reproducible notebook (`notebooks/churn_model.ipynb`)
- ✅ Executive one-slide summary (`slides/Executive_Slide.pdf`)
- ✅ Validation strategy and GCP architecture answers (inside notebook markdown cells)
- ✅ Output metrics and plots (`outputs/`)

---

## GCP Scale-Up Architecture (100M rows)
At 100M rows, I would use a GCP-native pipeline:
- Ingestion: **Cloud Storage**
- Transformation/feature engineering: **BigQuery** (and **Dataflow** where needed)
- Training + registry + experiments: **Vertex AI**
- Batch scoring orchestration: **Vertex Pipelines** or **Cloud Composer**
- Storage of predictions: **BigQuery**
- Optional real-time serving: **Vertex AI Endpoint**
- Monitoring: data quality, drift/calibration, and business KPIs

---