# Project FocusTime — Product PRD

Version: v0.2 definition draft, 2026-10-20 (historical filename retained).
Status: MVP definition recorded; implementation and capture validation not yet performed.

## Governing scope

[FT-001 scope and deployment decision](docs/mvp-scope-and-deployment.md) governs this PRD and the [technical PRD](Project_FocusTime_Technical_PRD_v0.1.md). It replaces the October 2025 category/report-first scope and conflicting E2EE promise. Earlier content remains in Git history. [Roadmap #1](https://github.com/Sivivatu/focus_time/issues/1) owns the implementation backlog; this document describes desired behavior, not implemented features.

## Problem and priority

The primary user's repeated project identification and manual assignment/update work in Solidtime is the first problem to solve. FocusTime should capture permitted activity, automatically attribute strong matches to projects, make ambiguity quick to review, and produce trustworthy daily/weekly project timesheets. Relevant RescueTime Timesheets suggestion/review behavior is a reference; broad RescueTime or Solidtime parity is not an MVP gate.

## Target and deployment

The user reports Windows 11 Professional x64, Build 26300.9711 (26H2). The confirmed pilot browser is Brave 1.97.59 (Official Build, 64-bit), Chromium 155.0.8059.85. Pilot projects are Alteryx Migrator Development (T3 Code, Alteryx App, identified by the T3 thread/project and supplied worktree) and TIL Administration/Presales (the Brave tab `https://app.convo.com/#/feed`, primarily Google Meet and also Microsoft Teams). FT-001 records the worktree and Convo feed clues. A generic meeting is a proposed ambiguity test; no real ambiguous example was supplied. Release-channel/lifecycle status and capture compatibility remain unverified. Brave is the pilot browser target; validate its actual version rather than assuming Chrome/Edge compatibility.

Selected deployment/privacy model (user response 2026-10-20): managed Convex with server-readable processing. Supporting stack defaults are React dashboard and Go per-user tray agent with an encrypted offline SQLite queue. WorkOS is the recommended hosted identity provider. The backend reads permitted metadata to run rules and reporting. Local encryption, TLS and storage encryption **are not E2EE**. Local/self-hosted operation and client-only encryption are outside the selected pilot model.

## First useful workflow

1. Capture foreground app and permitted context in the interactive user session, with local exclusions, pause and idle/lock/sleep accounting.
2. Match explicit, user-owned rules against reliable project clues. Explain each automatic assignment; leave conflicts and unknown context visible for review.
3. Review a daily timeline: accept suggestions, assign/correct, split/merge or exclude sessions. Optionally turn a correction into a narrow future rule. Manual overrides survive reruns.
4. Show current daily/weekly totals by project, with review status, unassigned time and capture gaps. Replaying uploads cannot add duplicate time.

Project identity, activity category and productivity rating are separate concepts. Generic shared app/domain matches must not silently assign every activity to one project. Suggestions are not final reviewed time entries or billable commitments.

## Windows MVP boundaries

The first slice above precedes the broader M1–M6 release: secure onboarding/device authorization, offline/restart recovery, project/rule management, manual non-computer time without double-counting, portable export, necessary history migration, privacy/deletion controls and Windows packaging. App/category summaries and optional weekly email come after useful project review. No report depends on a nightly warehouse refresh.

Capture only metadata; never screenshots, window contents, typed input or clipboard. Private browsing must be filtered before persistence using a verified mechanism. Unknown browser/private state suppresses sensitive browser context; title substring filtering is insufficient. Full URLs are not a default: use domains and explicitly permitted sanitized paths, removing credentials/query/fragment locally. For the Convo hash route, use only a locally configured exact match that emits a minimal context signal and discards the fragment; if unavailable, suggest for review rather than auto-assigning from the domain. Exact permissions and support boundaries are specified in FT-001 and must be proved before real capture.

Provide user-controlled retention, export and deletion. Proposed raw retention is 30 days; no indefinite raw retention or paid-only access to the user's export. Diagnostic logs exclude activity context and credentials. No additional AI or telemetry service receives activity by default.

## Success and replacement gate

Run a representative five-workday comparison against an independently reviewed interval/project reference. Initial targets: ≥80% automatic coverage of eligible active minutes, ≥95% duration-weighted correctness before correction, <5 minutes daily review including rule maintenance, and ≤5% final total-time discrepancy. FT-001 defines denominators, undefined cases and reporting of gaps, wrong assignments and per-project errors. These targets await user calibration; none has been achieved.

Also verify privacy, offline/restart recovery, duplicate safety, export/deletion and performance. Benchmark targets remain tracker CPU <2%, RAM <100 MB and week-view load <2s on recorded hardware. Keep Solidtime operating until needed history/export is verified and the user explicitly accepts replacement. No uninstall or data deletion is authorized.

## Deferred scope

MotherDuck/DuckDB warehouse, goals/alerts/blocking, rich trends, calendar/LLM suggestions, Android/macOS/Linux, additional browsers, teams, billing, Stripe, subscriptions, product telemetry and self-hosted packaging are later work. A companion extension needed for safe Windows URL capture is an MVP capture dependency, distinct from a future browser-only tracker.

## Acceptance state

The three definition criteria for issue #2 are documented and ready for review/integration. Windows/browser versions, both project contexts/clues and the deployment/privacy choice are recorded. The workflow is defined; numerical pilot targets remain starting targets to calibrate before the run. Implementation belongs to sibling issues; there is no working application in this definition change.
