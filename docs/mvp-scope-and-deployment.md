# FT-001: MVP scope, environment and deployment decision

Date: 2026-10-20. Status: definition recorded; managed/server-readable model selected and actual pilot environment/context supplied. Capture compatibility remains unverified.

Tracks [issue #2](https://github.com/Sivivatu/focus_time/issues/2), [roadmap #1](https://github.com/Sivivatu/focus_time/issues/1), and [Project](https://github.com/users/Sivivatu/projects/5). This document governs the revised product and technical PRDs. It defines outcomes and boundaries; it does not implement the app or sibling issues.

## Decision status

Confirmed user need: reduce repeated identification and manual project assignment in Solidtime, replicate relevant RescueTime behavior, and eventually retire the local Solidtime workflow. On 2026-10-20 the user supplied the Windows build and two project contexts below, and selected option 1: managed Convex hosting with server-readable processing. The user subsequently confirmed Brave Browser, primarily Google Meet with Microsoft Teams also used, and a T3 project/worktree clue. The user then supplied the exact Brave/Chromium versions and identified the Convo browser tab below. Required environment and representative-context facts are now recorded.

Product decisions for this proposal:

- Prioritize automatic project attribution, reusable corrections, and trustworthy project totals over broad competitor parity.
- Use deterministic, explainable rules first. Ambiguity remains visible. User overrides survive reprocessing.
- Start with one user and one interactive Windows device; Windows 11 Professional x64 is now user-confirmed.
- Keep one canonical activity/reporting store. Defer MotherDuck, AI classification, teams, billing, other platforms and broad telemetry.

Architecture: the user selected **managed Convex with server-readable processing**. Supporting stack defaults are Go per-user Windows tray agent, encrypted SQLite offline queue and React web dashboard; WorkOS is the recommended identity provider, with separately revocable device credentials. The trust model is selected; validate local privacy controls before real capture.

## Recorded pilot environment and support boundary

User-reported facts received 2026-10-20 are recorded verbatim below. Unreported details must not be inferred from the development host or hypothetical examples.

- Actual Windows: **Microsoft Windows 11 Professional (x64), Build 26300.9711 (26H2)**, as reported by the user. Release channel and lifecycle/support status have not been independently verified; do not infer stable-release support from this version string.
- Actual browser: **Brave 1.97.59 (Official Build) (64-bit)**, **Chromium 155.0.8059.85**, reported as up to date by the user from the browser About page. No separate release channel was supplied. Convo is a specific tab in this browser, not a separate browser product.
- Project context A: **Alteryx Migrator Development** — **T3 Code, Alteryx App**. User identifies the **T3 thread/project** as the distinguishing clue and supplies worktree `C:\Users\paulj\.t3\worktrees\alteryx_migrator\cli-push-migration`. Treat the project/worktree identity as a candidate rule signal only if a permitted collector exposes it; the supplied path is not evidence that a foreground-window API provides it. Prefer a local match that emits a minimal project/worktree identifier over uploading the full user-profile path. Sensitive exclusions remain to be configured.
- Project context B: **TIL Administration/Presales** — the specific Brave tab **`https://app.convo.com/#/feed`**, plus **primarily Google Meet and also Microsoft Teams**, clarified by the user. The supplied tab is the initial TIL context clue; it does not establish that all Convo activity or all meetings belong to TIL. A real shared/ambiguous activity has not been supplied; use the proposed generic-meeting case below for initial validation. Sensitive exclusions remain to be configured during onboarding, rather than inferred from these examples.
- Deployment/privacy: **confirmed managed Convex hosting with server-readable activity for rules/reporting, protected in transit and at rest; not end-to-end encryption** (user selected option 1).

Pilot target: the exact user-reported Windows build above, subject to capture validation and release-channel/lifecycle verification. General support remains proposed for Windows 11 x64 on a Microsoft-supported release. The pilot browser is Brave, replacing the earlier Chrome/Edge recommendation. The initial target is Brave 1.97.59 / Chromium 155.0.8059.85 (64-bit); prove active-tab correlation, native messaging/extension availability, and private-window suppression on that build. Chromium ancestry is not proof of compatibility. Chrome/Edge are not additional pilot requirements. Convo runs in a Brave tab and uses the same capture/privacy boundary. Record exact tested OS/build and browser/extension versions with the capture proof and each pilot run. Validate browser updates before claiming support. Windows 10, ARM, browsers other than the pinned Brave pilot target, remote/multi-user desktops, other OSes and mobile are outside this initial support claim. If the actual device falls outside the proposal, revise the target before acceptance rather than asserting compatibility. No version is verified today.

Capture permissions and limitations:

- Run as a standard-user process in the interactive login session, with opt-in login startup and visible pause/status controls. No administrator rights or Windows Service is required by the proposal. Elevated apps and protected desktops may be unreadable; expose gaps without elevating silently.
- Capture foreground process and allowed window metadata only. Use input-idle timing, never keyboard contents, clipboard, screenshots or window contents. Stop accounting at lock/sleep and exclude idle time; precise interval/idle semantics belong in FT-003.
- Browser URLs require a proven active-tab mechanism, potentially a companion extension plus native messaging host. Request only the permissions justified by that mechanism (potentially `tabs` and `nativeMessaging`); document exact manifest/host access and enterprise-policy constraints in FT-015. Continuous background collection cannot rely on a user gesture such as `activeTab` alone.
- Correlate the active browser window/tab with the foreground desktop window. Do not count background tabs or duplicate desktop and extension time. A repository path is usable only when actually exposed by an authorized capture source; do not assume the agent can read editor workspace paths.
- Exclude private browsing before persistence. Extension private-mode settings alone do not protect desktop title capture; title substring checks are insufficient. If private status or browser identity cannot be established, suppress browser titles/URLs and mark context unavailable. Never fall back to sensitive title collection when the extension disconnects.
- Default browser context to domain-only; enable sanitized path matching explicitly where needed to distinguish projects. Remove URL credentials, query strings and fragments locally before persistence/upload. The supplied Convo URL uses a hash route: an explicitly configured local exact match on host `app.convo.com` and fragment route `/feed` may emit a minimal `convo-feed` context signal, then discard the fragment. Never store arbitrary fragments or broaden this to every Convo page. If safe route matching is unavailable, show a domain-only suggestion for review rather than an automatic TIL assignment. Provide local app/site exclusions and pause before capture. Minimize titles too: they can contain secrets even without window contents.

Windows/session and browser behavior must be demonstrated on the actual interactive device. A Linux devcontainer or mocked payload cannot pass this requirement.

## Deployment and encryption choice

Confirmed user choice (2026-10-20): **managed Convex backend with server-readable processing**. It fits the existing Convex/React direction and allows one place to run rules, preserve corrections, and update reports without depending on an always-open dashboard. Hosted operation avoids asking a single pilot user to operate a server. Removing the local Solidtime workflow does not imply a local-only replacement.

Under this selected trust model, permitted activity metadata leaves the device over validated TLS and is readable by the authorized backend to perform attribution and reporting. Hosted service operators with authorized access may access plaintext; this is not end-to-end encryption. Local encryption and provider storage encryption protect different boundaries and do not change that fact. Verify provider encryption, access controls, hosting region and recovery/retention behavior before live deployment; no region or provider guarantee is asserted as already configured.

Local queue: encrypted SQLite, including sensitive journal/WAL/temp paths, using a random data key protected by Windows user-scoped credential facilities. Do not derive a key from an OIDC password: the agent does not receive one. Cipher/library selection, nonce handling, rotation, recovery and device revocation are implementation decisions in their owning issues. Authentication tokens are protected separately from queue data. Logs contain health/counts/error codes, never activity titles, paths, URLs or tokens.

Recommended pilot retention: raw activity 30 days, acknowledged local queue removed within 7 days, and unacknowledged data never silently evicted. Surface queue capacity/backpressure and pause rather than lose data silently. Reviewed project sessions remain until user deletion; retention settings, portable export and deletion must be available before replacement. Deletion covers derived data and queued replay; provider backup expiry must be disclosed. These are proposed defaults for confirmation during privacy design, not claims of implemented guarantees.

Alternatives considered:

- Local/self-hosted operation keeps control of hosting with the user but adds installation, backup, availability and update responsibilities. Self-hosting may still use server-readable processing; it does not imply E2EE. A fully local app would also change the web/backend packaging model.
- Client-only encryption keeps decryption keys off the service. Rules and aggregation must run on a trusted client; server-side raw-activity reports cannot remain as specified. Summary disclosure, multi-device keys, recovery and browser access need redesign. Select this if server visibility is unacceptable; do not merely add an E2EE label to the recommended architecture.

The user selected the managed/server-readable option; local/self-hosted operation and client-only E2EE are not the pilot model. Any later change to that choice requires revising this decision and both PRDs. WorkOS remains the recommended identity provider for the hosted option, not a requirement for a local-only design. No provider provisioning or deployment is authorized/performed by this definition document.

## First useful vertical slice

1. Create the two real pilot projects and narrow explicit rules from verified context signals.
2. Capture permitted foreground activity with idle/lock/sleep boundaries and a durable offline queue; synchronize without duplicate time.
3. Assign only unambiguous rule matches, recording reason and rule version. Conflicting matches become review items; missing context remains unassigned. Suggestions are separate from committed assignments and productivity categories.
4. Show a daily review queue and timeline. Allow assignment/correction, split/merge, exclusion, and an explicit option to create a narrowly scoped future rule. A correction alone must not silently generalize a rule or rewrite unrelated history.
5. Show current daily/weekly project totals with drill-down, review status, and separate unassigned, excluded, idle and capture-gap time. Edits and replay recompute totals without double-counting or overriding manual decisions.

Slice demonstration: switch repeatedly between the two real projects without selecting a timer/project each time; insert a shared ambiguous task, private browsing, idle/lock and an offline/reconnect period. Verify strong matches, visible ambiguity, no private metadata persisted, no idle attribution, correction survival after rerun, and identical totals after replay. Use the Alteryx T3 project/worktree signal when available and the Convo feed signal once verified; when either is unavailable, retain unassigned/reviewable time instead of inferring the project from the application. A generic meeting is a proposed ambiguity test, not a user-confirmed real example.

The first slice is narrower than the full Windows MVP in roadmap milestones M1–M6. Release work still includes portable exports, necessary manual/non-computer time, migration/history verification, privacy/recovery checks, installation and a representative workweek. App/category reporting and optional weekly email follow the project workflow; they do not substitute for it. M7–M8 competitor expansion remains later. No scaffolding, contract implementation, capture spike, migration, or sibling issue is completed here.

## Replacement pilot and measurable gates

Use a representative five-workday pilot alongside the current workflow, with both projects and ordinary ambiguity. Keep an independently reviewed reference log of interval boundaries, project labels, idle/exclusions and non-computer work; audit against it rather than treating FocusTime output or manually entered Solidtime totals as unquestioned truth. Freeze metric definitions and configured rules at run start; record subsequent rule changes and show results by day/project as well as overall.

Starting targets, to calibrate with the user before the run:

- **Automatic coverage ≥80%:** correctly or incorrectly auto-assigned eligible active minutes / all reference project-eligible computer-active minutes. Suggested-only and manually corrected minutes are not automatic coverage; report unavailable capture and unassigned minutes separately. Fix eligibility from the reference, not from successful matches.
- **Automatic correctness ≥95%:** automatically assigned minutes whose original project agrees with the reference / all automatically assigned minutes. Measure before corrections and weight by duration, not event count. Review all auto-assigned intervals in this small pilot; no automatic assignments means undefined correctness and a failed gate.
- **Review effort <5 minutes each workday:** timed normal daily review/correction/rule-maintenance effort. Record initial setup separately and report the existing Solidtime effort baseline, so an apparent improvement cannot hide setup work.
- **Total-time discrepancy ≤5%:** absolute difference between final reviewed FocusTime and reference eligible project time / reference eligible project time over the week. Report per-day and per-project errors too, so offsetting assignments are visible. Compare non-computer entries separately; a zero reference makes the metric undefined and requires another representative run.

Require all targets together and report missing capture, wrong-project minutes, exclusions, manual additions and review changes. In addition, prove private-mode suppression, offline/restart recovery, duplicate replay safety, export/deletion boundaries and acceptable CPU/RAM/report responsiveness on documented hardware. Keep the baseline performance goals (<2% tracker CPU, <100 MB RAM, <2s week view) as benchmark targets, not measured results.

Before retiring Solidtime: retain a verified export/backup, reconcile project mappings/counts/durations and needed historical access, and obtain an explicit replacement decision after the pilot. No uninstall or history deletion is authorized. If a gate fails, continue alongside Solidtime and address the failure; broad competitor parity is not the release criterion.

## Acceptance record

- AC1 **documented**: actual Windows and Brave/Chromium versions, both real project contexts and their supplied clues, and capture permissions/support boundaries are recorded. Their availability to a collector and version compatibility require later implementation validation. A real ambiguity example was not supplied; the generic-meeting scenario is explicitly a proposed test, not a confirmed user fact. It is not an additional definition acceptance gate.
- AC2 **documented**: user selected managed Convex/server-readable processing; both PRDs reflect this choice and explicitly disclaim E2EE.
- AC3 **documented**: vertical slice, pilot metrics and project-automation priority are defined; pilot targets still require calibration before execution.

All three definition criteria are documented. Issue #2 can be completed after review and integration of these documents; do not confuse definition acceptance with passing capture, security or pilot validation. No issue closure or implementation completion is claimed by this working-tree draft.

## References

- [Microsoft: interactive services and session isolation](https://learn.microsoft.com/en-us/windows/win32/services/interactive-services)
- [Microsoft: session-scoped GetLastInputInfo](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getlastinputinfo)
- [Chrome: extension incognito modes](https://developer.chrome.com/docs/extensions/reference/manifest/incognito)
- [Chrome: tabs permission and API](https://developer.chrome.com/docs/extensions/reference/api/tabs)
- [Chrome: native messaging](https://developer.chrome.com/docs/extensions/develop/concepts/native-messaging)
- [RescueTime: reviewable Timesheets suggestions](https://help.rescuetime.com/article/427-getting-started-with-rescuetime-timesheets)
- [Solidtime desktop activity schema](https://github.com/solidtime-io/solidtime-desktop/blob/main/src/main/db/schema.ts) shows local activity records, not established automatic project conversion.
- [Solidtime exports](https://docs.solidtime.io/user-guide/export) document structured project/time-entry export; do not assume passive desktop history is included.

Supporting assessment and research were read from the parent checkout as provided in the issue brief; they are not required links for this branch. Public references describe design constraints and behavioral precedent, not hands-on validation of the pilot environment.
