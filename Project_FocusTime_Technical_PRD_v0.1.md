# Project FocusTime — Technical PRD

Version: v0.2 definition draft, 2026-10-20 (historical filename retained).
Status: MVP definition recorded; implementation and capture validation not yet performed.

## Authority and changes

[FT-001 scope/deployment decision](docs/mvp-scope-and-deployment.md) governs this document and the [product PRD](Project_FocusTime_PRD_v0.1.md). It supersedes the original Go → Convex → MotherDuck pipeline, Service wording, password-derived encryption and title-based private filtering. Earlier examples remain in Git history; their sample schemas, API paths, environment variables and CI snippets are not accepted implementation contracts.

## Selected deployment and supporting stack

One standard-user Windows Go tray agent → authenticated TLS ingestion → managed Convex activity/rule/session storage and computation → React web dashboard. WorkOS is recommended for account identity; device pairing/token authorization is separate and revocable. The user selected managed Convex/server-readable processing on 2026-10-20; the supporting agent/dashboard stack and WorkOS remain architecture defaults rather than separately confirmed user preferences. FT-001 records Windows 11 Professional x64 Build 26300.9711 (26H2) and both real project contexts. Brave 1.97.59 (Official Build, 64-bit), Chromium 155.0.8059.85 is confirmed. FT-001 records the Alteryx T3 project/worktree clue and TIL Convo feed tab (`https://app.convo.com/#/feed`). A generic meeting is a proposed ambiguity test, not a user-confirmed example. Verify this exact Windows build/channel and Brave version before claiming capture support.

Convex is the sole canonical reporting authority. Recompute daily/weekly project totals after ingestion and corrections; no MotherDuck sync, second event store or nightly dependency. Deploy the backend through its managed platform and the dashboard as a web build on a host selected in the owning setup issue. No Docker backend publishing, MotherDuck credentials or Stripe stubs are prerequisites. Development and pilot/production data/credentials must be separate; infrastructure is not provisioned by FT-001.

## Agent and browser boundary

Run capture in the interactive login session, not a Session 0 Windows Service. Five-second foreground polling and a five-minute idle threshold are starting settings; interval boundaries, retrospective idle handling, timezone/day boundaries and clock changes require an explicit FT-003 contract. Never assume each sample proves five worked seconds. Stop on lock/sleep and expose gaps.

Use an encrypted SQLite offline queue with stable event IDs and acknowledged replay. Protect a random local data key with Windows user-scoped facilities; cover sensitive WAL/journal/temp data and protect credentials separately. Do not assume OIDC supplies a password for PBKDF2. Cipher/library details, recovery, rotation and queue bounds belong to the owning implementation issues and must be validated before real capture.

The pilot targets Brave, not an assumed Chrome/Edge installation. Brave capture requires an actual version-specific proof, potentially an extension/native messaging bridge with documented least-required permissions. Correlate the foreground window and active tab. Never collect background tabs or count the browser twice. Suppress titles/URLs when private status is unknown, including companion disconnects; extension incognito settings alone do not protect the desktop collector. Filter locally before storage/upload, default to domain-only and opt into sanitized paths. Never use “Incognito”/“Private” title substrings as the privacy boundary.

Convo uses a hash route. A locally configured exact host/route match may emit a minimal `convo-feed` signal, then discard the fragment before persistence/upload. Do not retain arbitrary fragments or auto-assign every Convo page from its domain; fall back to a reviewable suggestion if route matching cannot be done safely.

The supplied T3 worktree path is a candidate local project rule signal, not proof that foreground capture exposes it. Prefer minimal project identifiers; do not upload full user-profile paths unnecessarily. Missing T3 context or generic meeting activity must stay unassigned/reviewable rather than inherit a project from the app alone.

FT-001 records proposed version support and required user facts. Exact tested OS/browser/extension versions and actual permission grants are required evidence, not inferred support.

## Trust and data handling

The selected backend can decrypt/read permitted activity and perform rules/reporting. TLS and encryption at rest do not provide end-to-end encryption. Do not describe the service as unable to read activity. Verify provider controls and hosting region before live deployment. A future change to client-only encryption would require revising the selected model: move attribution/aggregation to trusted clients and redesign reporting/key recovery; if local/self-hosted operation is chosen, revisit packaging and operations.

Derive ownership from authenticated account/device identity; never trust a payload owner ID. Scope all data and operations to that owner, validate inputs and revoke devices. Do not ship backend administrative or identity-provider secrets in the agent/dashboard. Validate TLS certificates; negotiate supported secure TLS rather than claiming an unverified TLS-1.3-only endpoint.

Exclude clipboard, typed input, screenshots and window contents entirely. Local privacy controls precede persistence. Diagnostics contain status/counts/error codes, not titles, paths, URLs or tokens. Activity must not enter session replay or third-party AI/telemetry by default.

Proposed retention is raw activity 30 days, acknowledged local queue ≤7 days, reviewed sessions until user deletion, and no silent loss of unacknowledged data. Provide export/deletion and retention controls before replacement; prevent queued replay from resurrecting deleted data. Document derived-data deletion and provider backup expiry. These are design requirements, not implemented guarantees.

## Domain and computation responsibilities

FT-003 owns concrete schemas and API contracts. Required conceptual separation:

- Activity intervals: immutable captured evidence with device/event identity, start/end, permitted context, source and state.
- Projects (optional clients): stable owned identities, independently of app/category/productivity labels.
- Rules: explicit context predicates, specificity/conflict behavior, enablement and version.
- Assignments: automatic/manual/suggested/unassigned state, project and provenance. Conflicts need review; suggestions do not silently become reviewed entries.
- Sessions/time entries: editable project intervals referencing evidence, review state and override history; manual non-computer time must not duplicate captured time.

Ingestion validates ownership and intervals, acknowledges durable IDs, and tolerates repeated/offline delivery. Corrections and rule reruns update current totals without replacing manual overrides. Rule creation from correction is explicit and narrow; historical reclassification is bounded and reviewable. Reporting must expose unassigned, excluded, idle and unavailable capture separately.

## Validation and release boundary

First prove capture → deterministic project assignment/suggestion → correction → current daily/weekly totals using the two actual project contexts. Include ambiguity, private mode, idle/lock, offline replay and override survival. Then validate export/history integrity, deletion, crash recovery, device revocation and installation before the workweek replacement pilot.

Use real interactive Windows validation for capture; Linux development and fixtures cannot prove it. Tests in implementation issues should cover time accounting, conflicts/overrides, ownership, privacy suppression and duplicate ingestion, plus the complete workflow. Benchmark baseline targets (<2% CPU, <100 MB agent RAM, <2s week view) on recorded hardware. Metric formulas and replacement gates are in FT-001; no benchmark or pilot is completed here.

GitHub Actions build/test/artifact details, installer choice, exact endpoints, hosting configuration and observability integrations remain with sibling issues. MotherDuck, subscriptions, extensive telemetry and optional self-hosting are deferred. Required security/privacy and minimal diagnostics are part of the first real-data slice, not late “hardening.”
