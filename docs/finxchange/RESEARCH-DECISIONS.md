# FinXchange Research & Decisions

Version 1.0 | Research date: 2026-10-20

## Research summary
FinXchange's premise is an app-owned, approachable financial experience under poor or absent mobile data. Cached views and local drafts are feasible; settlement requires an authorized financial channel and authoritative confirmation. A sandbox or provider name alone does not prove the required capability or partner approval.

## Research sources
1. Central Bank of Nigeria — payment service provider and payment-system publications: https://www.cbn.gov.ng/PaymentsSystem/PSPs.html
2. Expo SQLite documentation: https://docs.expo.dev/versions/latest/sdk/sqlite/
3. Expo SecureStore documentation: https://docs.expo.dev/versions/latest/sdk/securestore/
4. Nigeria Data Protection Commission: https://ndpc.gov.ng/

The official CBN index is the starting point for current payment, USSD, open banking, licensing and consumer-protection rules. Expo documents persistent local SQLite and encrypted small key-value storage; neither makes the device authoritative for money movement. Verify live provider capabilities and legal obligations directly before launch.

## Architecture decisions
ADR-001: React Native + Expo + TypeScript, Android-first; validate native USSD/device requirements early.
ADR-002: Offline-aware, not offline-settling; server/provider is authoritative.
ADR-003: Provider adapters normalize bank/PSP/USSD differences.
ADR-004: Start with a modular monolith; split services only when evidence justifies the operational cost.
ADR-005: No cryptocurrency or blockchain settlement in the MVP.

## Provider research checklist
Potentially investigate licensed Nigerian banks, payment providers, USSD aggregators and bill-payment aggregators. Names previously discussed include Africa's Talking, Paystack, Flutterwave, ProvidusBank and Paga. Their appearance here does not claim support for the exact flow, an active partnership, current selection, or production approval.

For each candidate verify legal entity/licence, supported services and networks, sandbox/production onboarding, app-controlled USSD capability, auth/signatures, idempotency, status inquiry, callbacks, reversal, reconciliation, fees, settlement, limits, SLA, data handling, disputes, customer support and branding terms.

## Historical draft limits — not approved
Earlier concept notes mentioned:
- BVN or NIN: ₦50,000/day and ₦200,000 maximum.
- BVN + NIN: ₦200,000/day and ₦500,000 maximum.
- BVN + NIN + bill: ₦5,000,000/day and unlimited maximum.
These are unapproved historical assumptions, not legal limits or a launch recommendation. Reassess with the selected licensed partner, current CBN rules, risk controls, KYC/AML obligations and consumer-protection requirements. Implement approved limits server-side and make them configurable.

## Open decisions
| Decision | Owner | Evidence | Release impact |
|---|---|---|---|
| Operating model/licensing | Founder + counsel | Written assessment and agreement | Blocker |
| Bank/PSP/USSD provider | Integration lead | Capability matrix, sandbox, SLA | Blocker |
| Hidden/app-controlled USSD feasibility | Mobile lead + provider | Working proof and written confirmation | Blocker for that promise |
| Backend choice | Tech lead | Repo audit and team skills | Blocker |
| Wallet/ledger responsibility | Founder + compliance | Funds flow/accounting design | Blocker if holding funds |
| KYC tiers and limits | Compliance + partner | Current rules and partner policy | Blocker |
| Cache retention | Security/privacy owner | Threat model/data inventory | Before pilot |
| Languages/cooperative features | Product + partner | User research and data ownership | Later |

## Limitations
This pack is not an exhaustive audit of every global fintech, not proof of provider relationships or regulatory approval, and not a claim that these designs are already implemented. Revalidate live rules, provider capability and repository state before production.