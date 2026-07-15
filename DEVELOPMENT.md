# Local Setup & Development Runbook — RiskPulse

This document provides step-by-step setup guides, database migration routines, execution commands, and troubleshooting workflows for developers running and maintaining the RiskPulse application.

---

## 📋 Prerequisites

Ensure your development machine has the following tools installed:
*   **Python 3.9+**
*   **pip** (Python package installer)
*   **Git** (for version control)
*   **SQLite** (used for local development) or **PostgreSQL** (production)

---

## 🛠️ Step-by-Step Local Setup

### 1. Initialize Virtual Environment
Navigate to the root of the project directory and create a virtual environment to isolate project dependencies:

```powershell
# Create virtual environment named 'venv'
python -m venv venv

# Activate on Windows (PowerShell)
.\venv\Scripts\Activate.ps1

# Activate on macOS/Linux
source venv/bin/activate
```

### 2. Install Project Dependencies
Install the required packages listed in `requirements.txt`:

```bash
pip install -r requirements.txt
```

### 3. Configure Environment Variables
Create a file named `.env` in the root folder of the project. Add the following parameters:

```ini
# Flask configuration
SECRET_KEY=your-custom-secure-random-secret-key-string
FLASK_ENV=development

# Database setting (Omitting this defaults to local SQLite instance/database.db)
# DATABASE_URL=postgresql://user:password@localhost:5432/riskpulse_db

# Custom Model Scaling Defaults (Optional)
DEFAULT_BUDGET=200000
DEFAULT_TASKS=100
DEFAULT_DELAY_RATE=0.3
```

---

## 💾 Database Schema Upgrades & Migrations

If the database is being initialized for the first time or the models are updated, you must apply schemas to the SQL database. You can do this using one of two methods:

### Method A: Flask-Migrate (Recommended for PostgreSQL / Production)
Flask-Migrate provides complete version history for database schema modifications.

```bash
# 1. Initialize migration repository (Run once)
flask db init

# 2. Generate migration script from changes in app.py models
flask db migrate -m "Add custom profile and API key tables"

# 3. Apply changes to database.db / production PostgreSQL database
flask db upgrade

# Rollback changes if needed
flask db downgrade
```

### Method B: Standalone DDL Script (Fast SQLite Patch)
For quick local testing on SQLite where Flask-Migrate is not initialized, run:

```bash
python add_colums.py
```
*This executes manual DDL `ALTER TABLE` statements directly to add user profile configuration fields without losing existing data.*

---

## 🤖 Preparing the ML Model

To ensure the serialized Random Forest model works under your local version of `scikit-learn` (and to avoid version warning crashes), execute the model trainer:

```bash
python retrain_model.py
```
This builds and saves a fresh binary `risk_model.pkl` file in your root workspace using your environment's specific packages.

---

## 🚀 Running the Server Locally

### Development Server (Hot Reload Enabled)
Start the Flask built-in server:

```bash
python app.py
```
*   **Access**: Open your browser and go to [http://localhost:5000](http://localhost:5000).
*   **Port**: Defaults to `5000` (can be configured via `$PORT` environment variable).

### Production WSGI (Using Gunicorn)
To run the server locally under production conditions (similar to Render hosting), use Gunicorn:

```bash
gunicorn -w 4 -b 127.0.0.1:5000 app:app
```

---

## ☁️ Cloud Deployment Guidelines (Render & Vercel)

RiskPulse can be hosted on traditional cloud platforms (Render, Heroku) or serverless platforms (Vercel).

### Option 1: Render / Traditional VPS
1.  **Repository**: Push code to GitHub/GitLab.
2.  **Build Command**: `pip install -r requirements.txt && python retrain_model.py` (Recompiles the model binary).
3.  **Start Command**: `gunicorn app:app` (Referenced in [Procfile](file:///c:/Users/GURKIRAT%20SINGH/OneDrive/Desktop/2nd/simple%20projects/code_relay/project_risk_system/procfile)).
4.  **Environment Variables**: `SECRET_KEY`, `FLASK_ENV=production`, `DATABASE_URL` (PostgreSQL).

### Option 2: Vercel (Serverless Functions)
Vercel runs Flask as serverless functions. Because Vercel utilizes a read-only filesystem (`/var/task`), you must respect these serverless rules:
1.  **Logging**: RiskPulse automatically detects Vercel (`VERCEL=1`) and redirects logging from file-writing (`RotatingFileHandler`) to `StreamHandler` (std.out). This prevents the `OSError: [Errno 30] Read-only file system` crash.
2.  **Database**: You **MUST** use an external hosted database (e.g., Supabase or Neon PostgreSQL) via the `DATABASE_URL` environment variable. A local SQLite database is read-only and ephemeral, which will fail or lose data.
3.  **Model Compilation**: Vercel runs in a clean build workspace. Ensure `python retrain_model.py` is called if you change python environment settings.

---

## 🔍 Troubleshooting Guide

### 1. OSError: [Errno 30] Read-only file system (Vercel Logging Crash)
*   **Issue**: Server crashes at startup when creating `/logs/app.log` on serverless host platforms.
*   **Fix**: The logging setup in `app.py` has been updated to check for `os.environ.get("VERCEL")`. It falls back to `logging.StreamHandler()` if run in serverless mode, printing logs to the terminal console.

### 2. InconsistentVersionWarning
*   **Issue**: Model was trained on a different version of scikit-learn, causing an app crash during pickling.
*   **Fix**: Run `python retrain_model.py` locally or as part of the build step.

### 3. Database Locked Error (SQLite)
*   **Issue**: Too many concurrent connections or an uncommitted transaction has locked `database.db`.
*   **Fix**: Stop the local server, delete temporary `.db-journal` files, or commit open changes.

### 4. Rate Limit Exceeded (HTTP 429)
*   **Issue**: Flask-Limiter has blocked your local IP address due to too many rapid clicks during testing.
*   **Fix**: For testing environments, disable rate limits by setting `limiter = Limiter(app=app, enabled=False)` in `app.py`, or clear the browser cache and wait.

### 5. Avatar Upload Fails on Vercel
*   **Issue**: Uploading avatars fails because Vercel does not allow writing to `/static/avatars/`.
*   **Fix**: In serverless settings, you must configure a remote file storage bucket (e.g., AWS S3 or Cloudinary) and hook it up to the `/api/settings/avatar` endpoint instead of local disk writes. Local avatar updates will gracefully fail without crashing the server.
