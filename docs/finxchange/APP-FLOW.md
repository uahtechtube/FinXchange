# FinXchange Application Flow

Version 1.0 | 2026-10-20

## Main navigation
Unauthenticated: Splash → Welcome → Language → Phone → Consent/privacy → OTP verification → Security setup → required KYC steps.
Authenticated: Dashboard → Activity → Services → Beneficiaries → Notifications → Profile/Security → Help/Disputes.

## Onboarding
1. Check minimum app version.
2. Select language.
3. Enter phone number and display privacy notice/terms.
4. Request OTP from approved provider with resend and attempt limits.
5. Verify OTP server-side and create session/device.
6. Complete partner-required KYC; pending/rejected status must be explicit.
7. Configure app lock/biometric preference.
8. Show dashboard and sync status.
Failures: expired OTP → request new; rate limited → show wait; provider unavailable → do not mark verified; KYC pending → explain service restrictions.

## Dashboard and offline
1. Load permitted cached snapshots immediately.
2. Show last sync timestamp and stale/offline indicator.
3. Refresh from API when connected.
4. Disable or explain features requiring live validation while offline.
5. Allow local draft only where supported.
6. On reconnect, refresh services, beneficiary, fee, limits and risk; require user review before submission.

## Transfer
1. Choose Transfer and beneficiary.
2. Enter amount and optional narration.
3. Backend validates destination, currency, fees, limits, KYC, risk and provider availability.
4. Show confirmation: recipient, amount, fee, total debit and any quote expiry.
5. User confirms and completes step-up authentication if required.
6. Submit with unique Idempotency-Key.
7. Backend records transaction before dispatching to provider.
8. Confirmed result → reconcile and show receipt. Explicit final failure → safe explanation. Timeout/ambiguous response → Pending; check status rather than submitting again.
9. History contains reference, amount/currency, timestamp, status and support route.

## Airtime/data/bills
Choose service → provider/operator → destination/customer reference → validate/quote → show fee and total → confirm/authenticate → submit idempotently → wait for final provider status → receipt or pending/support flow. If debited but service not delivered, use provider-specific reversal and dispute process; do not advise retry before status check.

## Offline draft and sync
Create draft → validate local format only → save locally → label “Not sent — no payment has been made” → connectivity returns → fetch current fees/limits/beneficiary state → user reviews and confirms → submit once with idempotency key → reconcile → final state.
Never queue OTPs, PINs, or provider credentials. If a prior request may have reached the server, query status before retrying.

## USSD flow
Preferred: FinXchange UI → authorized provider session/API → bank/MNO/provider → normalized result → FinXchange status UI.
Fallback only where supported: clearly hand off to native USSD. Do not promise to suppress native UI across devices. Interrupted session = unknown until provider confirms.

## Status language
- Draft: saved on this device; not sent.
- Submitted: accepted by FinXchange server for processing.
- Pending: provider has not supplied final authoritative result.
- Successful: provider confirmation and reconciliation criteria satisfied.
- Failed: confirmed final rejection/failure.
- Reversed: provider confirms reversal.
- Needs review: evidence conflicts or cannot be obtained; support investigation required.

## Mermaid diagrams
### High-level
flowchart TD
  A[Launch] --> B{Session valid?}
  B -- No --> C[Sign in / onboarding]
  C --> D[Verification]
  D --> E[Dashboard]
  B -- Yes --> E
  E --> F{Network?}
  F -- Online --> G[Refresh API data]
  F -- Offline --> H[Cached data + stale label]
  E --> I[Prepare action]
  I --> J{Live validation needed?}
  J -- Yes, offline --> K[Explain unavailable or save draft]
  J -- No --> L[Review and submit]
  G --> L
  L --> M[Idempotent server + provider]
  M --> N{Authoritative outcome?}
  N -- Yes --> O[Final status / receipt]
  N -- Unknown --> P[Pending + reconciliation]
  P --> N

### Transfer sequence
sequenceDiagram
  participant U as User
  participant A as App
  participant F as FinXchange API
  participant D as Database
  participant P as Authorized Provider
  U->>A: Confirm transfer
  A->>F: POST transaction + Idempotency-Key
  F->>D: Create intent
  F->>P: Submit transfer
  P-->>F: Success, failure, or timeout
  F->>D: Persist state
  P-->>F: Signed callback if supported
  F->>D: Reconcile idempotently
  A->>F: Check transaction status
  F-->>A: Authoritative state
  A-->>U: Receipt or pending guidance