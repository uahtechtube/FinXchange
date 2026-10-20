# FinXchange Implementation Plan

Version 1.0 | 2026-10-20 | Estimates are indicative and require a repo audit.

## Phase 0 — Audit and feasibility (about 1 week)
- Inspect repository structure, app framework, current UI, backend, DB, tests, deployment and duplicate/abandoned variants.
- Run current app, lint, typecheck, tests and build; document failures.
- Map features to PRD and select one backend.
- Create provider capability matrix covering transfer, status, callbacks, reversal, bills, sandbox, production eligibility, fees and SLA.
- Prove USSD behavior on target Android devices; get written partner confirmation.
- Obtain legal/compliance review of operating model.
Exit: inventory, risk register, architecture decision record, buildable baseline.

## Phase 1 — Foundation (1–2 weeks)
Set conventions, environment config, secrets, request IDs, redacted logs, API versioning, DB migrations, CI, dependency scans, design tokens, navigation and localization framework.
Exit: reproducible build and passing migration tests.

## Phase 2 — Identity and offline cache (1–2 weeks)
Implement OTP adapter, sessions/devices/revocation, consent, partner KYC status, SQLite cache, last-sync metadata, offline banner, local draft lifecycle and logout cleanup.
Exit: offline tests pass; cached information never appears live without a timestamp.

## Phase 3 — Transaction core (2–3 weeks)
Implement guarded transaction state machine, idempotency, mock provider adapter, transfer confirmation, pending/status inquiry, webhook inbox, reconciliation worker, audit trail and support search.
Exit: timeout, duplicate request/callback and provider failure tests pass.

## Phase 4 — Services and controlled pilot (2–3 weeks)
Add contracted airtime/data/bill integration, provider availability/kill switch, fee transparency, reversal/dispute flows, security review, partner sandbox certification and small consented pilot.
Exit: product, engineering, compliance and partner sign-off; runbooks active.

## Phase 5 — Post-MVP
Local-language rollout (Hausa, Yoruba, Igbo, Pidgin, Kanuri), voice guidance, cooperative workflows through approved partner, more countries only after local compliance/provider readiness. Split services only if measured scale/team boundaries justify it.

## Roles
Product owner: scope and acceptance. Mobile engineer: app/offline/device testing. Backend engineer: API/state machine/adapters/migrations. QA: automated and provider-failure tests. Security/compliance: threat model and approvals. Integration lead: contracts, sandbox, SLAs. Design/research: prototype, accessibility and localization. One person may cover multiple roles, but independent security/compliance review is preferable.

## Priorities
P0: repo audit, authentication/revocation, authorized provider, transaction state machine, idempotency, pending/status/reconciliation, offline/draft copy, fees/confirmation, audit logs, compliance sign-off.
P1: service catalogue, support/disputes, operations console, provider kill switch, localization framework, metrics/alerts.
P2: additional languages, voice, cooperative features, multi-country expansion, AI features after data governance review.

## Test and release gates
Unit/integration tests on each PR; provider contract tests; E2E onboarding, transfer, timeout, offline draft/resume, reversal and dispute; chaos tests for delays, dropped responses and duplicate callbacks; secrets/dependency scans, authorization tests and penetration test before public launch. Production release requires staging smoke tests, partner approval, support/reconciliation runbooks and rollback/kill-switch plan.