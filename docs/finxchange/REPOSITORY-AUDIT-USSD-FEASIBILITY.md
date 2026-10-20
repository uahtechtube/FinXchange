# FinXchange Repository Audit and USSD Feasibility Plan

**Audit date:** 2026-10-20  
**Repository:** `uahtechtube/FinXchange`  
**Branch reviewed:** `main`  
**Scope:** Read-only source/configuration review; no application code was changed as part of this audit.

## Executive summary

The current repository is a **React + TypeScript web application built with Vite**, not a React Native/Expo Android project. It already contains UI pages for a dashboard, services, transfers, chat assistance, and voice payment, so the project has a useful prototype base for the Vusa AI conversational experience. However, the checked-in evidence does not establish a production backend, a native Android AccessibilityService, a tested USSD automation engine, or a live authorized mobile-money integration.

**Recommendation:** Keep the current web app as the UI/product prototype. Build a separate Android proof of concept for USSD feasibility only after policy/security review. Do not retrofit native USSD assumptions into the web app or present demo/local state as real financial execution.

## 1. Repository findings

| Area | Evidence in repository | Assessment |
|---|---|---|
| Framework | `package.json`: React 18, TypeScript, React Router, Vite | Web SPA; not React Native/Expo |
| Build | Scripts are `vite`, `vite build`, `vite preview` | Build/test commands need to be run in a local environment to confirm health |
| Routing | `App.tsx` uses `HashRouter` and web route components | Browser-style navigation |
| Chat | `pages/Chatbot.tsx` calls Google Gemini from client code using `process.env.API_KEY` | FAQ/chat prototype; it is not a typed financial command engine |
| Voice | `pages/VoicePayment.tsx` uses Gemini Live/browser audio APIs and includes a confirmation modal | Existing voice UX prototype; browser APIs do not implement Android USSD automation |
| Financial UI | Dashboard, services, transactions, transfer/withdrawal and other pages are present | Broad demo UI; presence of a page is not evidence of a live service |
| Provider integration | README references Paystack and `VoicePayment.tsx` imports `services/bankApi` | Must inspect each service, server boundary, environment handling, and provider agreement before relying on it |
| Runtime state | README describes localStorage for user state, transactions, balance and settings | Local state is not a financial ledger and cannot prove settlement |
| Backend | No backend service or server directory is established by the inspected root tree/configuration | Backend existence and production readiness are unverified; plan for a trusted server-side transaction layer |
| Android native project | Root tree has no visible `android/`, Gradle project, Android manifest or AccessibilityService implementation | No evidence of a native Android USSD prototype in this repository |
| Tests/CI | No test script appears in the inspected `package.json` | Add unit/integration tests and CI before financial workflows are considered release-ready |

## 2. Key risks to resolve

1. **Client-side secrets:** `pages/Chatbot.tsx` initializes Gemini with `process.env.API_KEY`, while `index.html` loads `/env.js`. If a real API key is placed in a browser-served file, users can extract it. Move secret-bearing calls behind a backend and rotate any key that has been exposed.
2. **Browser is not Android native:** A Vite SPA cannot directly provide an Android `AccessibilityService`. A native Android project/module and device-level tests are required for that approach.
3. **Local storage is not authoritative:** balances and transaction records held in the browser can be altered and do not prove a provider has settled a payment.
4. **Natural language must not execute money movement directly:** the current chat component sends text to Gemini and displays a response. Implement a strict intent schema, validation, clarification, explicit transaction review, and server-side authorization separately from free-form AI responses.
5. **Provider status and duplicate protection:** transaction initiation must use server-side idempotency, authoritative provider references, status polling/webhooks where available, and reconciliation. Timeouts must become `pending`/`needs_review`, not automatic retries.
6. **Dependency/configuration mismatch:** `package.json` defines a Vite build, but `README.md` describes a no-bundler/import-map setup and using `serve`. Standardize the developer instructions on the actual Vite project and validate a clean install/build.
7. **Potential source path issue:** `pages/VoicePayment.tsx` imports `../../services/bankApi`; verify that the referenced file exists at the expected path and that `npm run build` succeeds from a clean checkout.

## 3. Recommended architecture

### Stage A — keep the existing web prototype
- Use the existing React/TypeScript/Vite app for UI iteration and clickable demos.
- Refactor the chat assistant to return a typed intent proposal rather than generating free-form instructions that could be mistaken for transaction execution.
- Keep provider credentials and all transaction authorization on a trusted backend; never ship secret keys in `env.js` or browser bundles.
- Clearly label simulated balances, provider responses, and demo transactions.

### Stage B — Android USSD proof of concept
- Create a separate Android-native proof-of-concept project or an Expo development build with a custom native module if React Native/Expo is selected for the production mobile app. The current repo alone does not provide that native runtime.
- First document the exact AccessibilityService use case and conduct Google Play policy/security review. Do not assume policy acceptance.
- On test devices/SIMs, investigate whether USSD dialogs produce accessible events and whether interaction remains reliable across Android versions, OEMs, SIM selection, operator menus, timeouts, and permission changes.
- Never collect or log a user's mobile-money PIN in chat. Prefer provider-controlled authentication or documented provider APIs.
- Compare accessibility-based automation with safer alternatives: user-controlled USSD handoff, authorized provider API/SDK, or a contracted aggregator.
- Gate any real financial operation behind an approved provider integration and a server-side transaction state machine.

### Stage C — typed intent and transaction lifecycle

Suggested intent contract:
```ts
type FinancialIntent =
  | { action: 'check_balance' }
  | { action: 'send_money'; amountMinor: number; currency: 'NGN'; recipientId?: string; recipientPhone?: string }
  | { action: 'cash_out'; amountMinor: number; currency: 'NGN'; destinationId?: string };
```

Validate amount, currency, recipient and provider capability on a trusted backend. An AI model may propose an intent, but deterministic validation and user confirmation must decide whether it is eligible to proceed.

Suggested transaction states:
`draft → awaiting_confirmation → authorized → submitted → pending → succeeded | failed | reversed | needs_review`.

Use a unique idempotency key for each confirmed intent. If the provider times out after submission, check status/reconcile before any retry. Changing the recipient, amount, currency, fee or destination invalidates the earlier confirmation.

## 4. USSD feasibility test plan

### Test questions
1. Does the chosen carrier's USSD session appear in accessibility events on the target Android devices?
2. Is the dialog content readable and interactable without fragile coordinate taps?
3. Does the method behave consistently across at least two target Android versions, multiple device/OEM variants and the intended SIM/operator combinations?
4. What happens on incoming calls, app backgrounding, SIM-selection prompts, language changes, unexpected menus, timeouts, network loss and user cancellation?
5. Can the service avoid capturing unrelated screen content or sensitive credentials?
6. Is the use case permissible under current Google Play policy and accepted in the required Play Console declaration?
7. Does the provider/operator permit this interaction and the proposed authentication/transaction model?

### Pass/fail gate
- **Pass to further prototype:** documented permission scope, policy/security review completed, reliable navigation of a limited non-financial test menu, no credential logging, explicit user control and reproducible tests.
- **Do not proceed with AccessibilityService:** policy or provider permission is unclear/denied, unrelated screen data would be accessed, PIN handling cannot be made acceptable, or reliability is too low.
- **Fallback:** user-controlled USSD handoff or an authorized provider API/SDK/aggregator. A fallback must not claim automated completion if the user or provider still has to finish the operation.

## 5. Immediate implementation backlog

| Priority | Task | Deliverable |
|---|---|---|
| P0 | Run clean install and `npm run build`; resolve any compile/path/configuration errors | Build log and reproducible setup instructions |
| P0 | Inspect all provider and API helper files and remove browser-exposed secrets | Integration inventory and secret-handling remediation |
| P0 | Mark mock/local financial data as demo-only | Clear prototype labels and no misleading success receipts |
| P1 | Define intent schema and deterministic parser tests for balance, send money and cash-out | Unit-tested parser; no payment execution |
| P1 | Build structured review/confirm/cancel component matching Vusa AI interaction | UI prototype with recipient, amount, fee, total and states |
| P1 | Write AccessibilityService policy assessment and provider capability matrix | Signed-off feasibility decision |
| P2 | Create native Android proof of concept only if gates pass | Device/operator test report |
| P2 | Specify backend transaction service, idempotency and reconciliation | API contract and integration tests |

## 6. Official references to re-check before implementation

- Android `AccessibilityService`: https://developer.android.com/reference/android/accessibilityservice/AccessibilityService
- Google Play AccessibilityService policy: https://support.google.com/googleplay/android-developer/answer/10964491?hl=en
- Expo development builds: https://docs.expo.dev/develop/development-builds/introduction/

## Audit limitations

This review inspected the GitHub tree and selected files (`package.json`, `App.tsx`, `pages/Chatbot.tsx`, `pages/VoicePayment.tsx`, `index.html`, `index.tsx` and `README.md`). It did not run `npm install`, execute the build, inspect every provider helper, test a physical Android device/SIM, verify any provider account, or submit a Google Play policy declaration. Findings described as unverified require those follow-up checks.