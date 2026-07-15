# 🚀 RiskPulse — Real-Time Project Risk & Delay Prediction System

RiskPulse is an AI-powered, full-stack enterprise analytics platform designed to predict project delay probabilities and monitor portfolio health in real time. It implements a complete machine learning system combining a Random Forest predictive pipeline with a modern, glassmorphic SaaS dashboard.

🔗 **[🌐 View Live Production Demo](https://riskpulse-ygtv.onrender.com)**

---

## 📖 Table of Contents
1. [📌 Project Overview](#-project-overview)
2. [📚 Detailed Technical Documentation](#-detailed-technical-documentation)
3. [🤖 Core Features](#-core-features)
4. [🛠️ Tech Stack](#️-tech-stack)
5. [📁 Workspace Structure](#-workspace-structure)
6. [⚙️ Quick Local Installation](#️-quick-local-installation)
7. [👨‍💻 Author Info & Support](#-author-info--support)

---

## 📌 Project Overview

RiskPulse helps project managers make data-driven decisions by translating project metrics into predictive outcomes. 
*   **Predict Risk in Real-Time**: Evaluates scope progression, burn rates, and team parameters to determine the probability of project delay.
*   **Actionable Insights**: Explains risk metrics through tree-derived features and logical diagnostics.
*   **Real-Time Analytics**: Generates trend curves, portfolio metrics, and risk heatmaps.

---

## 📚 Detailed Technical Documentation

For in-depth analysis of the system components, explore the specialized sub-documents:

```
                  ┌────────────────────────┐
                  │       README.md        │
                  │  (Landing & Overview)  │
                  └───────────┬────────────┘
                              │
         ┌────────────┬───────┴────────┬────────────┐
         ▼            ▼                ▼            ▼
 ┌──────────────┐ ┌──────────────┐ ┌──────────┐ ┌──────────────┐
 │ ARCHITECTURE │ │     API      │ │ MACHINE  │ │ DEVELOPMENT  │
 │   .md        │ │ DOCUMENTATION│ │ LEARNING │ │   .md        │
 │              │ │     .md      │ │   .md    │ │              │
 └──────────────┘ └──────────────┘ └──────────┘ └──────────────┘
```

*   **[🏛️ System Architecture (`ARCHITECTURE.md`)](file:///c:/Users/GURKIRAT%20SINGH/OneDrive/Desktop/2nd/simple%20projects/code_relay/project_risk_system/ARCHITECTURE.md)**: Deep dive into application component relationships, SQLAlchemy database models, E-R diagrams, security policies, and rate-limiting structures.
*   **[📡 API Specification (`API_DOCUMENTATION.md`)](file:///c:/Users/GURKIRAT%20SINGH/OneDrive/Desktop/2nd/simple%20projects/code_relay/project_risk_system/API_DOCUMENTATION.md)**: Complete guide to all Flask routes, setting controllers, payload schemas, input validations, and integration code.
*   **[🧠 Machine Learning Pipeline (`MACHINE_LEARNING.md`)](file:///c:/Users/GURKIRAT%20SINGH/OneDrive/Desktop/2nd/simple%20projects/code_relay/project_risk_system/MACHINE_LEARNING.md)**: Specifications of the Random Forest model, mathematical formulas for the 12 engineered features, explainability rules, and retraining processes.
*   **[🛠️ Setup & Operations Runbook (`DEVELOPMENT.md`)](file:///c:/Users/GURKIRAT%20SINGH/OneDrive/Desktop/2nd/simple%20projects/code_relay/project_risk_system/DEVELOPMENT.md)**: Standard developer setup, environment configurations, migrations guides, WSGI deployments, and quick troubleshooting fixes.

---

## 🤖 Core Features

*   **🔒 Secure SaaS Workspace**: User accounts protected by `Flask-Bcrypt` password hashing and session cookies hardened with `HttpOnly` and `SameSite` flags.
*   **🧠 Random Forest Prediction**: Instant predictions using a custom-engineered pipeline linked to an optimized machine learning model.
*   **📊 Interactive Glassmorphism Dashboard**: Smooth, responsive UI featuring dynamic Chart.js timeseries graphs, circular gauges with custom counters, and detailed KPI cards.
*   **🔑 API Key Management Manager**: Generates hashed API credentials directly from the settings interface for seamless programmatic integrations.
*   **📈 Manager AI Summary**: Calculates trend shifts, highlights high-risk outliers, and automatically outputs recommendations for resources.

---

## 🛠️ Tech Stack

*   **Backend**: Flask, Flask-SQLAlchemy, Flask-Login, Flask-Bcrypt, Flask-Migrate, Flask-Limiter.
*   **Machine Learning**: scikit-learn (Random Forest Classifier), NumPy, Joblib.
*   **Frontend**: HTML5, CSS3 (Vanilla Glassmorphic design), Vanilla JavaScript.
*   **Visualizations**: Chart.js (SVG circular gauges and dynamic lines).
*   **Databases**: SQLite (Development) and PostgreSQL (Production hosting via Render).
*   **Web Server**: Gunicorn (Production WSGI).

---

## 📁 Workspace Structure

```
project_risk_system/
│
├── app.py                      # Main Flask application & routes controller
├── train_model.py              # Baseline scikit-learn model compiler script
├── retrain_model.py            # Environment-aware retraining script
├── add_colums.py               # Manual sqlite schema migration script
├── risk_model.pkl              # Compiled Random Forest model binary
├── project_data.csv            # Original training project dataset
├── requirements.txt            # Python dependencies configuration
├── procfile                    # Render/Heroku WSGI application runner
│
├── static/                     # Assets directory (CSS, JS, Avatars)
│   ├── style.css               # Main dashboard stylesheet
│   ├── profile-style.css       # Profile settings stylesheet
│   ├── auth.css                # Authentication page styles
│   └── script.js               # Frontend fetch and rendering logic
│
├── templates/                  # Jinja2 HTML layout pages
│   ├── dashboard.html          # Main UI view
│   ├── analytics.html          # Reports and distributions charts
│   ├── profile.html            # Profile info timeline page
│   └── settings.html           # Settings, API key manager, passwords
│
└── logs/                       # Rotating production logs
    └── app.log                 # Server activity logging target
```

---

## ⚙️ Quick Local Installation

To run the application locally in under 2 minutes:

1.  **Clone & Navigate**:
    ```bash
    git clone https://github.com/your-username/project-risk-system.git
    cd project-risk-system
    ```
2.  **Initialize Environment & Install Packages**:
    ```bash
    python -m venv venv
    # Windows: .\venv\Scripts\Activate.ps1
    # Unix: source venv/bin/activate
    pip install -r requirements.txt
    ```
3.  **Prepare local model binary**:
    ```bash
    python retrain_model.py
    ```
4.  **Run Application**:
    ```bash
    python app.py
    ```
    *Navigate to `http://localhost:5000` to access the dashboard.*

For database migrations or troubleshooting, please refer to the **[Setup & Operations Runbook (`DEVELOPMENT.md`)](file:///c:/Users/GURKIRAT%20SINGH/OneDrive/Desktop/2nd/simple%20projects/code_relay/project_risk_system/DEVELOPMENT.md)**.

---

## 👨‍💻 Author Info & Support

*   **Author**: Gurkirat Singh Bhangoo
*   **Version**: 2.0.0 (Enhanced Edition)
*   **License**: MIT License (See `License.txt` for details)

*⭐ Star this repo if you find it helpful for learning or deploying end-to-end full-stack machine learning applications!*