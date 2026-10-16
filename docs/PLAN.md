# Nexus — Product and Engineering Plan

**Status:** Planning only (not implemented)  
**Updated:** 2026-10-16  
**Primary client:** iPhone 17 Pro Max, native SwiftUI  
**Desktop agent targets:** Windows first, macOS later  
**Guiding principle:** Offline-first personal assistance; private, permission-controlled remote orchestration

## 1. Product vision

Nexus is a personal AI assistant that helps its owner understand day-to-day work, learn technical skills, design and build personal projects, securely inspect trusted computers, trigger approved automation, and receive meaningful progress updates. It should also provide configurable hydration, standing, focus, and break reminders.

Nexus is **not** an unrestricted remote-access tool, a background surveillance tool, or an autonomous agent with unbounded authority. User control, transparent actions, and privacy are requirements.

### Core user journeys

1. **Ask about computer status:** From the phone, retrieve authenticated, fresh CPU/RAM/storage/uptime and online state for a paired personal PC.
2. **Ask about a project:** See tracked repository branch, working tree status, last commit, CI result, tests and milestone evidence, with timestamps and explicit unknowns.
3. **Trigger an approved job:** Select a registered task (e.g., run repository tests), confirm where needed, monitor progress, inspect sanitized logs, cancel safely.
4. **Receive meaningful updates:** Push on completion/failure/approval needed; provide a periodic opt-in digest, with quiet hours and deduplication.
5. **Personal assistance:** On-device chat, voice, searchable personal notes and opt-in reminders even without connectivity.
6. **Learn and build:** Explain documentation and errors; prepare a plan, ask an approved development agent to work in a sandbox branch, show its findings and request review before merges.
7. **Integrate with n8n:** Trigger an allowlisted workflow using a least-privilege service account and show execution state, without exposing a public unrestricted webhook.

### Explicit boundaries

- **Personal computer first.** Do not install on or connect to an employer-owned device, company repositories, or company data without express IT/security authorization.
- **Public repository:** No secrets, tokens, API keys, certificates, device IDs, private logs, or business content committed.
- **Offline means local features only.** Remote PC status, GitHub, web search, cloud AI, remote n8n and remote notifications need a connection.
- No always-on iOS microphone promise; iOS background execution restrictions apply. Start with push-to-talk and standard local notifications.
- No claim that arbitrary iPhone settings or third-party apps can be controlled. Use supported App Intents, Shortcuts and permissions.

## 2. Architecture

```text
iPhone / SwiftUI
  - Chat, push-to-talk, offline model adapter, encrypted memory
  - Device dashboard, jobs, projects, reminders, approvals
         |
         | Private encrypted network (evaluate Tailscale / IT-approved equivalent)
         | PLUS application-level pairing, scoped tokens, revocation
         v
Nexus Desktop Agent (Python / FastAPI)
  - Read-only OS metrics, allowlisted project inventory
  - AuthN/AuthZ, job broker, run log, health check
  - No arbitrary shell or filesystem access from the phone
         |
         +-- Local task worker: sandboxed allowlisted test/build tasks
         +-- Git status and approved GitHub integration
         +-- n8n connector (approved endpoints only)
         +-- Optional Codex / development-agent adapter
         +-- Notification delivery service (APNs provider when needed)

Shared contract: versioned OpenAPI schema, structured events, strict validation.
Local state: SQLite for agent history; iOS protected local persistence/Keychain.
Secrets: OS secret store or approved vault; never checked into Git.
```

**Connectivity:** For v0.1 bind the desktop API to `127.0.0.1`; add private remote connectivity only after pairing, endpoint authentication, network policy and tests are complete. A private VPN alone does not replace app-level access control. Prefer TLS and short-lived, device-scoped credentials; pin/verify endpoints where feasible. Avoid UPnP and public port forwarding.

**System roles:** Phone requests actions; desktop agent applies policy and executes only approved actions; external AI may propose actions but cannot bypass policy. Every job has an idempotency key, timeout, bounded output, structured status and audit record.

## 3. Planned technology

| Layer | Initial choice | Notes |
|---|---|---|
| Desktop API | Python 3.12+, FastAPI, Pydantic | Pin versions after compatibility check |
| Device metrics | psutil | Gracefully omit unsupported sensors |
| Local state | SQLite | Migrations, retention, backups |
| Desktop tests | pytest, Ruff, typing, HTTP API tests | CI on Windows and macOS |
| iPhone | SwiftUI, Swift concurrency | Xcode on Mac required for native build/signing |
| Offline AI | Apple Foundation Models when available | Feature detection and offline tests; optional local Qwen adapter |
| Speech | Apple speech APIs / on-device option | Confirm actual offline availability per device/locale |
| Device link | Tailscale or comparable private mesh | No public inbound API |
| Remote updates | APNs via minimal trusted service | Never embed APNs signing credentials in iOS app |
| Development integrations | Git/GitHub, n8n, optional Codex | Adapter-based, least-privilege |
| Cloud models | OpenAI, Gemini, Claude | Opt-in, server-side credentials, budget and data controls |

Avoid depending on one model provider. Model routing is a policy engine: check whether the task is private, offline, paid, or requires tools, and seek permission before sending sensitive content to remote APIs. Codex is a *coding agent integration*, not simply a chat model.

## 4. Phased delivery with evidence-based acceptance criteria

### P0 — Repository foundation and security baseline
- Monorepo folders: `apps/desktop-agent`, `apps/ios`, `packages/protocol`, `docs`, `tests`, `.github/workflows`.
- Set Python formatting/linting, tests, dependency locking, pre-commit secret scanning, dependency review and minimal CI.
- Document threat model, data classification, supported platforms, ADRs and coding conventions.
- Create `.gitignore` covering `.env`, keys, logs, local DBs, model weights and derived build artifacts.
- **Done:** clean checkout passes lint/tests; no plaintext test secrets; CI runs on PR; README clearly distinguishes implemented and planned functionality.

### P1 — Desktop Agent v0.1 (FIRST IMPLEMENTATION)
- Localhost-only read-only endpoints: `GET /healthz`, `GET /v1/system`, `GET /v1/projects` (approved paths only).
- Response includes `observed_at` ISO timestamp and source; avoid unsupported temperature claims.
- Configured allowlist of monitored repo folders; safe subprocess use for `git` with fixed arguments and no shell interpolation.
- Timeout, rate limiting, request IDs, sanitized logs; no root/admin requirement for routine monitoring.
- Support Windows first, macOS later using portable APIs.
- No remote job execution or network exposure at this phase.
- **Done:** Windows test host reports accurate status and a test repository; no secret values returned; unauthorized/out-of-scope paths rejected; unit/API tests and CI pass.

### P2 — Private device connection and pairing
- Paired identity, explicit device enrollment, device-scoped capabilities, revoke and rotate credentials.
- Establish private mesh connectivity (Tailscale evaluated first), no router ports opened.
- Auth every protected endpoint, replay protection, restrictive firewall rules, documented disconnection behavior.
- **Done:** phone or test client retrieves read-only status over mobile data while paired, unpaired client denied, revoked client denied, traffic protected, audit events confirmed.

### P3 — iOS Mission Control v0.1
- Native SwiftUI device list and live system cards with measured-at timestamp; stale/offline states; local cached last-known readings labeled stale.
- Pairing flow, secure token storage in Keychain, error handling, refresh, accessibility and device battery considerations.
- Basic personal dashboard and locally scheduled stand/water reminders with quiet hours.
- **Done:** works on a real iPhone, correctly distinguishes live/stale/offline, can revoke a pairing, reminders fire without desktop connection, offline interactions do not require a backend.

### P4 — Approved jobs and project monitoring
- Job registry for pre-approved scripts/tests and n8n workflows, with per-job scopes, cwd, sandbox, timeouts, concurrency and resource limits.
- Jobs report queued/running/succeeded/failed/cancelled; log redaction; idempotency; cancel; approval for write actions.
- Git and GitHub PR/CI status adapters; progress derived from explicit milestones/evidence, never invented percentages.
- **Done:** authorized test job completes and reports verified results; unsupported command refused; cancellation and retry tested; untrusted output cannot become an instruction.

### P5 — Notifications and assistant
- APNs notifications backed by a trusted service for desktop job completion, failures and approvals.
- Opt-in daily/weekly digests, quiet hours, deduplication, alert severity rules and notification privacy.
- Apple on-device conversation and push-to-talk; searchable, deletable local memory; offline mode indicators.
- **Done:** remote completion produces one meaningful notification with no sensitive payload; local assistant and reminders still work without internet.

### P6 — AI development orchestration and learning
- Optional Codex integration via isolated worktree/branch, user-approved repositories and bounded runs; test/review output before PR creation.
- Optional Claude/Gemini/OpenAI cloud model adapters; per-provider limits and cost telemetry, no secrets in client.
- Documentation and code learning mode with citations to authorized source files.
- **Done:** a mock development task produces diff, tests, and review; no automatic deploy or merge by default; errors/limits lead to safe stop.

### P7 — Reliability and production-like hardening
- Runbook for installation, upgrade, backup, restore, revoke-all, incident handling and uninstall.
- Threat modeling, penetration-style auth tests, fault injection, privacy review, mobile power profiling and cross-platform QA.
- **Done:** restore drill, security regression tests, CI, and documented manual acceptance checklist pass.

## 5. Minimal shared API contract (draft; not deployed)

```text
GET  /healthz                public only on localhost for P1; minimal content
GET  /v1/system              read:system
GET  /v1/projects            read:projects (configured allowlist)
GET  /v1/jobs/{id}           read:jobs (P4)
POST /v1/jobs                run:<registered-task> (P4, scoped and validated)
POST /v1/jobs/{id}/cancel    cancel:jobs (P4)
POST /v1/devices/pair        explicit pairing flow (P2)
POST /v1/devices/revoke      device administration (P2)
```

Version every payload and state the source, freshness and authorization scope. No `POST /execute` with arbitrary shell commands. Never expose private stack traces in API responses.

## 6. Suggested repository layout

```text
Nexus/
  README.md
  docs/
    PLAN.md
    SECURITY.md                  # P0
    ARCHITECTURE.md              # P0
    decisions/                   # P0
  apps/
    desktop-agent/               # P1
    ios/                         # P3
  packages/
    protocol/                    # P0/P1
  tests/
  .github/workflows/             # P0
```

## 7. Security controls and threat scenarios

Threats: stolen phone, stolen computer, leaked tokens, malicious PR/log contents, rogue VPN peer, API replay, prompt injection, unauthorized employer data access, remote shell misuse, costly AI loops.

Controls: local-only by default, enrollment with physical/user confirmation, scoped non-reusable credentials, TLS/private network, revocation, signed/audited job requests, strict allowlists, least privilege, isolated project worktrees, limits on time/cost/actions, rate limiting, secure logs, encrypted storage at rest, fail-closed auth, manual confirmation for destructive actions and separation of company/personal contexts.

**Fail-safe rules:** If device auth, project scope, provider policy or confirmation is unclear, deny and explain; never silently fall back to insecure networking. Do not run shell text received from a language model without mapping it to an explicit approved task.

## 8. Project status and notification semantics

- Computer heartbeat: reachable/unreachable and last verified time; no false implication that PC is off when network merely fails.
- Project status: branch, commit SHA, dirty/clean state, CI status + timestamp, automated test status + evidence, actual milestone status.
- Never mark a feature complete solely because code was committed, tests passed, or an AI claimed success.
- Alerts: job failed/completed, review required, critical disk shortage, prolonged device disconnect (opt-in), CI regression, milestone completion (verified).
- Reminders: stand, hydrate, rest, with snooze/quiet hours; user-configurable and stored locally.

## 9. First Codex work order

> Implement **P0 + P1 only** in `jasongil003/Nexus`. Start with the plan and README. Build a safe Windows-first Python/FastAPI desktop agent that exposes localhost-only read-only health, system metrics, and configured Git repo status. Use typed response schemas and UTC timestamps, psutil, strict configured path allowlists, fixed Git arguments, timeouts, rate limiting, redacted structured logs, explicit error states, pytest/Ruff/type checks and CI. Write tests for path traversal, malformed configuration, missing Git, secrets leakage, unsupported sensors, and authorization-boundary placeholders. Do not add arbitrary command execution, remote network binding, public tunnels, cloud keys, live company integrations, iOS promises, or fabricated status. Update docs and tests alongside code. Commit only after verification, report exactly what passed and what remains unimplemented.

## 10. Prioritization and non-goals

**Now:** P0, then P1.  
**Next:** Secure pairing (P2), iPhone dashboard (P3).  
**Later:** Jobs/project supervision (P4), alerts/AI assistant (P5), coding-agent integration (P6), hardening (P7).

Not in v0.1: full remote desktop, unrestricted shell, invisible screen monitoring, always-on iOS hotword, automatic production deployments, autonomous merge to main, company computer access without authorization.

## 11. Decisions to validate during implementation

- Actual desktop OS and whether personal hardware or IT-approved machine.
- Availability of a Mac with supported Xcode for iOS builds.
- Choice of private transport, enrollment UX and notification relay.
- Offline model performance, supported languages, thermal and battery profile on actual iPhone.
- Whether GitHub/n8n/cloud provider accounts may be connected; credentials and data handling policies.
- Desktop agent distribution/signing and unattended update method.
