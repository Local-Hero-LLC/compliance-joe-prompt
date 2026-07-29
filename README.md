# Compliance Joe (Inbound Pre-Qual + Compliance Voice Agent)

Inbound voice-agent prompt running on Ultravox (model `ultravox-v0.6-llama3.3-70b`). It screens inbound callers for fraud and compliance, verifies their location, pre-qualifies legitimate callers, and hands off to a licensed agent. This is a candidate for review, not a final production ship.

## Live test

- **Call:** 970-409-1156
- **Ultravox agent:** https://app.ultravox.ai/agents/a5e4eedc-be3f-4104-82cd-30ec8bc7ce2e
- **Prompt:** [`compliance_joe_v6.md`](compliance_joe_v6.md)

---

## State verification (the double check)

Two independent signals must agree before a caller can proceed:

1. The state tied to the caller's **phone number**, looked up via Trestle (`getCallerState` tool), with the Twilio-captured state as a fallback if Trestle returns nothing.
2. The state the **caller says out loud** when asked "what state are you calling from?"

If the two do not match, the call is **dropped** (spoof / fraud check). Verified on the live line: a matching state proceeds into pre-qualification and transfer; a mismatched state (the number resolves to one state, the caller says another) hangs up.

---

## What else it does

- Opens with the exact recorded-line disclosure: "This call may be recorded for quality and training purposes."
- Drops incentivized / transferred ("you called me") / survey / off-topic callers at the reason-for-calling step.
- Never infers an unstated answer during pre-qualification (it re-asks instead of assuming).
- Does **not** speak its internal verification logic aloud. The earlier version narrated things like "matches the state code of PA from the getCallerState tool"; that leak is fixed and confirmed across many real calls.

---

## For Charlotte: notes and open items

**Confirmed working on the live line** (production `v0.6` model, real inbound calls): Trestle returns a two-letter code and the caller says the full state name, and v0.6 correctly treats them as the same and proceeds (e.g., the number resolved to `PA`, the caller said "Pennsylvania", it matched, ran pre-qual, and transferred). A mismatched spoken state is dropped, and no internal reasoning is spoken aloud.

**Open items worth your eyes:**

1. **Number portability / false drops.** A caller's Trestle-registered state can differ from where they actually are. In testing, a 305 (Miami) number resolved to `PA` on Trestle, so a caller physically in Florida saying "Florida" gets dropped as a mismatch. Worth deciding how much tolerance the state check should allow, or whether to lean on the `{{twilioState}}` value instead.
2. **Trestle-error fallback is unverified on the real phone.** On every live test call `getCallerState` succeeded, so the `{{twilioState}}` error branch never ran. Please confirm with a number Trestle cannot resolve that the agent falls back to `{{twilioState}}` and still proceeds on a match, rather than dropping a legit caller.
3. **Graceful goodbye on a mismatch is not reliable on v0.6.** It drops the caller correctly (never proceeds, leaks nothing) but usually hangs up without speaking a closing line first. Three variants forcing "speak the goodbye, then hang up" produced it only about 2 of 9 times, so it is treated as advisory for now.
