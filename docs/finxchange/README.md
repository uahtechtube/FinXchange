# FinXchange Documentation

Version 1.0 · Reviewed 2026-10-20 · Proposed baseline

- [PRD](./PRD.md)
- [TRD](./TRD.md)
- [Application Flow](./APP-FLOW.md)
- [Design Brief](./DESIGN-BRIEF.md)
- [Database Schema](./DATABASE-SCHEMA.md)
- [Implementation Plan](./IMPLEMENTATION-PLAN.md)
- [API Contract](./API-CONTRACT.md)
- [Security & Compliance](./SECURITY-COMPLIANCE.md)
- [Research & Decisions](./RESEARCH-DECISIONS.md)

FinXchange is a Nigeria-first, offline-aware financial access app. The app may display permitted cached information and save local drafts without mobile data. A transfer, bill payment, airtime purchase, or balance refresh is not complete until an authorized provider confirms it. Offline UI is not offline settlement.

The app should own the user experience where an authorized API or provider-controlled USSD integration permits it. Ordinary Android apps cannot universally hide or control native USSD sessions; this requires a tested proof-of-concept and written provider confirmation.

These documents define a target baseline, not proof that every feature is implemented, partner-approved, licensed, or production-ready.