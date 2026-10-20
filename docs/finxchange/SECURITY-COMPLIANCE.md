# FinXchange Security, Privacy & Compliance

Version 1.0 | 2026-10-20

This is an engineering checklist, not legal advice or evidence of regulatory approval.

## Operating model gate
Before handling real customer funds, document whether FinXchange is a technology provider to a licensed institution, an authorized agent, a licensed payment/wallet operator, or another approved model. That choice determines licensing, KYC/AML, safeguarding, dispute, reporting, outsourcing and support obligations. Do not describe FinXchange as a bank or claim it holds deposits unless legally accurate and approved.

## Official research starting points
- Central Bank of Nigeria payment service providers and payment-system publications: https://www.cbn.gov.ng/PaymentsSystem/PSPs.html
- Nigeria Data Protection Commission: https://ndpc.gov.ng/
Check current CBN rules and publications relevant to USSD, open banking, payment services, bill payments, consumer protection and electronic channels. Rules change; counsel and partner compliance teams must confirm applicability and current versions. Exposure drafts are not final rules.

## Authentication
Rate-limit OTP requests and verification. Use short-lived access tokens, refresh-token rotation and session revocation. Store tokens in secure device storage; never hardcode secrets. Use step-up authentication for new beneficiaries and risk-triggered transactions. Do not log OTPs, PINs, passwords, tokens, full account numbers or identity document contents.

## Transaction integrity
- Unique idempotency key per logical transaction.
- Server-side amount, currency, fee, limit, KYC, destination and risk validation.
- Confirmation immediately before submission.
- Verify webhook signatures and prevent replay.
- Ambiguous timeout stays pending/unknown.
- Reconciliation and dispute workflow retain evidence.
- Client cannot set success state.
- No blind retry after possible debit.
- Manual status/limit changes require authorization and audit.

## Data protection
Define controller/processor roles with partners, data inventory, purpose/lawful basis, consent where required, retention, access controls and deletion process. Minimize local cache. Encrypt in transit/at rest and backups. Mask identifiers in logs/support. Define breach response, vendor review and privacy notice. Do not use financial/identity data for model training or marketing without approved legal basis and policy.

## Operations and fraud
Use partner/compliance-approved server-side limits, velocity controls and anomaly monitoring. Maintain provider kill switch, incident playbook, dual control for high-impact admin actions, periodic access review, reconciliation alerts, customer support scripts and incident simulations.

## USSD-specific checks
Confirm provider/operator authorization; session start/timeout/cancel/result semantics; fees; network/device coverage; consent/disclosures; callback behavior; production contract; and whether the requested app-controlled experience is actually supported. Test target Android versions and dual-SIM devices. Do not promise hidden native USSD unless proven.

## Pre-production checklist
- [ ] Operating model and regulatory responsibilities approved.
- [ ] Partner contract, production access, SLA, reversal/dispute rules approved.
- [ ] Threat model and privacy impact assessment completed where required.
- [ ] Penetration test completed; critical/high issues resolved.
- [ ] Secret scan and dependency audit clean.
- [ ] Backup/restore, incident response and access review tested.
- [ ] Reconciliation/support runbooks operational.
- [ ] Privacy notice, terms, fees, limits and product claims approved.