# FinXchange Database Schema

Version 1.0 | 2026-10-20 | Proposed PostgreSQL baseline

## Rules
PostgreSQL is the authoritative server store; device SQLite is only cache/draft storage. Money uses integer minor units plus ISO currency code. Never use floating point. Keep provider payloads minimal and redact sensitive data. Never store raw OTPs, PINs, bank credentials or card security codes. A transaction record is not automatically a double-entry ledger.

## Relationships
Users own devices, sessions, beneficiaries, consents, transactions and disputes. Transactions have provider attempts and immutable status events. Providers own provider attempts. Privileged operations create audit events.

## Initial SQL
The following is a design baseline; turn it into reviewed, versioned migrations before use.

    CREATE EXTENSION IF NOT EXISTS pgcrypto;

    CREATE TYPE transaction_status AS ENUM
      ('submitted','pending_provider','succeeded','failed','reversed','expired','needs_review');

    CREATE TABLE users (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      phone_e164 VARCHAR(20) UNIQUE NOT NULL,
      display_name VARCHAR(120),
      preferred_language VARCHAR(12) NOT NULL DEFAULT 'en',
      account_status VARCHAR(24) NOT NULL DEFAULT 'pending_verification'
        CHECK (account_status IN ('pending_verification','active','restricted','closed')),
      kyc_status VARCHAR(24) NOT NULL DEFAULT 'not_started'
        CHECK (kyc_status IN ('not_started','pending','verified','rejected','review')),
      created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
      updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
    );

    CREATE TABLE devices (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      user_id UUID NOT NULL REFERENCES users(id),
      device_public_id VARCHAR(128) NOT NULL,
      platform VARCHAR(16) NOT NULL CHECK (platform IN ('android','ios','web')),
      app_version VARCHAR(32),
      trusted_at TIMESTAMPTZ,
      revoked_at TIMESTAMPTZ,
      created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
      UNIQUE(user_id, device_public_id)
    );

    CREATE TABLE sessions (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      user_id UUID NOT NULL REFERENCES users(id),
      device_id UUID REFERENCES devices(id),
      refresh_token_hash TEXT NOT NULL,
      expires_at TIMESTAMPTZ NOT NULL,
      revoked_at TIMESTAMPTZ,
      created_at TIMESTAMPTZ NOT NULL DEFAULT now()
    );

    CREATE TABLE consents (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      user_id UUID NOT NULL REFERENCES users(id),
      consent_type VARCHAR(64) NOT NULL,
      policy_version VARCHAR(32) NOT NULL,
      granted BOOLEAN NOT NULL,
      captured_at TIMESTAMPTZ NOT NULL DEFAULT now(),
      withdrawn_at TIMESTAMPTZ
    );

    CREATE TABLE beneficiaries (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      user_id UUID NOT NULL REFERENCES users(id),
      display_name VARCHAR(120) NOT NULL,
      institution_code VARCHAR(64),
      destination_token TEXT,
      destination_masked VARCHAR(64),
      currency CHAR(3) NOT NULL DEFAULT 'NGN',
      validation_status VARCHAR(24) NOT NULL DEFAULT 'unverified'
        CHECK (validation_status IN ('unverified','valid','invalid','expired')),
      active BOOLEAN NOT NULL DEFAULT TRUE,
      created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
      updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
    );

    CREATE TABLE providers (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      code VARCHAR(64) UNIQUE NOT NULL,
      display_name VARCHAR(120) NOT NULL,
      capability JSONB NOT NULL DEFAULT '{}'::jsonb,
      environment VARCHAR(16) NOT NULL CHECK (environment IN ('sandbox','production')),
      enabled BOOLEAN NOT NULL DEFAULT FALSE,
      health_status VARCHAR(24) NOT NULL DEFAULT 'unknown',
      created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
      updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
    );

    CREATE TABLE transactions (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      user_id UUID NOT NULL REFERENCES users(id),
      beneficiary_id UUID REFERENCES beneficiaries(id),
      type VARCHAR(32) NOT NULL CHECK (type IN ('transfer','airtime','data','bill','exam_pin','cooperative')),
      status transaction_status NOT NULL DEFAULT 'submitted',
      amount_minor BIGINT NOT NULL CHECK (amount_minor > 0),
      fee_minor BIGINT NOT NULL DEFAULT 0 CHECK (fee_minor >= 0),
      currency CHAR(3) NOT NULL,
      client_idempotency_key VARCHAR(128) NOT NULL,
      provider_reference VARCHAR(160),
      narration VARCHAR(160),
      destination_snapshot JSONB NOT NULL DEFAULT '{}'::jsonb,
      failure_code VARCHAR(64),
      created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
      submitted_at TIMESTAMPTZ,
      completed_at TIMESTAMPTZ,
      updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
      UNIQUE(user_id, client_idempotency_key)
    );

    CREATE INDEX idx_transactions_user_created ON transactions(user_id, created_at DESC);
    CREATE INDEX idx_transactions_status_created ON transactions(status, created_at);

    CREATE TABLE provider_attempts (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      transaction_id UUID NOT NULL REFERENCES transactions(id),
      provider_id UUID NOT NULL REFERENCES providers(id),
      attempt_number INTEGER NOT NULL CHECK (attempt_number > 0),
      request_correlation_id VARCHAR(128) NOT NULL,
      provider_reference VARCHAR(160),
      outcome VARCHAR(24) NOT NULL DEFAULT 'unknown'
        CHECK (outcome IN ('accepted','declined','timeout','error','unknown')),
      safe_response_code VARCHAR(64),
      started_at TIMESTAMPTZ NOT NULL DEFAULT now(),
      finished_at TIMESTAMPTZ,
      UNIQUE(transaction_id, attempt_number)
    );

    CREATE TABLE transaction_events (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      transaction_id UUID NOT NULL REFERENCES transactions(id),
      event_type VARCHAR(64) NOT NULL,
      from_status transaction_status,
      to_status transaction_status,
      source VARCHAR(32) NOT NULL,
      provider_event_id VARCHAR(160),
      safe_metadata JSONB NOT NULL DEFAULT '{}'::jsonb,
      occurred_at TIMESTAMPTZ NOT NULL DEFAULT now()
    );

    CREATE TABLE disputes (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      transaction_id UUID NOT NULL REFERENCES transactions(id),
      user_id UUID NOT NULL REFERENCES users(id),
      category VARCHAR(64) NOT NULL,
      description TEXT,
      status VARCHAR(24) NOT NULL DEFAULT 'open'
        CHECK (status IN ('open','investigating','resolved','rejected')),
      resolution_summary TEXT,
      created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
      resolved_at TIMESTAMPTZ
    );

    CREATE TABLE audit_events (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      actor_user_id UUID REFERENCES users(id),
      actor_type VARCHAR(24) NOT NULL,
      action VARCHAR(96) NOT NULL,
      target_type VARCHAR(64) NOT NULL,
      target_id UUID,
      request_id VARCHAR(128),
      safe_metadata JSONB NOT NULL DEFAULT '{}'::jsonb,
      created_at TIMESTAMPTZ NOT NULL DEFAULT now()
    );

## Required follow-up schema work
- Add a durable webhook inbox with provider-scoped unique event IDs, signature metadata and replay controls.
- Ensure provider references are unique within provider scope where required.
- Define currency exponent, fee consistency and permitted currency per provider.
- Add application-level tenant/user authorization and database least-privilege roles.
- Define retention, export, deletion, encrypted backups and tested restore.
- Restrict destination_snapshot to an allowlist of masked/tokenized fields.
- If FinXchange becomes responsible for customer funds/balances, design a separately reviewed double-entry ledger with immutable journal entries, balanced postings and reconciliation.

## Mobile SQLite cache
Conceptual tables: cached_profile, cached_beneficiaries, cached_transactions, sync_metadata and local_drafts. Include fetched_at and schema_version. Drafts must be marked not_sent and cleared/retained according to logout and device-revocation policy. Keep tokens in SecureStore, not SQLite.