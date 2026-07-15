# API Specification & Integration Guide — RiskPulse

This document provides detailed documentation for the RiskPulse REST API, including endpoint URLs, authentication requirements, query/body schemas, validation rules, HTTP status codes, and implementation examples.

---

## 🔑 Authentication

RiskPulse supports two authentication schemes:

1.  **Session Authentication (Cookies)**: Used by the front-end dashboard. Upon calling `/login`, a secure HTTP-Only cookie `session` is stored in the user's browser and must be supplied in subsequent requests.
2.  **API Token Authentication**: (Key-based headers) Under active implementation. Standard API clients supply keys in the header as:
    ```http
    Authorization: Bearer rp_live_xxxxxxxxxxxxxxxxxxxxxxxx
    ```

---

## 🗺️ Endpoint Directory

| Method | Path | Authentication | Description | Rate Limit |
| :--- | :--- | :--- | :--- | :--- |
| **GET** | `/health` | None | Verify server status and ML model health. | None |
| **POST** | `/register` | None | Create a new user account. | 5 per hour |
| **POST** | `/login` | None | Authenticate credentials and establish session. | 10 per minute |
| **GET** | `/logout` | Session | Terminate active user session. | None |
| **POST** | `/predict` | Session | Submit project data to run ML delay risk prediction. | 30 per minute |
| **GET** | `/history` | Session | Fetch last 100 predictions ran by active user. | 50 per hour |
| **GET** | `/api/profile` | Session | Get aggregate user portfolio analytics & activity timeline. | None |
| **GET** | `/api/settings/load` | Session | Retrieve settings and profile preference states. | None |
| **POST** | `/api/settings/profile` | Session | Update user info details (name, bio, role, email). | 20 per hour |
| **POST** | `/api/settings/password`| Session | Securely update password hash. | 5 per hour |
| **POST** | `/api/settings/appearance`| Session | Set preferences for dark mode, accents, and fonts. | None |
| **POST** | `/api/settings/avatar` | Session | Upload avatar image (multipart form data, max 2MB). | 10 per hour |
| **POST** | `/api/settings/notifications`| Session | Toggle notification settings (email, push alerts). | None |
| **POST** | `/api/settings/privacy` | Session | Toggle public visibility configuration of profile. | None |
| **POST** | `/api/settings/apikeys/generate`| Session | Generate a new API access key. | 5 per hour |
| **POST** | `/api/settings/apikeys/revoke`| Session | De-authorize and delete an existing API key. | None |

---

## 📡 Endpoint Details & Examples

### 1. Model Prediction
`POST /predict`
Submits raw project metrics, engineering features, logs project entry, and returns prediction.

#### Request Headers
```http
Content-Type: application/json
```

#### Request JSON Body
```json
{
  "name": "E-Commerce Re-platforming",
  "progress": 45.5,
  "deadline": 90,
  "budget": 65.0,
  "team": 8
}
```
*   `name` *(String, optional)*: Project label. Default is `"Untitled Project"`.
*   `progress` *(Float, required)*: Percentage completion. Range: `0.0` to `100.0`.
*   `deadline` *(Float, required)*: Number of days remaining to deadline. Must be positive.
*   `budget` *(Float, required)*: Percentage of allocated budget spent. Range: `0.0` to `100.0`.
*   `team` *(Float, required)*: Number of team members assigned. Must be positive.

#### Response JSON (Success 200 OK)
```json
{
  "risk": 72.45,
  "status": "Delayed",
  "confidence": 72.45,
  "top_factors": [
    {
      "feature": "completion_rate",
      "importance": 24.12
    },
    {
      "feature": "budget_burn_rate",
      "importance": 18.05
    },
    {
      "feature": "productivity_score",
      "importance": 12.87
    }
  ],
  "reasons": [
    "High budget usage",
    "High delay pressure"
  ]
}
```

#### Error Response (400 Bad Request — Validation Failure)
```json
{
  "error": "Progress must be between 0 and 100"
}
```

#### Error Response (503 Service Unavailable — Model Missing)
```json
{
  "error": "ML model not available. Please contact support."
}
```

#### Example cURL
```bash
curl -X POST http://localhost:5000/predict \
     -H "Content-Type: application/json" \
     -d '{"name": "App Migration", "progress": 30.0, "deadline": 45.0, "budget": 85.0, "team": 4}'
```

---

### 2. Project History
`GET /history`
Returns a JSON list of the last 100 predictions.

#### Response JSON (Success 200 OK)
```json
[
  {
    "id": 12,
    "name": "E-Commerce Re-platforming",
    "risk": 72.45,
    "status": "Delayed",
    "timestamp": "2026-07-15 18:30"
  },
  {
    "id": 11,
    "name": "Database Upgrade",
    "risk": 15.2,
    "status": "On Track",
    "timestamp": "2026-07-15 14:15"
  }
]
```

---

### 3. Profile Aggregates
`GET /api/profile`
Calculates stats and lists recent actions for the current user's profile dashboard.

#### Response JSON (Success 200 OK)
```json
{
  "username": "gurkirat_bhangoo",
  "email": "gurkirat@example.com",
  "joined": "Jul 2026",
  "stats": {
    "projects": 12,
    "avg_risk": 43.8
  },
  "risk": {
    "critical": 2,
    "high": 3,
    "medium": 4,
    "low": 3
  },
  "activities": [
    {
      "title": "Analysis Completed",
      "desc": "E-Commerce Re-platforming — Risk 72.45%",
      "time": "15 Jul"
    }
  ]
}
```

---

### 4. Load Preferences
`GET /api/settings/load`
Retrieves account metadata and dashboard UI custom preferences.

#### Response JSON (Success 200 OK)
```json
{
  "first_name": "Gurkirat",
  "last_name": "Singh",
  "email": "gurkirat@example.com",
  "role": "Lead PM",
  "organisation": "TechCorp",
  "bio": "Managing enterprise cloud migrations",
  "avatar": "/static/avatars/avatar_1_fa32c8901.png",
  "dark_mode": true,
  "accent_colour": "Aurora",
  "font_size": "Default (15px)",
  "notifications": {
    "email_notifications": true,
    "push_alerts": false
  },
  "public_profile": false
}
```

---

### 5. Update Profile Details
`POST /api/settings/profile`
Modifies profile fields (lengths restricted).

#### Request JSON Body
```json
{
  "first_name": "Gurkirat",
  "last_name": "Bhangoo",
  "email": "new_email@example.com",
  "role": "Director of Product",
  "organisation": "RiskPulse Corp",
  "bio": "Specializing in risk automation and machine learning deployments."
}
```

#### Response JSON (Success 200 OK)
```json
{
  "success": true
}
```

#### Error Response (400 Bad Request — Email Collision)
```json
{
  "success": false,
  "error": "Email already in use"
}
```

---

### 6. Upload Avatar Image
`POST /api/settings/avatar`
Uploads profile picture file. Must be multipart form-data.

#### Form Parameters
*   `avatar`: Binary file data. Allowed extensions: `jpg`, `jpeg`, `png`, `gif`, `webp`. Size must be $\le$ 2MB.

#### Response JSON (Success 200 OK)
```json
{
  "success": true,
  "url": "/static/avatars/avatar_1_5af98ce2ab048c12.png"
}
```

---

### 7. API Key Generator
`POST /api/settings/apikeys/generate`
Generates token, hashes it, and displays the raw token exactly once.

#### Request JSON Body
```json
{
  "name": "Production CI Script"
}
```

#### Response JSON (Success 200 OK)
```json
{
  "success": true,
  "key": "rp_live_ef9a823b1cd4c8e7629ab6e1a49c25091bfe",
  "key_id": "8bfa2e128cc19a2b",
  "warning": "Save this key now. You won't see it again."
}
```

---

### 8. System Status
`GET /health`
Validates critical component availability.

#### Response JSON (Success 200 OK)
```json
{
  "status": "healthy",
  "model_loaded": true,
  "database": "connected"
}
```

---

## 🚫 HTTP Standard Errors

| HTTP Code | Name | Cause | JSON Payload Format |
| :--- | :--- | :--- | :--- |
| **400** | Bad Request | Validation checks failed on parameters. | `{"error": "ErrorMessage"}` |
| **401** | Unauthorized | Session cookie expired, absent, or invalid token. | `{"error": "Unauthorized Access"}` |
| **413** | Payload Too Large | Profile image upload size exceeded 2MB limit. | `{"error": "File too large. Maximum size is 2MB"}` |
| **429** | Too Many Requests | User violated route rate limit thresholds. | `{"error": "Rate limit exceeded. Please try again later."}` |
| **500** | Internal Error | Database query runtime error or unhandled backend crash. | `{"error": "Internal server error"}` |
| **503** | Service Unavailable | The `risk_model.pkl` binary failed to load. | `{"error": "ML model not available"}` |
