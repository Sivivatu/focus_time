# Project FocusTime — Task List (MVP)

Source: Project_FocusTime_PRD_v0.1.md and Project_FocusTime_Technical_PRD_v0.1.md

## Phase Plan
- [ ] v0.1 Prototype — basic tracker + manual upload
- [ ] v0.2 Integration — end-to-end pipeline + basic dashboard
- [ ] v0.3 Stabilization — encryption hardening + error tracking
- [ ] v0.4 MVP — weekly report + Windows installer

## Windows Tracker (Go)
- [ ] Capture active window title + browser URL at 5s interval
- [ ] Implement idle detection (no input ≥ 5 minutes)
- [ ] Choose local encrypted datastore (SQLite or BoltDB)
- [ ] Implement local encryption (PBKDF2 + AES-256-GCM)
- [ ] Implement batch uploads with retry/backoff; offline buffering
- [ ] Device registration + token-based auth
- [ ] Config/env handling (intervals, log level, endpoints)
- [ ] Structured JSON logging + daily rotation
- [ ] Auto-start on login; Windows packaging steps

## Backend (Convex)
- [ ] Initialize Convex project and environments
- [ ] Integrate auth via WorkOS (login/session)
- [ ] Device-token model + upload auth middleware
- [ ] POST `/api/v1/events/batch` with payload validation
- [ ] Persist raw events (schema for app_name/title/url/duration/ts)
- [ ] Scheduled aggregation jobs (daily/weekly)
- [ ] GET `/api/v1/summary/daily` and `/api/v1/summary/weekly`
- [ ] POST `/api/v1/user/settings` (working hours, preferences)
- [ ] Stripe stubs for future paid tiers (no-op for MVP)

## Analytics (DuckDB via MotherDuck)
- [ ] Create schemas for daily/weekly aggregates
- [ ] Connect Convex jobs to MotherDuck (token, permissions)
- [ ] Aggregation queries for top apps, categories, timelines
- [ ] Performance tuning for dashboard query patterns

## React Dashboard
- [ ] Scaffold app with auth (WorkOS) + base routing
- [ ] Data fetching hooks for summaries (daily/weekly)
- [ ] Daily/weekly summary views (charts, tables)
- [ ] Activity timeline and top apps/categories widgets
- [ ] Manual re-categorization + optional back-dating
- [ ] Loading/error states; auto-refresh behavior

## Security & Privacy
- [ ] URL/title sanitization and privacy filters (skip Incognito/Private)
- [ ] Enforce TLS 1.3; validate certs on agent uploads
- [ ] Per-user partitioning/encryption at rest on backend
- [ ] Exclude PII from logs; document data handling

## Configuration & Ops
- [ ] Define required env vars (Convex, WorkOS, MotherDuck, Sentry, PostHog)
- [ ] Standardize local paths: `%APPDATA%/FocusTime/data/events.db`
- [ ] Agent log rotation policy and retention

## CI/CD
- [ ] GitHub Actions: build/test Go agent
- [ ] GitHub Actions: build/test dashboard
- [ ] GitHub Actions: build/test backend
- [ ] Build agent binary; publish artifacts
- [ ] Canary deploy backend to Convex branch environment
- [ ] Tag Docker images for backend/dashboard

## Monitoring & Telemetry
- [ ] Integrate Sentry in backend (exceptions, performance)
- [ ] Integrate PostHog in dashboard (feature flags, usage)
- [ ] Agent structured logs for upload/queue status
- [ ] MotherDuck job status logging and basic alerts

## Testing
- [ ] Agent unit tests: capture, idle, batching, encryption
- [ ] Backend integration tests: auth, payload validation, aggregation
- [ ] Dashboard component tests: charts and data hooks
- [ ] E2E smoke: tracker → upload → summaries
- [ ] Coverage target ≥ 80% for agent/backend logic

## MVP Reporting
- [ ] Weekly summary generation job (server-side)
- [ ] Email/report delivery with minimal template

## Release & Distribution
- [ ] Windows installer (Winget or Inno Setup)
- [ ] Versioning policy (semver v0.x)
- [ ] Release notes and onboarding guide

## Docs
- [ ] Quickstart for agent + dashboard setup
- [ ] API reference: event batch and summary endpoints
- [ ] Architecture and security model overview

