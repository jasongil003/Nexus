# Nexus

**Nexus** is a planned offline-first personal AI assistant for iPhone with a secure, permission-controlled desktop companion.

> **Project status:** Planning. The features below are goals, not implemented capabilities.

## Vision

Use an iPhone 17 Pro Max to converse with a personal assistant, learn and organize day-to-day work, receive reminders, monitor authorized computers and projects, and orchestrate approved development workflows privately and securely.

### Planned capabilities

- **Personal assistance:** On-device AI, voice, memory, document learning and configurable stand/hydration reminders.
- **Private PC monitoring:** Health, uptime, storage, selected Git repositories and agent-managed jobs, with timestamps and clear offline states.
- **Secure task execution:** Allowlisted tasks, device pairing, scoped authorization, audit logs and explicit confirmations.
- **Project intelligence:** Evidence-based project milestones, commits, tests, CI status, job progress and completion notifications.
- **Automation:** Approved n8n workflows and optionally Codex-driven development jobs.
- **Multi-model AI:** Apple Foundation Models / optional local Qwen, plus opt-in OpenAI, Claude and Gemini through safely stored backend credentials.

## Build order

1. **P0 — Foundation:** Repository structure, shared contracts, threat model, security defaults and CI.
2. **P1 — Desktop Agent:** Localhost-only Python/FastAPI, read-only system monitoring and approved Git repo status.
3. **P2 — Secure Pairing:** Private network, per-device credentials, access scopes and revocation.
4. **P3 — iPhone Mission Control:** SwiftUI device dashboard, secure pairing and local reminders.
5. **P4 — Project Jobs:** Allowlisted tasks, n8n, GitHub and status tracking.
6. **P5 — Notifications and AI:** APNs alerts, on-device assistant, voice and memory.
7. **P6 — Development Orchestration:** Optional Codex, model routing and cost-limited execution.
8. **P7 — Hardening:** Backup, recovery, cross-platform testing and security validation.

Read the detailed **[Product & Engineering Plan](docs/PLAN.md)** for architecture, acceptance criteria, security rules, API draft and the first Codex work order.

## Security principles

- Personal computers first. Employer-owned devices or company data require explicit IT/security authorization.
- Localhost-only desktop API initially; no public inbound exposure.
- Private networking alone is not authentication. Use device pairing, narrow permissions and revocable credentials.
- No arbitrary remote shell commands, silent screen monitoring, or autonomous production deployment.
- Never commit secrets, private documents, certificates, credentials or sensitive logs.
- Offline local assistance remains usable without network access; remote status and cloud AI need connectivity.

## Current milestone

**Next: P0 and P1 — safe, read-only desktop agent with automated tests.** No executable components have been added yet.

## License

No open-source license has been selected yet. Do not assume permission to reuse third-party project code without verifying its license.
