# FinXchange Design Brief

Version 1.0 | 2026-10-20

## Objective
A trustworthy, modern, low-bandwidth financial UI that is easy for first-time digital-finance users and precise about transaction status. Visual polish must never imply an offline or pending action has settled.

## Brand direction
FinXchange — “Smart Digital Banking for Everyone.” Calm, clear, respectful and practical. Use clean banking surfaces, white/light backgrounds, deep blue and green/teal accents, restrained gradients, simple icons and subtle elevation. Check existing repository brand assets before inventing logo variants or exact hex values.

## Design principles
1. Status before decoration: offline, stale, pending, failed, reversed and success must be distinct and labelled in text.
2. No false certainty: green success styling only after server/provider confirmation.
3. Plain language, low payload and predictable navigation.
4. Transparent recipient, amount, currency, fee, total debit and expiry before confirmation.
5. Errors explain what is known and the safest next action.
6. Accessible contrast, scalable text, screen-reader labels and touch targets around 44–48 dp where feasible.
7. Localization-ready layouts; no text baked into images.
8. Privacy by default; avoid sensitive notification previews.

## Core screens
- Welcome/language and phone/OTP.
- Dashboard with permitted snapshots, quick actions, recent activity, last sync.
- Transfer and beneficiary selection.
- Confirmation with recipient, amount, fee and total.
- Transaction detail with reference, timeline, status and support.
- Services catalogue for airtime/data/bills.
- Beneficiaries and profile/security.
- Support/dispute intake.
- Operations console for masked transaction records, provider health and reconciliation exceptions.

## Components
Primary/secondary/destructive buttons; labelled fields; amount input with explicit currency; accessible status pill; offline/stale banner; fee breakdown; masked beneficiary row; transaction timeline; receipt; loading/empty/error states; confirmation sheet; language selector; connectivity indicator.

## Interaction requirements
- Confirmation before financial submission.
- Disable duplicate taps but retain server idempotency.
- Timeout shows Pending/Checking and “Check status,” not “Pay again.”
- Offline drafts persistently say “Not sent.”
- Cached data appears immediately with timestamp.
- Never use colour alone to convey state.
- Destructive actions require confirmation.

## Proposed tokens
Centralized theme: brand primary deep blue; secondary green/teal; white surface; subtle neutral surface; near-black primary text; dark-neutral secondary text; accessible success green, warning amber, error red, pending blue/amber. Use 4 dp spacing scale, 16 dp card radius, 10–12 dp control radius, system sans-serif and tabular numerals where available. Validate all contrast ratios and check existing brand assets before final values.

## Copy examples
Use: “Saved on this device. Not sent.” “We’re checking the provider’s response. Don’t make another payment yet.” “Last updated 10:42 AM.” “We couldn’t confirm the result. Your transaction is pending review.”
Avoid: “Offline payment successful,” unsupported “instant transfer” claims, or “failed” when provider outcome is unknown.

## Research
Prototype with 5–8 representative users. Test whether users distinguish cached/live balances, understand offline drafts, recover from transfer timeout, find references and raise disputes. Measure task completion and misunderstandings; revise accordingly.

## Reference-video design direction: conversational assistant
Add an optional Chat tab with a clear text composer, send action, microphone affordance only when voice is supported, audio playback controls, and Quick Commands such as Check balance and Send money. Keep conventional Home, Services, Favourites/Beneficiaries, Activity, and Settings navigation available; final tab labels should be validated against the existing app. For transfers, render the assistant's interpretation as a structured review card rather than a free-form message: recipient, destination (masked), amount/currency, fee, total debit, and Confirm/Cancel. Show progress and state labels clearly. A spoken or chat response is never itself proof of settlement. Use provider-owned secure PIN/authorization UI where required; never collect PINs in the chat composer.