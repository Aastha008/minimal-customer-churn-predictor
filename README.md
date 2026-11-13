# Minimal Customer Churn Predictor

![Python Version](https://img.shields.io/badge/python-3.10%2B-blue.svg)
![Build Status](https://img.shields.io/badge/build-passing-brightgreen.svg)
![License](https://img.shields.io/badge/license-MIT-green.svg)
![Code Style](https://img.shields.io/badge/code%20style-black-000000.svg)

An end-to-end Machine Learning microservice designed to process historical customer behavior, feature usage, and billing telemetry to predict user churn risk. The system automatically engineers engagement metrics, trains lightweight classification models, exposes REST endpoints for inference, and generates visual performance dashboards.

---

## 🎯 Problem Statement & Target Audience

### The Problem
Acquiring new customers costs 5x to 25x more than retaining existing ones. Most subscription-based businesses fail to spot churn indicators—such as declining login frequency, dropping feature usage, or failed payment retries—until an account cancellation has already occurred. By the time a cancellation notice arrives, proactive retention efforts are ineffective.

### Target Audience
* **Customer Success & Growth Teams:** Needing automated risk scores and churn driver insights to target at-risk users with intervention campaigns.
* **Data & Analytics Engineers:** Looking for a standard, easy-to-deploy ML baseline microservice that integrates easily with existing data warehouses and APIs.
* **Product Managers:** Requiring insight into feature usage trends that correlate with user drop-off.

---

## ✨ Key Features

* **Batch Ingestion Pipeline:** Ingests user activity logs and billing data from CSV sources or SQL database queries.
* **Automated Feature Engineering:** Aggregates raw event logs into actionable customer health indicators (e.g., session frequency, feature interaction depth, 30-day activity trends).
* **RESTful Inference Engine:** High-performance API endpoints for real-time risk scoring by `user_id` and batch predictions.
* **Model Evaluation & Dashboarding:** Automatically builds classification metrics (Precision, Recall, F1, ROC-AUC) and outputs feature importance visualizations using Matplotlib.
* **Model Serialization:** Saves and reloads versioned models using Joblib for quick cold starts and lightweight deployment.

---

## 🏗️ Architecture & System Design

The application follows a modular, layer-separated architecture prioritizing decoupled feature engineering, robust Pydantic data schemas, and high-speed FastAPI endpoints.

```
                  ┌────────────────────────┐
                  │   Raw Activity Data    │
                  │   (CSV / SQL Queries)  │
                  └───────────┬────────────┘
                              │
                              ▼
                  ┌────────────────────────┐
                  │    Ingestion Engine    │
                  │     (src/core.py)      │
                  └───────────┬────────────┘
                              │
                              ▼
                  ┌────────────────────────┐
                  │ Feature Engineering    │
                  │  & Data Validation     │
                  │    (src/models.py)     │
                  └───────────┬────────────┘
                              │
         ┌────────────────────┴────────────────────┐
         ▼                                         ▼
┌─────────────────┐                       ┌──────────────────┐
│  Model Training │                       │ Prediction Service│
│  & Serialization│                       │ (FastAPI Core)   │
│ (Scikit-Learn/  │                       │  (src/api.py)    │
│     Joblib)     │                       └────────┬─────────┘
└────────┬────────┘                                │
         │                                         │
         ▼                                         ▼
┌─────────────────┐                       ┌──────────────────┐
│ Dashboard &     │                       │ REST Endpoint    │
│ Charting Module │                       │ /predict/{user}  │
│  (Matplotlib)   │                       └──────────────────┘
└─────────────────┘
```

### Module Responsibilities

* **`src/models.py`**: Defines immutable Pydantic schemas for log records, engineered feature vectors, risk prediction outputs, and model metrics.
* **`src/core.py`**: Contains data processing routines, feature calculation logic, Scikit-Learn training pipelines, model serialization via Joblib, and chart rendering routines.
* **`src/api.py`**: Exposes production-ready FastAPI endpoints for log ingestion, batch training execution, risk scoring, and binary metric assets.
* **`src/main.py`**: Initializes the FastAPI application instance, configures CORS/middleware policies, and boots the Uvicorn web server process.

---

## 🛠️ Technology Stack

| Component | Technology | Purpose |
| :--- | :--- | :--- |
| **Language** | Python 3.10+ | Core runtime environment |
| **API Framework** | FastAPI | High-speed, async REST web service |
| **Data Processing** | Pandas, NumPy | Data transformation and feature aggregation |
| **Machine Learning**| Scikit-Learn | Binary classification algorithms and metrics |
| **Validation** | Pydantic v2 | Data schema validation and typing |
| **Serialization** | Joblib | Model artifact saving and loading |
| **Visualization** | Matplotlib | Automated chart rendering for dashboards |
| **Web Server** | Uvicorn | ASGI server implementation |

---

## 📦 Installation & Setup

### Prerequisites
* Python 3.10 or higher
* `pip` and `venv` tools installed

### 1. Clone Repository & Navigate
```bash
git clone https://github.com/your-org/minimal-customer-churn-predictor.git
cd minimal-customer-churn-predictor
```

### 2. Set Up Virtual Environment
```bash
# macOS/Linux
python3 -m venv venv
source venv/bin/activate

# Windows (PowerShell)
python -m venv venv
.\venv\Scripts\Activate.ps1
```

### 3. Install Dependencies
```bash
pip install --upgrade pip
pip install -r requirements.txt
```

---

## ⚙️ Environment Variables Configuration

Create a `.env` file in the root directory by copying the provided template:

```bash
cp .env.example .env
```

Available environment variables:

| Variable | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `HOST` | String | `0.0.0.0` | Server binding address |
| `PORT` | Integer | `8000` | Application port |
| `MODEL_PATH` | String | `data/models/churn_model.pkl` | Storage location for serialized model artifact |
| `DATA_DIR` | String | `data/processed/` | Path for batch CSV/SQL log imports |
| `LOG_LEVEL` | String | `INFO` | Application logging verbosity (`DEBUG`, `INFO`, `WARNING`, `ERROR`) |

---

## 🚀 Usage Guide

### Starting the Web Service

Start the FastAPI application server using Uvicorn:

```bash
python -m src.main
```
Alternatively, launch via Uvicorn CLI directly:
```bash
uvicorn src.main:app --host 0.0.0.0 --port 8000 --reload
```

Interactive OpenAPI documentation is automatically available at:
`http://localhost:8000/docs`

---

### REST API Endpoints & Examples

#### 1. Ingest User Activity Logs
Ingest a batch of recent interaction and billing events into the processing pipeline.

* **Endpoint:** `POST /api/v1/ingest`
* **Content-Type:** `application/json`

**Sample Request Body:**
```json
[
  {
    "user_id": "usr_99812",
    "timestamp": "2026-03-25T10:15:30Z",
    "session_duration_sec": 480,
    "features_accessed": ["dashboard", "exports", "billing"],
    "support_tickets_count": 2,
    "payment_failed": false
  }
]
```

**cURL Example:**
```bash
curl -X POST "http://localhost:8000/api/v1/ingest" \
     -H "Content-Type: application/json" \
     -d '[
       {
         "user_id": "usr_99812",
         "timestamp": "2026-03-25T10:15:30Z",
         "session_duration_sec": 480,
         "features_accessed": ["dashboard", "exports"],
         "support_tickets_count": 2,
         "payment_failed": false
       }
     ]'
```

---

#### 2. Trigger Model Retraining
Trigger the automated feature engineering and model training pipeline.

* **Endpoint:** `POST /api/v1/train`

**cURL Example:**
```bash
curl -X POST "http://localhost:8000/api/v1/train"
```

**Response:**
```json
{
  "status": "success",
  "message": "Model trained successfully.",
  "metrics": {
    "accuracy": 0.912,
    "precision": 0.885,
    "recall": 0.841,
    "f1_score": 0.862
  }
}
```

---

#### 3. Fetch Single User Churn Risk
Get an instant risk evaluation for a target user ID.

* **Endpoint:** `GET /api/v1/predict/{user_id}`

**cURL Example:**
```bash
curl -X GET "http://localhost:8000/api/v1/predict/usr_99812"
```

**Response:**
```json
{
  "user_id": "usr_99812",
  "churn_risk_score": 0.742,
  "risk_level": "HIGH",
  "key_drivers": [
    "Decreased session frequency (-45% YoY)",
    "Recent increase in support tickets",
    "No key feature interaction in 14 days"
  ],
  "evaluated_at": "2026-03-30T14:22:00Z"
}
```

---

#### 4. Model Performance Dashboard
Retrieve visual performance metrics and feature rankings as an image file.

* **Endpoint:** `GET /api/v1/dashboard`
* **Response:** `image/png`

**cURL Example:**
```bash
curl -X GET "http://localhost:8000/api/v1/dashboard" --output dashboard.png
```

---

## 🧪 Automated Testing

Automated testing coverage spans data transformations, feature calculation correctness, and FastAPI endpoint execution.

### Run All Tests
```bash
pytest
```

### Run Module-Specific Test Suites
```bash
# Test Core Pipeline & Feature Engineering
pytest tests/test_core.py

# Test API Endpoints
pytest tests/test_api.py
```

### Run Tests with Coverage Reports
```bash
pytest --cov=src tests/ --cov-report=term-missing
```

---

## 📁 Project Structure

```
.
├── .env.example            # Environment configuration template
├── .gitignore              # Ignored files and system artifacts
├── requirements.txt        # Production dependencies list
├── src/
│   ├── __init__.py         # Package initialization
│   ├── api.py              # FastAPI route handlers & API logic
│   ├── core.py             # Feature engineering, ML logic & visualizations
│   ├── main.py             # App initialization & Uvicorn runner
│   └── models.py           # Pydantic schemas and data validation definitions
└── tests/
    ├── __init__.py         # Test package initialization
    ├── test_api.py         # Integration tests for FastAPI endpoints
    └── test_core.py        # Unit tests for ML algorithms and transformations
```

---

## 🛣️ Roadmap & Future Enhancements

* [ ] **SHAP Integration:** Incorporate SHAP (SHapley Additive exPlanations) for direct local model explainability at the individual user level.
* [ ] **Real-time Streaming:** Add Apache Kafka / AWS Kinesis integration for streaming event ingestion and low-latency feature stores.
* [ ] **Webhooks Support:** Allow automated dispatching of retention webhooks to CRM platforms (e.g., Salesforce, HubSpot, Customer.io) when high risk scores are detected.
* [ ] **A/B Model Shadowing:** Support side-by-side performance monitoring between production and challenger models before switching models.

---

## 📄 License

Distributed under the MIT License. See `LICENSE` for more information.