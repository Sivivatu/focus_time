# ⚙️ Project FocusTime – Technical PRD  
**Version:** v0.1  
**Date:** October 2025  
**Owner:** Paul Houghton  
**Audience:** Codex Engineering Team  

## 1. System Architecture Overview  

FocusTime consists of four cooperating subsystems:

```
+------------------+         +------------------+         +------------------+         +------------------+
| Windows Tracker  |  --->   | Convex Backend   |  --->   | DuckDB Analytics |  --->   | React Dashboard  |
| (Go Service)     |         | (API + Storage)  |         | (via MotherDuck) |         | (Web UI)         |
+------------------+         +------------------+         +------------------+         +------------------+
```

### Data Flow
1. **Go Tracker** captures events (active window, URL, idle) every 5 seconds.  
2. Data is encrypted locally, batched, and uploaded to **Convex** via HTTPS.  
3. Convex authenticates via WorkOS, stores raw events, and triggers daily aggregations.  
4. Aggregated data is synced into **DuckDB/MotherDuck** for analysis.  
5. **React Dashboard** queries Convex for pre-aggregated summaries and renders reports.

## 2. Module Responsibilities  

### 🧠 `focus-agent` (Go Tracker)  
- Polls active window/process every 5 s.  
- Detects idle after 5 min no input.  
- Persists events locally (embedded DB).  
- Encrypts and uploads on connectivity.  
- Provides minimal UI for settings (config YAML + tray context).

### 🔗 `focus-backend` (Convex)  
- Auth: WorkOS OIDC flow.  
- Data ingest: HTTPS event API.  
- Storage: Convex DB for raw events + metadata.  
- Jobs: cron aggregation → DuckDB via MotherDuck API.  
- Future: Stripe subscription integration.

### 📊 `focus-analytics` (DuckDB / MotherDuck)  
- Maintains daily/weekly summary tables.  
- Provides read-optimised views for dashboard queries.  
- Acts as intermediate warehouse for ML/AI extensions.

### 🖥️ `focus-dashboard` (React)  
- Fetches aggregated stats via Convex SDK.  
- Implements visuals (Time by Category, Top Apps, Timeline).  
- Supports dark mode and accessibility baseline (WCAG 2.1).  
- Future integrations: PostHog analytics, Sentry error tracking.

## 3. Data Model Schemas  

### Event (Base Unit)
```json
{
  "id": "uuid",
  "user_id": "uuid",
  "device_id": "uuid",
  "timestamp": "2025-10-10T09:30:05Z",
  "app_name": "chrome.exe",
  "window_title": "Project FocusTime – Docs",
  "url": "https://github.com/codex/focustime",
  "category": "Productive",
  "project": "Codex",
  "client": "Internal",
  "is_idle": false,
  "duration": 5,
  "is_private": false,
  "created_at": "2025-10-10T09:30:10Z"
}
```

### Aggregated Day
```json
{
  "user_id": "uuid",
  "date": "2025-10-10",
  "total_active_minutes": 465,
  "by_category": {
    "Productive": 390,
    "Neutral": 45,
    "Distracting": 30
  },
  "top_apps": [
    {"app": "code.exe", "minutes": 180},
    {"app": "chrome.exe", "minutes": 120}
  ],
  "goals": {
    "productive_target": 420,
    "achieved": false
  }
}
```

## 4. API Endpoints (Summary)

| Method | Endpoint | Description |
|--------|-----------|-------------|
| `POST` | `/api/v1/events/batch` | Upload batched event payloads from tracker. |
| `GET` | `/api/v1/summary/daily` | Fetch daily aggregate for dashboard. |
| `GET` | `/api/v1/summary/weekly` | Fetch 7-day aggregate. |
| `POST` | `/api/v1/user/settings` | Update working hours or preferences. |

### Example Request: Event Batch
```http
POST /api/v1/events/batch
Authorization: Bearer <device_token>
Content-Type: application/json
```
```json
{
  "device_id": "DEVICE-1234",
  "events": [
    {"timestamp":"2025-10-10T09:00:00Z","app_name":"chrome.exe","window_title":"Docs","url":"https://...","duration":5}
  ]
}
```

### Example Response
```json
{"status":"ok","stored":1,"errors":[]}
```

## 5. Configuration & Environment  

### Environment Variables
| Key | Description |
|-----|--------------|
| `CONVEX_URL` | Backend endpoint for data upload. |
| `CONVEX_API_KEY` | Auth token for internal jobs. |
| `WORKOS_CLIENT_ID` / `WORKOS_SECRET` | OAuth credentials. |
| `MOTHERDUCK_TOKEN` | Access token for DuckDB cloud. |
| `SENTRY_DSN` | Error tracking endpoint. |
| `POSTHOG_API_KEY` | Analytics telemetry. |
| `AGENT_UPLOAD_INTERVAL` | Seconds between upload batches (default = 30). |
| `AGENT_LOG_LEVEL` | debug | info | warn | error. |

### Local Storage (Agent)
- Default path: `%APPDATA%/FocusTime/data/events.db`  
- Encrypted with user-derived key (PBKDF2 + AES-256-GCM).  
- Logs rotated daily (`focus-agent.log`).  

## 6. Security & Encryption Model  

- **Local encryption:** AES-256-GCM; key derived from user credentials using PBKDF2.  
- **Transport:** HTTPS (TLS 1.3 minimum).  
- **Backend storage:** Convex encrypted at rest + per-user key partition.  
- **Privacy filters:**  
  - Skip window titles containing “Incognito” or “Private”.  
  - Never capture clipboard or typed input.  

## 7. CI/CD Pipeline Overview  

**Primary system:** GitHub Actions  

```
on: [push, pull_request, release]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - checkout
      - setup-go@v5
      - run: go test ./...
      - run: go build ./cmd/focus-agent
      - setup-node@v4
      - run: npm ci && npm run test
      - build-backend
      - docker buildx bake deploy
```

- **Canary deployments** via Convex branch environments.  
- **Tagged Docker images** for backend & dashboard.  
- **Versioning:** Semantic ( v0.x for MVP ).  

## 8. Monitoring & Logging Standards  

| Layer | Tool | Purpose |
|-------|------|---------|
| Go Agent | Structured JSON logs | Local debug / upload errors |
| Convex Backend | Sentry.io | Exceptions + performance metrics |
| Dashboard | PostHog | Feature flags + usage telemetry |
| MotherDuck | Internal logs | Aggregation job status |

All logs must exclude PII (window titles, URLs sanitised before submission).  

## 9. Testing Strategy  

| Layer | Type | Description |
|-------|------|-------------|
| Agent | Unit Tests | Event capture, idle logic, encryption. |
| Backend | Integration Tests | API auth, payload validation, aggregation jobs. |
| Dashboard | Component Tests | Charts render / data fetch hooks. |
| E2E | Smoke Tests | Tracker → Upload → Summary flow. |

**Coverage Goal:** > 80 % backend / agent logic.  

## 10. Release Plan & Next Steps  

1. **v0.1 Prototype:**  
   - Implement tracker logging + manual upload command.  
   - Stub Convex endpoint for testing.  

2. **v0.2 Integration:**  
   - End-to-end event pipeline + dashboard display.  

3. **v0.3 Stabilisation:**  
   - Encryption hardening + error tracking live.  

4. **v0.4 MVP Release:**  
   - Weekly report generation + installer (Winget).  

**End of Document**  
_This Technical PRD provides conceptual build guidance for Project FocusTime MVP. Detailed schema and repo structure will be defined in v0.2 Technical Design Document._
