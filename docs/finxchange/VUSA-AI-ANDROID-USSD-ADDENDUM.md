# Product Requirements Addendum — Vusa AI Reference and Android USSD Feasibility

**Date:** 2026-10-20  
**Status:** Proposed; architecture and policy feasibility must be validated before implementation.

## Product direction

FinXchange should explore a chat-first financial assistant inspired by the Vusa AI demonstration. The interface should help users express common tasks in plain language instead of manually navigating long, multi-step USSD menus. The initial market remains Nigeria; Ghanaian Cedi (GHS) is a reference-video example, not a change to FinXchange's Nigeria-first scope.

### Non-technical requirements
- **Vision:** Reduce friction and errors when accessing supported mobile-money tasks through natural-language interaction.
- **Audience:** Users of authorized mobile-money and financial services, especially those who find multi-step USSD flows difficult or have unreliable mobile data.
- **Chat commands:** Support the typed intents `check_balance`, `send_money`, and `cash_out` only where the selected provider/network supports the operation.
- **Entity extraction:** Resolve action, amount, currency, recipient/contact or phone number, and service. Ask a clarifying question if any financial detail is ambiguous.
- **Quick commands:** Offer one-tap actions such as Check Balance, Send Money, and Cash Out, with unsupported actions hidden or clearly disabled.
- **Intent confirmation:** Before a financial action, show the resolved recipient/destination, amount, currency, fee, total debit and Confirm/Cancel controls.
- **Feedback:** Show plain-language progress, authoritative status, fees, reference and remaining balance only when returned by a trusted source.
- **Navigation:** Consider Home, Chat, Services, Favourites and Settings as the primary navigation, with Activity/transaction history reachable without relying on chat.
- **Distribution:** Google Play is the intended Android distribution channel, subject to policy review, permission declarations and approval. Do not promise acceptance before review.

## Technical requirements and constraints

### Android-first platform
- Target Android first. Do not promise iOS parity for an accessibility-driven USSD prototype; assess iOS separately if required.
- React Native/Expo may remain the app framework, but a native Android module/service may be required for any approved AccessibilityService proof of concept. Verify Expo development-build and native-module compatibility before choosing implementation details.
- Keep conventional form-based flows available if chat, speech recognition, or automation is unavailable.

### USSD automation feasibility gate

The reference concept proposes Android `AccessibilityService` to observe and interact with USSD dialogs. Android exposes APIs for accessibility events, active-window content and user-interface gestures, but this does **not** guarantee reliable automation of every carrier's USSD dialog or every Android device.

Google Play policy is a material release risk. The official policy says Accessibility API automation must have a narrow, clearly understood purpose; autonomous initiation, planning and execution of actions or decisions is prohibited for general apps. Accessibility tools are specifically for apps whose core purpose assists people with disabilities. FinXchange must not mislabel itself as an accessibility tool merely to obtain the exemption. A disclosure, affirmative consent and Play Console declaration may be required, and approval is not guaranteed.

Therefore:
- Treat AccessibilityService-based USSD interaction as a research spike, not a committed production architecture.
- Before coding, document exactly which USSD screen events are read, which actions are automated, what data is accessed, and why narrower APIs cannot achieve the same purpose.
- Obtain policy review and test with target Android versions, OEMs, SIM/network providers and real USSD flows.
- Do not bypass device security controls, mislead users, or run financial actions autonomously.
- If the design cannot meet Google Play policy, fall back to a user-visible native USSD handoff or an authorized provider API/SDK. Provider APIs remain preferable where available.
- Accessibility permission onboarding must explain the exact purpose, data accessed, operation boundaries and how to disable the service. The user must explicitly enable it in Android Settings; never pressure or trick users into granting it.

### Natural-language processing
- Convert free-form requests into a typed intent object: `action`, `amount_minor` or normalized decimal string, `currency`, `recipient_reference`, `service_provider`, and unresolved fields.
- Normalize Nigerian Naira (NGN) for the initial market; do not silently convert currencies.
- Contact names can be ambiguous or duplicated. Ask the user to select a specific beneficiary and confirm a masked destination before money movement.
- NLU may prepare or explain an action but cannot itself authorize or execute a transfer.
- Reject unsupported commands safely and provide ordinary form-based navigation.

### PIN and authentication handling

The sample description proposes an in-app numeric PIN prompt for USSD authorization. This must **not** be adopted as a default requirement without written provider authorization and a reviewed security design. An app that collects and forwards a mobile-money PIN creates severe credential theft, logging, interception and policy risks.

Preferred requirements:
- Use a provider-owned secure PIN/authentication screen, provider SDK, or documented authorization mechanism whenever available.
- Do not ask users to type mobile-money PINs into the conversational message field, voice input, analytics, crash reports or ordinary app logs.
- Do not store raw PINs, OTPs or reusable provider credentials. Do not retain PINs after a session.
- If a provider explicitly requires a PIN entry flow that FinXchange hosts, block implementation until a documented threat model, provider contract, secure keypad/transport design, independent security review and legal/compliance approval are completed. Do not claim an in-app keypad is inherently secure.
- Explain which provider is receiving authentication and why, without misleading users about how credentials are used.

### Transaction state and safety
- Chat intent → clarification → structured review → explicit confirmation → authorized authentication → submit → provider status/reconciliation.
- A balance request must display its source and timestamp when cached.
- A transaction timeout or interrupted USSD session is `pending`/`needs_review`, not a confirmed failure or success.
- Never automatically resubmit a payment whose outcome is uncertain; query provider status first.
- Confirmation must be explicit and specific to the transaction; changing recipient, amount, fee or destination invalidates the previous confirmation.

## Release gates
1. Policy/legal review of the proposed AccessibilityService use and Google Play declaration requirements.
2. Written evidence that the chosen provider/network permits the intended USSD interaction and authentication model.
3. Android proof of concept on target devices and operators, including interrupted dialogs and unexpected prompts.
4. Security threat model and independent review of credential handling.
5. Test suite for ambiguous names, malformed amounts, wrong currency, duplicate commands, accidental confirmation, timeout, duplicate callbacks and provider failure.
6. Google Play listing and in-app disclosures accurately describe permissions and data access.
7. If any gate fails, ship chat plus quick actions and a user-controlled USSD handoff/provider integration instead of accessibility-driven automation.

## Official references
- Android AccessibilityService API: https://developer.android.com/reference/android/accessibilityservice/AccessibilityService
- Google Play — Use of the AccessibilityService API: https://support.google.com/googleplay/android-developer/answer/10964491?hl=en
- Google Play policy summary for sensitive permissions/APIs: https://support.google.com/googleplay/android-developer/answer/16558241?hl=en

These sources should be rechecked during implementation and Play Console submission because policies and platform behavior can change.