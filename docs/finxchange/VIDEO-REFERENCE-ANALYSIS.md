# Reference Video Analysis — Vusa AI Mobile Money Experience

**Reviewed:** 2026-10-20  
**Source:** User-provided 86-second vertical video (360 × 640).  
**Purpose:** Translate observable interaction patterns into FinXchange product and implementation requirements.

## What the video visibly demonstrates

The recording shows an app branded **VUSA / Vusa AI** with an AI-assisted chat interface connected to a mobile-money service. It is a reference for interaction design, not evidence that FinXchange has the same provider access or capabilities.

Observed screens and actions:
1. **Chat-first home for financial commands.** The app asks what the user wants to do and accepts plain-language commands.
2. **Quick Commands.** Visible shortcuts include Check balance, Send money, and Cash out.
3. **PIN gate.** A mobile-money PIN entry screen appears before the financial action proceeds. FinXchange must not copy or store the PIN; authentication should remain with the authorized provider's secure flow.
4. **Natural-language balance request.** The user types “check balance”; the chat shows that it is requesting the balance from the network. The footage does not show a clear successful balance result, so do not treat this as proof of a successful balance lookup.
5. **Natural-language transfer.** The user enters a command to send money to “Amina.” The assistant restates the amount and recipient and asks the user to confirm or cancel before proceeding.
6. **Voice interaction.** Speaker/audio icons and a microphone affordance are visible, suggesting spoken input and spoken responses.
7. **Result explanation.** The displayed response describes a completed mobile-money transfer, recipient/service, fee, and remaining balance. This is a demo result in the recording, not verification of a real transaction.
8. **Bottom navigation.** Visible tabs include Home, Chat, Services, Favourites, and Settings.
9. **Conversational fallback.** The message field supports typing, with a send button; voice is an additional input mode rather than the only interaction.

## Product implications for FinXchange

### Adopt as design inspiration
- A conversational **FinXchange Assistant** as an optional shortcut to core financial services.
- Plain-language intents mapped to explicit, typed actions (for example, check_balance, send_transfer, cash_out).
- Quick-action cards for common tasks, plus conventional navigation for discoverability and accessibility.
- A confirmation summary before any transfer: recipient, destination, amount, currency, fee, total debit, and cancel/confirm actions.
- Clear transaction progress and outcome messages that explain fees and references without exposing sensitive data.
- Optional text-to-speech, speech input, and multilingual prompts behind feature flags.
- An explicit secure authorization handoff when a provider requires a PIN or other credential.

### Do not copy without validation
- The video does not establish the provider, API, operator, regulatory permissions, actual offline behavior, fraud controls, or transaction authenticity.
- Do not assume a conversational UI can execute transfers through arbitrary banks or mobile-money networks.
- Do not promise that voice commands or the chat interface work without data. Speech recognition may require connectivity; local UI availability is not settlement.
- Do not request a provider PIN in ordinary chat or log it. Use an approved provider-owned secure authentication flow.
- Do not show “successful” until the authorized provider's result is verified and reconciliation policy is satisfied.
- Do not treat the displayed Ghanaian cedi example or the Vusa branding as a FinXchange product decision. FinXchange remains Nigeria-first, with NGN and Nigerian provider capabilities prioritized.

## Proposed FinXchange assistant flow

1. User opens Chat/Assistant or selects a Quick Action.
2. User types or speaks a request.
3. App parses the request into a typed intent and asks clarifying questions if recipient, amount, currency, or service is ambiguous.
4. For balance inquiries, fetch through an authorized provider and label cached data with its last-sync time.
5. For transfers, resolve and validate the beneficiary; fetch current fee/limits; show a review screen.
6. User explicitly confirms or cancels. Step-up authentication is handled securely through the approved flow.
7. Backend creates an idempotent transaction intent and dispatches to the authorized provider.
8. App shows **Pending / Checking status** if the response is ambiguous; it must not ask the user to pay again until status is checked.
9. Display a receipt only after authoritative confirmation; include reference, amount, fee, timestamp, and support path.
10. Provide a conventional non-voice route for every task.

## Additional acceptance criteria

- An ambiguous natural-language command must never trigger a financial transaction by itself.
- A financial command must be converted to a reviewable transaction summary before confirmation.
- “Cancel” before submission must stop the action; after submission it must explain that cancellation may no longer be possible and check provider status.
- Voice transcripts must not contain PINs, OTPs, full credentials, or unnecessary personal data in logs.
- Assistant output must distinguish a quote, a locally saved draft, a submitted request, a pending result, and a confirmed transaction.
- If voice or AI parsing is unavailable, users can complete the same task through forms and buttons.
- Test names, amounts, recipient disambiguation, speech errors, background noise, low connectivity, timeout, duplicate intent, provider failure, and interrupted authentication.

## Confidence and limitations

This analysis is based on visible frames in the supplied video. The phone screen is cracked and some text is difficult to read; no provider documentation or source code was included. The recording demonstrates a sample interaction, not production readiness or verified integration behavior.