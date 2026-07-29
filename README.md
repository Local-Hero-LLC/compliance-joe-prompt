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

## Known limitation (for Charlotte)

> [!WARNING]
> On the v0.6 model I could not get the **graceful goodbye on a mismatch** to fire reliably. The mismatch drop itself is reliable (the caller never proceeds and no verification detail is revealed), but the agent usually hangs up **without** first speaking a closing line. Three prompt variants that explicitly forced "speak the goodbye, then hang up, never silent" produced the spoken goodbye only about 2 of 9 times. For now the spoken goodbye is treated as advisory; the silent drop is accepted because it rejects the caller and leaks nothing.
