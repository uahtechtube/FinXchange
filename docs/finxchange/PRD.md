# Product Requirements Document — FinXchange

Version 1.0 | 2026-10-20 | Status: proposed baseline

## Product summary
FinXchange is a Nigeria-first mobile financial access app intended to improve usability for people with poor, costly, or absent mobile data. It combines an approachable banking-style UI, permitted cached data, safe local drafts, and authorized banking/payment/USSD integrations. Cooperative member services may become a differentiator through a qualified partner. Cryptocurrency and blockchain settlement are out of scope for this MVP.

**Platform:** Android-first React Native/Expo mobile app. **Market:** Nigeria initially. **Languages:** English first; Hausa, Yoruba, Igbo, Nigerian Pidgin, and Kanuri staged later.

## Problem
Online-first apps can fail on unreliable networks. USSD can work without mobile data but is often text-heavy and fragmented. Users need a clear interface that explains what is cached, what has been sent, and whether a provider has confirmed a transaction.

## Goals and proposed metrics
- Core navigation and permitted previously synced information work offline.
- Every displayed cached value has a last-synced timestamp.
- Ambiguous requests become pending/unknown, not falsely successful or automatically resubmitted.
- No duplicate debit caused by client retry logic in automated tests.
- Track provider-confirmed success rate, pending duration, reversals, disputes per 1,000 transactions, crash-free sessions, and task completion.
These are targets to validate, not current performance claims.

## Users
1. Everyday customer managing beneficiaries, transfers, airtime, bills, and activity.
2. Low-connectivity customer who needs to understand offline limitations and resume safely.
3. Cooperative member viewing services through an authorized cooperative partner (later).
4. Support agent investigating transactions with minimum necessary access.
5. Operations/compliance staff monitoring providers, reconciliation, disputes, and audit trails.

## MVP scope
- Onboarding, consent/privacy notice, phone verification via approved provider.
- Secure authentication, session/device management, profile and partner-required KYC state.
- Dashboard with cached snapshots and stale/offline labels.
- Beneficiary management and transaction history.
- Transfer through contracted authorized provider.
- Airtime/data/bill purchase only where an approved integration exists.
- Local-only transaction drafts, review/reconfirmation, idempotent submission.
- Pending status, status inquiry, reconciliation, receipt, support/dispute entry.
- English, accessible low-bandwidth UI, feature flags, provider health controls.

## Later / excluded
Later: cooperative membership/contributions through approved partner, voice guidance, additional languages and African markets, deeper analytics after governance review.
Excluded from MVP: FinXchange independently holding deposits, issuing e-money or lending without an approved legal model; crypto/blockchain; guaranteed invisible control of native USSD; offline final settlement; fabricated local success receipts; storing raw bank credentials, OTPs, or PINs.

## Functional requirements
- FR-01 Onboard and verify user server-side.
- FR-02 Authenticate securely; revoke sessions and step up verification for risky actions.
- FR-03 Show cached data with last-sync time and stale status.
- FR-04 Create, edit, deactivate beneficiaries; validate destination before payment.
- FR-05 Validate recipient, amount, currency, fee, limits, KYC, and risk server-side.
- FR-06 Provide only supported, enabled service catalogue items.
- FR-07 Show transaction history, immutable reference, amount, currency, timestamps, status.
- FR-08 Require idempotency keys; check status before retrying ambiguous requests.
- FR-09 Mark offline drafts “Not sent — no payment has been made.”
- FR-10 Reconcile provider callbacks and status polling; audit transitions.
- FR-11 Provide transaction-specific support/dispute intake.
- FR-12 Externalize UI strings for localization.
- FR-13 Restrict operations tools by role and log privileged access.
- FR-14 Keep notifications free of full account details.

## Transaction states
Draft/local queue → submitted → pending provider → succeeded, failed, reversed, expired, or needs review. Drafts are not server transactions. Submitted means accepted for processing, not settled. Success requires authoritative provider confirmation and reconciliation policy. Timeouts remain pending until checked; state changes are server-authoritative.

## Non-functional requirements
Security: TLS, least privilege, secret management, secure token storage, server authorization, rate limits, audit logs. Reliability: durable writes, idempotency, reconciliation. Performance: target cached first useful screen ≤2 seconds on representative low-end Android devices, measured in testing. Accessibility: scalable text, contrast, screen-reader labels and large touch targets. Privacy: data minimization, retention and deletion rules. Observability: redacted logs, provider metrics, pending-age alerts.

## Acceptance / release gates
- Provider sandbox tests pass for timeout, duplicate request/callback, reversal and reconciliation.
- No unresolved critical/high security findings.
- Offline and weak-network behavior tested on target Android devices.
- User testing confirms users understand stale, draft, pending, and success states.
- Support, incident, dispute and reconciliation runbooks are ready.
- Legal/compliance and partner approvals are documented.

## Dependencies and risks
Provider capability, contract, SLA, fees, production approval, KYC/AML responsibilities, licensing model and dispute process must be confirmed. App-controlled hidden USSD cannot be promised until tested on target devices and confirmed by provider/operator. Draft transaction limits from prior ideation are not approved policy and must be reviewed with partner/compliance.

## Reference-video-informed assistant requirements
The user-provided Vusa AI sample demonstrates a chat-first interface with quick commands (Check balance, Send money, Cash out), text input, microphone/audio affordances, a mobile-money PIN gate, a transfer summary with confirm/cancel, and a response that explains the amount, fee, recipient/service and remaining balance. This is design inspiration only; it does not prove provider integration, real transaction success, or offline execution. Add an optional FinXchange Assistant as a progressive enhancement, not the only way to use the app. Supported intents should include balance inquiry, transfer preparation, and cash-out only when authorized provider capability exists. Natural language must resolve to typed intents; ambiguous requests require clarification. Money movement must show recipient, amount, currency, fee and total debit and require explicit confirmation. Voice and multilingual features are staged and feature-flagged. Never ask for or log a PIN in chat; use a provider-approved secure authorization flow. The assistant must distinguish cached information, drafts, submitted requests, pending outcomes, and confirmed transactions.