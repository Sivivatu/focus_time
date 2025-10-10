# 🧭 Project FocusTime  
**Lean MVP Product Requirements Document**  
**Owner:** Paul Houghton  
**Version:** v0.1 Draft  
**Status:** In Definition Phase  
**Date:** October 2025  

## 1. Executive Summary

**Project FocusTime** is an internal Codex initiative to build a privacy-respecting, data-driven personal productivity tracker inspired by RescueTime.  
It will enable users to understand how they spend their computer time, automatically generate activity summaries, and lay the foundation for future team and AI-assisted reporting.

The MVP will consist of:  
1. A **Windows background activity tracker** written in Go that securely records application and browser usage.  
2. A **web dashboard** built with React and a Convex backend that visualises time by category, project, and client.  
3. A **DuckDB analytics layer (via MotherDuck)** to enable daily aggregations and summary reports.  

The goal is to deliver a lightweight, reliable system for individual users and freelancers, designed for eventual expansion to team and cross-platform use.

## 2. Objectives & Success Criteria

### Objectives
- Provide full access and transparency to users’ activity data.  
- Automatically generate meaningful time-usage summaries and weekly reports.  
- Establish an extensible architecture for multi-platform tracking (Windows → Android → Chrome → Apple).  
- Ensure offline reliability and robust encryption to guarantee user privacy.

### Success Criteria
- ✅ Windows tracker records active window titles and browser URLs accurately within a 5-second polling interval.  
- ✅ Data is encrypted locally and uploaded automatically when online.  
- ✅ The React dashboard renders aggregated daily and weekly summaries without manual refresh.  
- ✅ Weekly summary email/report generated server-side.  
- ✅ End-to-end pipeline (Go → Convex → DuckDB → React) operates with negligible system overhead (<2 % CPU, <100 MB RAM).  

## 3. Target Users & Use Cases

**Primary user:** individuals and freelancers who want clear insight into their digital work patterns.  

**Initial use cases:**
- Automatically logging work activity for timesheet creation.  
- Reviewing daily and weekly activity reports by project/client.  
- Identifying patterns of productive vs. distracting work.  
- Accessing raw activity data for personal analytics.

Future use cases (post-MVP):  
- Multi-device synchronisation (Windows + mobile).  
- Team dashboards with aggregated visibility.  
- AI-driven categorisation and semantic tagging.

## 4. System Overview & Architecture

FocusTime comprises three cooperating components:

1. **Windows Tracker (Go Service)**  
   - Runs as a background process.  
   - Captures active window title and browser URL every 5 seconds.  
   - Detects idle time after 5 minutes of no keyboard/mouse input.  
   - Stores encrypted data locally (SQLite or BoltDB).  
   - Streams data to Convex when connectivity is restored.

2. **Convex Backend**  
   - Receives and validates device uploads via a device-token model.  
   - Handles data encryption, storage, and user authentication (via WorkOS).  
   - Invokes scheduled aggregation jobs to update DuckDB daily tables.  
   - Supports future Stripe integration for paid tiers.

3. **DuckDB (via MotherDuck)**  
   - Stores daily and weekly aggregated views for reporting.  
   - Enables fast analytical queries from the React dashboard.  

4. **React Web Dashboard**  
   - Displays time by category, top apps, and activity timelines.  
   - Allows manual re-categorisation and optional back-dating.  
   - Provides configurable working hours and holiday tracking.  
   - Offers downloadable summary reports (JSON export for paid tiers).  

![System Architecture Diagram Placeholder]

## 5. Functional Requirements

### 5.1 Windows Tracker
- Capture:  
  - Active window titles and process names.  
  - Full URLs for browser activity (active tab only).  
  - Idle detection via keyboard/mouse inactivity > 5 min.  
  - Time windows defined by user (e.g. 09:00–17:00).  
- Privacy:  
  - Never capture window contents.  
  - Redact incognito/private windows locally.  
  - Locally encrypt all logs before upload.  
- Storage & Upload:  
  - Store events locally when offline; batch upload on reconnect.  
  - Support configurable verbosity logging in JSON format.  
  - Include lightweight context UI for settings (polling rate, verbosity, working hours).

### 5.2 Backend (Convex)
- Handle user/device authentication via WorkOS.  
- Receive event payloads via secure HTTPS endpoints.  
- Encrypt at rest using user-derived keys and platform-level encryption.  
- Trigger daily aggregation jobs into DuckDB.  
- Expose a simple API endpoint for dashboard queries.  

### 5.3 Analytics Layer (DuckDB / MotherDuck)
- Maintain event-level data for detailed history.  
- Create daily aggregates for reporting:  
  - Total active time by app, category, client/project.  
  - Productive vs. neutral vs. distracting classification.  
  - Work vs. personal segmentation.  
- Retain raw data indefinitely (initially single-user storage).  

### 5.4 Dashboard (React)
- **Views:** Today, Week, Trends, Categories, Apps/Websites, Goals, Reports.  
- **Charts:** time by category (stacked bar), top apps, daily timeline.  
- **User Settings:** working hours, day-of-week profile, dark mode default.  
- **Accessibility:** WCAG 2.1 compliant, fully keyboard-navigable.  
- **Reports:** daily/weekly/monthly summaries, downloadable or emailed.  

![Dashboard Overview Placeholder]  

## 6. Non-Functional Requirements

### 6.1 Security & Privacy
- End-to-end encryption: locally derived keys per user.  
- TLS enforced for all network communication.  
- No window content capture, only metadata.  
- GDPR-aligned design (data portability, erasure readiness).  

### 6.2 Performance
- Tracker CPU < 2 %; RAM < 100 MB.  
- Dashboard load < 2 s for a one-week view.  
- Aggregation jobs complete within 30 s for a single user.  

### 6.3 Reliability
- Offline-first design for the tracker.  
- Local data persistence with retry queue.  
- Unit and integration testing pipeline with coverage > 80 %.  

### 6.4 Maintainability
- Modular codebases:  
  - `focus-agent` (Go)  
  - `focus-backend` (Convex + DuckDB)  
  - `focus-dashboard` (React)  
- Configuration via environment variables.  
- Clear separation of development and production environments.

## 7. CI/CD & Observability

- **Version control:** GitHub (monorepo).  
- **CI/CD:** GitHub Actions for build, lint, test, and deploy.  
  - Automated builds for Go service binaries.  
  - Dockerised deployment for backend and analytics stack.  
- **Feature flags:** PostHog.  
- **Error tracking:** Sentry.io integration for both backend and dashboard.  
- **Logging:** Structured JSON logs with severity levels.  
- **Monitoring:** Convex metrics dashboard + optional MotherDuck queries.

![CI/CD Pipeline Diagram Placeholder]

## 8. MVP Acceptance Criteria

- Windows tracker runs continuously, recording valid event data.  
- Event data successfully uploads to Convex and aggregates nightly.  
- React dashboard displays daily and weekly usage summaries.  
- Weekly report generation functional (view and email).  
- All communications encrypted end-to-end.  
- Basic onboarding (WorkOS sign-in) functional.  
- System stable for 1-week continuous run with minimal performance impact.  

## 9. Future Roadmap (Not in MVP Scope)

| Phase | Feature | Description |
|-------|----------|-------------|
| **Phase 2** | Android Tracker | Background tracking of app usage, integrated via shared Convex API. |
| **Phase 3** | Chrome Extension | Browser-only tracking for cross-device continuity. |
| **Phase 4** | Apple & Linux Support | Native agents for macOS & Linux. |
| **Phase 5** | Team Dashboards | Aggregated reporting by team/project with privacy-preserving summarisation. |
| **Phase 6** | AI/LLM Tagging | Semantic clustering of activity using OpenAI/Gemini via OpenRouter. |
| **Phase 7** | Paid Plans | Stripe-enabled subscriptions unlocking multiple devices, raw data export, and team analytics. |
| **Phase 8** | Self-Host Option | Docker-based deployment for enterprise or privacy-first users. |

![Product Roadmap Placeholder]

## 10. Next Steps

1. **Confirm technical stack setup** (Convex project, DuckDB schema, Go build pipeline).  
2. **Define API contracts** for event upload and dashboard queries.  
3. **Create design mock-ups** for dashboard layout (RescueTime-style summary aesthetic).  
4. **Implement Windows tracker prototype** with secure local storage.  
5. **Stand-up CI/CD pipeline** with PostHog and Sentry integrations.  
6. **Deliver MVP milestone demo** (tracker + dashboard + weekly report).  

**End of Document**  
_This PRD defines the initial implementation scope for Project FocusTime MVP. Subsequent updates (v0.2+) will expand on data schemas, API specifications, and UI mock-ups._
