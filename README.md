# Compliance Joe (Inbound Pre-Qual + Compliance Voice Agent)

Inbound voice-agent prompt on Ultravox (model `ultravox-v0.6-llama3.3-70b`). It screens inbound callers for fraud and compliance, verifies their location, pre-qualifies legitimate callers, and hands off to a licensed agent.

> [!IMPORTANT]
> **v7 is a TEST-LINE CANDIDATE, not campaign production.** It runs only on the test line below, not on any live Calls-to-Convert campaign. Do not promote or merge it into the production call flow without a human reviewing the final prompt diff.

## Live test

- **Call:** 970-409-1156
- **Ultravox agent:** https://app.ultravox.ai/agents/a5e4eedc-be3f-4104-82cd-30ec8bc7ce2e
- **Prompt:** [`compliance_joe_v7.md`](compliance_joe_v7.md)
- **Prompt SHA-256 (LF):** `dc9b162ddfbe9f56bd14bb1def41ecac33c3cf7678477192c5d905a050c6543d`

---

## State verification (the double check)

Two independent signals must agree before a caller can proceed:

1. The state tied to the caller's **phone number**, via Trestle (`getCallerState`; `{{twilioState}}` as fallback if Trestle returns nothing).
2. The state the **caller says** when asked.

If they do not match, the call is dropped. A matching state proceeds into pre-qualification and transfer.

## What it does

- Opens with the exact recorded-line disclosure ("This call may be recorded for quality and training purposes.").
- The two-way state check above.
- Drops incentivized / transferred ("you called me") / survey / off-topic callers at the reason-for-calling step.
- Never-infer: re-asks instead of assuming an unstated pre-qual answer.
- Does not speak its internal verification logic aloud (see below).

## Narration status

**v7 does not speak its internal state-check logic aloud.** The state section was rewritten to the production pattern (condition, then an exact spoken line, then a tool action) with all internal-reasoning language removed (no "silently determine," "compare," internal variable names, "validate," or "on record").

An earlier candidate ("cand1") did **not** reliably fix this. On real calls it still leaked intermittently ("I'm going to silently determine your state," "your state matched," "that doesn't match"). v7 is the version that fixes it.

## Evidence (2026-07-29)

Verified on **49 real PSTN calls** on the production model, with **0 narration leaks**:

- **Bulk test:** 20 diverse matching + 10 mismatch + 5 uncertainty = 35 calls, 0 leaks.
- **Regression gate:** match / mismatch / uncertainty / fraud (incentivized, transferred, off-topic) / never-infer, 2 reps each = 14 calls, 0 leaks, all behaviors held.

## Detector methodology

A call is flagged as a leak if **either** detector fires:

1. **Deterministic regex blacklist** (fast, exact known phrasings).
2. **Sonnet LLM judge**, calibrated before use against every known leak (0 missed) and known-clean lines such as the state question and the generic goodbye (0 false positives), to catch novel phrasings the blacklist would miss.

The keyword-only detector used earlier is why a leak was once missed; the judge closes that gap and the two run together.

---

## Known limitations

> [!WARNING]
> - **Silent mismatch rejection is accepted.** On a mismatch, v7 drops the call (never proceeds, reveals nothing) but usually **without** a spoken goodbye. v0.6 will not say the goodbye reliably (about 2 of 9 attempts), so the spoken goodbye is treated as advisory.
> - **Real Trestle-error fallback is unverified.** On every test call `getCallerState` succeeded, so the `{{twilioState}}` error branch never ran on the real phone. Confirm with a number Trestle cannot resolve that it falls back and still proceeds on a match.
> - **Possible false drops from inaccurate or ported-number state data.** Trestle's registered state can differ from where the caller actually is (in testing a 305 / Miami number resolved to PA), so a legitimate caller can be dropped as a mismatch.

## For Charlotte

The three items above are the open ones for you: whether to tolerate ported/inaccurate state data, confirming the Trestle-error fallback on the real phone, and whether the silent mismatch drop is acceptable or the goodbye needs backend help. Integration into the production call flow (the Geico-replica for L2C) is your and Raghu's side.
