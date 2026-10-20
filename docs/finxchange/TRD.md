# Technical Requirements Document — FinXchange

Version 1.0 | 2026-10-20 | Proposed architecture

## Principles
API-first; provider-agnostic; offline-aware but server-authoritative; secure-by-default; idempotent and auditable money movement; start modular rather than prematurely distributed.

## Proposed stack
- Mobile: React Native + Expo + TypeScript, Android-first.
- Local cache: expo-sqlite for cached snapshots, sync metadata and local drafts.
- Small secrets/tokens: expo-secure-store and platform keystore/keychain.
- Backend: choose one after repository audit — NestJS/TypeScript or Laravel/PHP. Do not maintain competing backends for MVP.
- Server database: PostgreSQL with versioned migrations.
- Async work: transactional outbox and database-backed worker initially; managed queue if scale requires.
- API: REST + OpenAPI, versioned under /api/v1.
- CI/CD: lint, typecheck, unit/integration tests, dependency and secret scans, staged deployment.
Official references: https://docs.expo.dev/versions/latest/sdk/sqlite/ and https://docs.expo.dev/versions/latest/sdk/securestore/.

## Components
1. Mobile app: UI, connectivity state, local cache, draft queue, localization.
2. API edge: auth, validation, rate limits, request IDs.
3. Identity: users, devices, sessions, OTP adapter and step-up checks.
4. Profile/beneficiary module.
5. Transaction orchestration: limits, idempotency, transaction record, provider dispatch.
6. Provider adapters for transfer, bill, airtime/data, balance/history and status checks.
7. Webhook ingress: signature validation, replay defense, durable inbox.
8. Reconciliation worker: status checks, exception queue and resolution.
9. Notifications and support/disputes.
10. Operations console with role-based access, provider health, kill switches and audit logs.

## Connectivity model
Online: app → FinXchange API → policy and limits → durable transaction record → authorized partner → callback/status inquiry → reconciliation → app.
Offline: permitted cached views remain available; a local draft may be saved. On reconnect, revalidate beneficiary, fees, limits, risk and provider availability, then ask the user to review and confirm. Never claim settlement offline.
USSD: use a documented, authorized API/SDK/session integration. Native USSD UI cannot be assumed controllable or hideable across Android devices. Prove feasibility on target devices before committing to that UX.

## API requirements
- HTTPS and JSON, UTC ISO-8601 timestamps.
- Authenticated endpoints and short-lived sessions with revocation.
- Idempotency-Key required for transaction creation.
- Every response includes request_id; every transaction has immutable transaction_id.
- Cursor pagination for history.
- Safe error object: code, user-safe message, request_id, optional field errors.
- Webhooks validate signature, timestamp window, provider/event uniqueness, and event ordering; persist before acknowledging.
- Never expose upstream secrets, raw payloads, or stack traces.

## Security and reliability
Use parameterized queries, server-side authorization and risk limits, secure secret store, encrypted backups, redacted logs, rate limits, device/session revocation, and least privilege. Represent money as integer minor units plus explicit ISO currency; never floating point. Persist transaction intent and outbox atomically. Timeout means unknown/pending, not automatic failure. Duplicate callbacks must be harmless. Never blind-retry a possibly submitted payment. Audit admin actions and manual state changes.

## Testing
Unit tests for fees, limits, validation and state transitions; DB integration tests; provider adapter contract tests; E2E onboarding/transfer/timeout/offline draft/resume/reversal/dispute; chaos tests for delayed responses, duplicate callbacks and outages; security tests for authz, replay, rate limits and secret leaks; low-end Android and weak-network matrix.

## Deployment and operations
Separate development, staging and production credentials/databases. Never copy production PII into development. Measure API latency, provider success/error rates, pending age, reconciliation backlog, callback delay, crashes and support contacts. Require staging smoke tests and explicit production approval.

## Decisions to resolve
Audit current source before selecting backend/navigation; prove USSD behavior; obtain provider contracts and sandbox; confirm regulatory operating model; decide whether FinXchange merely mirrors provider transactions or is responsible for a regulated double-entry ledger.