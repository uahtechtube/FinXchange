# FinXchange REST API Contract — Draft

Version 1.0 | 2026-10-20

## Conventions
Prefix /api/v1; HTTPS JSON; UTC ISO-8601 timestamps. Auth required for user resources. Mutating transaction requests require Idempotency-Key. Include request_id in every response and immutable transaction_id for transactions. Cursor pagination for history. Never expose raw provider payloads, secrets or stack traces.

## Endpoint map
- POST /auth/otp/request — request verification code.
- POST /auth/otp/verify — verify code and create session.
- POST /auth/session/refresh — rotate session.
- POST /auth/session/revoke — revoke session.
- GET /me — profile and KYC/policy state.
- PATCH /me/preferences — language/preferences.
- GET /dashboard — snapshot with explicit as_of timestamps.
- GET/POST /beneficiaries — list/create.
- PATCH /beneficiaries/{id} — edit/deactivate.
- POST /beneficiaries/{id}/validate — provider validation.
- GET /services — supported service catalogue.
- POST /transactions — submit transaction.
- GET /transactions — cursor-paginated history.
- GET /transactions/{id} — status and timeline.
- POST /transactions/{id}/check-status — safe provider status inquiry.
- POST /disputes — create dispute.
- GET /disputes/{id} — dispute status.

## Transaction request example
    {
      "type": "transfer",
      "beneficiary_id": "UUID",
      "amount_minor": 250000,
      "currency": "NGN",
      "narration": "Optional note",
      "confirmation_version": "quote-or-fee-version"
    }

Header: Idempotency-Key: unique-random-key. Amount is integer minor units according to the currency exponent. The server calculates and validates fees/total; client-provided fees are not trusted.

## Response example
    {
      "request_id": "req_example",
      "transaction": {
        "id": "UUID",
        "status": "pending_provider",
        "amount_minor": 250000,
        "fee_minor": 0,
        "total_debit_minor": 250000,
        "currency": "NGN",
        "created_at": "2026-10-20T12:00:00Z",
        "status_url": "/api/v1/transactions/UUID"
      }
    }

Illustrative only. Real schema and fee rules must match selected provider contract and approved policy.

## Error format
    {
      "request_id": "req_example",
      "error": {
        "code": "TRANSACTION_STATUS_UNKNOWN",
        "message": "We could not confirm the result yet. Check status before trying again."
      }
    }

Required codes: VALIDATION_ERROR, AUTHENTICATION_REQUIRED, STEP_UP_REQUIRED, RATE_LIMITED, PROVIDER_UNAVAILABLE, LIMIT_EXCEEDED, IDEMPOTENCY_CONFLICT, TRANSACTION_STATUS_UNKNOWN, TRANSACTION_NOT_FOUND, KYC_ACTION_REQUIRED, SERVICE_UNAVAILABLE.

## Webhooks
Verify provider signature and timestamp; reject replay; deduplicate by provider + event ID; persist to durable inbox before acknowledging; process idempotently; do not trust event ordering; query authoritative status if callbacks conflict; log redacted metadata only.