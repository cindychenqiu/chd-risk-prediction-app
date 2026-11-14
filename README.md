# Coronary Heart Disease Risk Prediction

End-to-end application that estimates the 10-year risk of Coronary Heart Disease (CHD) from clinical markers in a Framingham-based coronary heart disease dataset, from exploratory analysis and model comparison to a FastAPI backend and a React web interface.

Group project for the Scientific Programming course of the MSc in Health Data Science (MHEDAS), Universitat Rovira i Virgili.

## Overview

Cardiovascular diseases are the leading cause of death worldwide, and early risk assessment helps prevent complications such as myocardial infarction and heart failure. This project builds a model that predicts whether a person will develop CHD within ten years, and turns it into a web application where a user enters clinical values and receives a risk level.

The dataset has 4238 records and 16 variables, with about 15% positive cases. The final model uses eight features: BMI, hypertension, pulse pressure, cigarettes per day, total cholesterol, glucose, heart rate and an age group code.

## Results

The best traditional model was a Support Vector Machine on standard-scaled data with class weights, which is the model served by the API. Boosted and stacked models were also tested to increase recall. All values come from the held-out test set (848 samples, 15.2% positive cases).

| Model | Data scaling | Accuracy | Precision | Recall | F1 | ROC-AUC |
|-------|--------------|----------|-----------|--------|----|---------|
| SVM with class weights (deployed) | Standard | 0.6698 | 0.2525 | 0.5969 | 0.3548 | 0.6897 |
| Stacking ensemble (Stacking_ET) | Standard | 0.6050 | 0.2359 | 0.7132 | 0.3545 | 0.6832 |
| AdaBoost | Robust | 0.4929 | 0.2032 | 0.7985 | 0.3239 | 0.6780 |

Key findings:

- The SVM gave the best balance of F1 and ROC-AUC, so it was chosen for deployment.
- Boosting and stacking raise recall to 0.71 to 0.80, but precision stays low, so most positive alerts are false positives.
- Discrimination is modest (ROC-AUC about 0.7), which is typical for this dataset. The application is an educational demonstration of the full pipeline, not a diagnostic tool.

[Video demo of the application](docs/demo.mp4) | [Web app](https://sp-frontend-qtk3.onrender.com) | [API](https://sp-backend-qkjs.onrender.com) (free hosting, so the first load can take about a minute)

## Data

The dataset was provided by the course instructor. The original CSV (`coronary_disease.csv`, 4238 rows and 16 columns) and a data dictionary are in `data/raw/`. The cleaned dataset, engineered features, train-test splits (unscaled, standard-scaled and robust-scaled) and model comparison tables are in `data/processed/`.

## Methods

1. **Cleaning:** snake_case column names, type fixes, median or mode imputation for variables with under 5% missing values, KNN imputation for glucose (9.2% missing), and outliers profiled with IQR and z-scores but kept because they are clinically plausible.
2. **Feature engineering:** pulse pressure, BMI category, age groups and risk factor counts, with multicollinearity checked through the variance inflation factor.
3. **Baseline models:** Logistic Regression, KNN, Decision Tree, Random Forest and SVM on unscaled, standard-scaled and robust-scaled data, handling class imbalance with class weights and SMOTE (inside cross-validation only), and selecting by F1 and ROC-AUC.
4. **Boosting and stacking:** XGBoost, LightGBM and AdaBoost tuned with Optuna, plus two stacked ensembles, with decision threshold optimization to raise recall.
5. **Deployment:** the SVM and its scaler are served by a FastAPI backend, a React (Vite and Tailwind CSS) frontend shows color-coded risk levels, and both run with Docker Compose and on Render.

The full methodology, results and discussion are in [`docs/chd_prediction_report.pdf`](docs/chd_prediction_report.pdf).

## Repository structure

```
.
├── backend/              FastAPI service with the production model and feature logic
├── data/
│   ├── processed/        Cleaned data, splits and model comparison tables
│   └── raw/              Original dataset and data dictionary
├── docs/
│   └── chd_prediction_report.pdf
├── figures/              Plots for each phase
├── frontend/             React and Vite application
├── models/               Saved models, scaler and metadata
├── notebooks/
│   ├── 01_EDA.ipynb
│   ├── 02_ML_Models.ipynb
│   ├── 03_Model_Selection.ipynb
│   └── 04_API_Dev_and_Deploy.ipynb
├── src/                  Preprocessing and modeling helpers
├── create_scaler.py      Regenerates the scaler if it is missing
├── docker-compose.yml
├── main.py
├── README.md
└── render.yaml

```

## How to run

**Docker Compose (frontend and backend):**

```bash
git clone https://github.com/cindychenqiu/chd-risk-prediction-app.git
cd chd-risk-prediction-app
docker-compose up --build
```

The frontend is at `http://localhost:3000` and the backend at `http://localhost:8000`.

**Backend only:**

```bash
cd backend
python -m venv .venv
source .venv\Scripts\activate    # on macOS/Linux .venv/bin/activate 
pip install -r requirements.txt
python ../create_scaler.py       # only if models/scaler_standard.pkl is missing
uvicorn api.main:app --reload --host 0.0.0.0 --port 8000
```

**Frontend only:**

```bash
cd frontend
npm install
npm run dev
```

The frontend runs at `http://localhost:5173` and calls the API at `http://localhost:8000`.

**Notebooks:** open them in order (01 to 04) to regenerate the figures and artifacts.

**API endpoints:**

| Endpoint | Description |
|----------|-------------|
| `GET /` and `GET /health` | Service and model status |
| `POST /predict` | Takes age, systolic_bp, diastolic_bp, bmi, heart_rate, total_cholesterol, glucose, cigarettes_per_day and hypertension, and returns `prediction`, `probability` and `risk_level` (Low, Medium or High) |

Pulse pressure and the age group are computed by the API from the raw inputs.

## Limitations and next steps

- **Modest predictive performance:** ROC-AUC is about 0.69 and precision about 0.25, so the tool is a demonstration and must not be used for clinical decisions.
- **Single cohort:** the training data covers a specific population, which limits how well the model generalizes to others.
- **Class imbalance:** only about 15% of records are positive, which forces a trade-off between recall and precision.
- **Interpretability:** SHAP analysis was used, but the non-linear SVM needs further validation to be clinically transparent.
- **Next steps:** longitudinal data and more biomarkers, such as genetic risk scores, to improve precision.

## Authors and acknowledgements

Group project by Zaid Al Hakioui El Mettioui, Gemma Esteve Valls, Keith Megara
Osores Vicuña, Carlos Pérez de Arenaza García, Maria Sánchez Sánchez, Kamela Xhengo and Cindy Chen.  
  
[LinkedIn](https://www.linkedin.com/in/cindychenqiu/) | [GitHub](https://github.com/cindychenqiu)