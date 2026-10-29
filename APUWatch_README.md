<div align="center">

# ✈️ APU Watch
### Aircraft APU Predictive Maintenance & Remaining Useful Life (RUL) Estimation

**A machine-learning pipeline and dashboard for estimating the Remaining Useful Life of aircraft engine/APU systems from cycle-based sensor data.**

![Python](https://img.shields.io/badge/Python-3.11-blue)
![LightGBM](https://img.shields.io/badge/Model-LightGBM-green)
![Flask](https://img.shields.io/badge/Web-Flask-black)
![Docker](https://img.shields.io/badge/Deployment-Docker-blue)
![Google Cloud](https://img.shields.io/badge/Cloud-Google%20Cloud-orange)

</div>

## Project Overview

**APU Watch** is a software-based predictive-maintenance project focused on estimating the **Remaining Useful Life (RUL)** of aircraft engine/APU systems from cycle-based sensor readings. It combines a Python machine-learning pipeline with a Flask web application where users can upload a CSV file and view model predictions and evaluation metrics.

The project uses the **NASA C-MAPSS FD002 turbofan engine simulation dataset** as its starting point. It includes preprocessing, feature engineering, engine-level train/test splitting, a LightGBM regression model, inference, prediction reports, and supporting MLOps components for data/model versioning and cloud deployment.

The project is designed for research and experimentation. It is **not connected to live aircraft sensors**, and its predictions should not be used for real aircraft maintenance decisions without rigorous validation and aviation safety approval.

## Problem Statement

Aircraft engines operate over many cycles, and their sensor readings can change as operating conditions and component health change. Estimating how many cycles may remain before the end of an observed engine run can help researchers explore data-driven maintenance approaches.

APU Watch uses historical/simulated sensor data to learn relationships between operating settings, sensor behavior, and an RUL target. It produces an estimated RUL for each input row/cycle and reports how closely predictions match the target values available in the evaluation data.

## What the Project Includes

- **CSV-based inference:** Upload a CSV containing engine IDs, cycle numbers, operating settings, and sensor readings.
- **Data preparation:** Sort and validate input records and apply preprocessing consistent with the modeling pipeline.
- **Feature engineering:** Operating-condition normalization, cycle normalization, and rolling-window statistics/trend features.
- **Engine-level data splitting:** Keep complete engine histories in either the training or test set to reduce leakage between sets.
- **Fault-injection experiments:** Generate synthetic sensor fault patterns such as gradual drift, bias shift, increased noise, spikes, and coupled faults.
- **RUL regression:** Train a LightGBM regressor to estimate remaining cycles.
- **Model evaluation:** Calculate regression metrics including MSE, MAE, RMSE, and R².
- **Web dashboard:** Flask application with a CSV upload workflow and charts for prediction results and model metrics.
- **Prediction and drift monitoring components:** Sensor-baseline statistics and drift-related logic are included in the codebase.
- **MLOps and deployment components:** MLflow/DVC dependencies, Docker configuration, Google Cloud integration code, Cloud SQL schema, and a GitHub Actions deployment workflow are included. Actual cloud deployment requires configuration and credentials.

## Machine Learning Approach

### Dataset and input data

The project is configured around NASA C-MAPSS FD002-style engine run-to-failure data. The expected input schema contains:

- `engine_id` — engine identifier
- `cycle` — operating cycle number
- `op_setting_1`, `op_setting_2`, `op_setting_3` — operating settings
- `sensor_1` through `sensor_21` — sensor measurements

The preprocessing and modeling code selects a subset of informative sensor columns for engineered features. Input files must follow the expected schema.

### Fault injection

The code includes synthetic fault-injection experiments to explore model behavior under altered sensor patterns:

| Fault pattern | Example affected sensor(s) | Purpose |
|---|---|---|
| Gradual drift | `sensor_2`, `sensor_7` | Simulate a gradual change in readings |
| Bias shift | `sensor_4`, `sensor_8` | Simulate an offset in readings |
| Increased noise | `sensor_3` | Simulate noisier measurements |
| Intermittent spikes | `sensor_9` | Simulate short-lived abnormal readings |
| Coupled fault | `sensor_11`, `sensor_12` | Simulate related changes across sensors |

These are **synthetic experimental patterns**, not verified physical failure signatures.

### Feature engineering

The pipeline includes feature transformations such as:

- **Operating-condition normalization:** Center sensor values within groups of similar operating settings.
- **Cycle normalization:** Represent cycle position relative to the maximum observed cycle in an engine history.
- **Rolling-window features:** Use recent rolling means, rolling standard deviations, and changes/trends in sensor readings.
- **Scaling:** Apply a saved `StandardScaler` to the relevant features.

The implementation aims to keep feature processing consistent between training and inference.

### Target and model

The main model is a **LightGBM Regressor (`LGBMRegressor`)**. The target is RUL, represented in the code as a remaining-cycle estimate. The configured training target is generated from the observed engine history; in particular, `max_cycle - cycle` is used as a proxy target in parts of the inference workflow.

**Important limitation:** If an uploaded engine history is incomplete, calculating `max_cycle - cycle` from that file only measures cycles remaining within the supplied window. It does not independently reveal the engine's true future time-to-failure. A valid real-world RUL estimate requires an appropriate target definition and representative validation data.

## Evaluation

The pipeline computes:

- **MSE (Mean Squared Error):** Average squared prediction error.
- **MAE (Mean Absolute Error):** Average absolute prediction error.
- **RMSE (Root Mean Squared Error):** Error magnitude expressed in the target's units.
- **R²:** How well predictions explain target variation relative to a baseline.

The repository contains timestamped validation reports. Results differ between runs and samples, so metrics should always be reported with the dataset, split, and evaluation procedure used. Do not interpret one stored report as a guaranteed generalization result or as evidence of operational aircraft readiness.

## Application Workflow

1. Prepare a CSV file in the expected schema.
2. Open the APU Watch web application.
3. Upload the CSV file through the **Load Flight Sensor Data** interface.
4. The Flask backend calls the inference pipeline.
5. The pipeline validates and preprocesses the records, loads the saved scaler/model, and estimates RUL.
6. The application returns predictions and calculated evaluation metrics, and the dashboard visualizes the results.
7. When configured for Google Cloud, the application includes code paths to store uploads in Cloud Storage, record predictions in Cloud SQL, and publish custom monitoring metrics.

## Technology Stack

| Area | Technologies |
|---|---|
| Language | Python |
| Data processing | Pandas, NumPy |
| Machine learning | LightGBM, scikit-learn |
| Preprocessing | StandardScaler |
| Visualization | Chart.js, Matplotlib, Seaborn |
| Web application | Flask, Flask-CORS, HTML, CSS, JavaScript |
| Model/artifact storage | Joblib, local artifact files |
| Experiment/data versioning | MLflow, DVC |
| Containerization | Docker |
| Cloud components | Google Cloud Storage, Cloud Run, Secret Manager, Cloud Monitoring |
| Database | PostgreSQL / Google Cloud SQL |
| Automation | GitHub Actions |
| Testing | pytest |

Some cloud and automation features require additional configuration and have not been independently verified as deployed merely by being present in the repository.

## Repository Structure

```text
APU_Predictive_Maintenance_System/
├── Artifacts/
│   ├── Model/                 # Saved LightGBM model
│   ├── Scaler/                # Saved feature scaler
│   ├── raw/                   # Raw dataset sample
│   └── model_validations/     # Timestamped predictions and metrics
├── NoteBook/                  # Experiments and feature-engineering notebooks
├── Utils/
│   ├── Exception/             # Custom exception handling
│   └── Logging/               # Logging utilities
├── config/
│   └── config.yaml            # Data, feature, and model configuration
├── entrypoint/
│   ├── data_split.py
│   ├── feature_engineering.py
│   ├── training.py
│   ├── inference.py
│   └── retrain.py
├── frontend/
│   ├── app.py                 # Flask application and API routes
│   ├── templates/             # Dashboard HTML
│   └── static/                # CSS and JavaScript
├── infra/
│   └── schema.sql             # PostgreSQL tables for predictions/monitoring
├── src/
│   ├── Data_preprocessing/
│   ├── Pipelines/
│   └── monitoring/
├── tests/                     # pytest tests
├── Dockerfile
├── requirements.txt
└── README.md
```

## Getting Started Locally

### Prerequisites

- Python 3.11 recommended
- Git
- A compatible input CSV and the model/scaler artifacts required by the inference pipeline

### Installation

```bash
git clone <YOUR_GITHUB_REPOSITORY_URL>
cd APU_Predictive_Maintenance_System

python -m venv .venv
```

Activate the environment:

**Windows PowerShell**
```powershell
.\.venv\Scripts\Activate.ps1
```

**macOS/Linux**
```bash
source .venv/bin/activate
```

Install dependencies:

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### Run the web application

From the repository root:

```bash
python frontend/app.py
```

Open the local URL printed by Flask in your browser (the Docker configuration is set up for port `8080`).

> **Before running:** Confirm that the configured model and scaler files exist under `Artifacts/Model/` and `Artifacts/Scaler/`, and that any required DVC-managed data/artifacts have been pulled. The exact training and inference paths are defined in `config/config.yaml`.

### Run the pipeline entry points

Run these from the repository root, after verifying the data paths in `config/config.yaml` and that required input files are present:

```bash
python entrypoint/data_split.py
python entrypoint/feature_engineering.py
python entrypoint/training.py
```

The inference entry point accepts a CSV path; check its command-line options before running:

```bash
python entrypoint/inference.py --help
```

### Run tests

```bash
pytest tests/ -v
```

## Docker and Cloud Deployment

A `Dockerfile` and a GitHub Actions workflow are included. The workflow is configured to run tests, build and push a Docker image to Google Artifact Registry, and deploy to Google Cloud Run on pushes to `main`.

Cloud deployment requires the relevant Google Cloud project, service account, Artifact Registry repository, IAM permissions, secrets, buckets, database, and environment variables to be configured. The workflow file is a deployment configuration, not proof that a live deployment is currently available.

## Limitations and Responsible Use

- This is a research and software-development project, not a certified aircraft maintenance tool.
- The model estimates RUL from the data and target construction available to it; it does not directly confirm a physical engine fault.
- Synthetic injected faults may not represent real aircraft failure modes.
- Dataset-derived cycle estimates are not the same as verified remaining service life.
- Validation results can vary; report the test design and metrics transparently.
- Real-world use would require representative operational data, domain-expert review, rigorous validation, and applicable aviation approvals.

## Future Improvements

- Validate the model on clearly defined, held-out engine histories and report reproducible metrics.
- Improve target construction and assess the limitations of proxy RUL labels.
- Test robustness to realistic sensor noise and fault patterns.
- Add model explainability and clearer prediction uncertainty.
- Strengthen automated tests and reproducible training.
- Complete and verify cloud monitoring, scheduled retraining, and model promotion in a configured environment.
- Investigate Indian operating-condition features only when reliable supporting data is available.

## Acknowledgment

This project uses the NASA C-MAPSS turbofan engine simulation dataset as its data foundation. Refer to the original NASA dataset documentation and terms when redistributing data or derived files.

---

**APU Watch** — Exploring machine-learning-based remaining useful life estimation from aircraft engine sensor data.
