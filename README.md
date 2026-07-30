# Compliance Joe (Inbound Pre-Qual + Compliance Voice Agent)

Inbound voice-agent prompt on Ultravox (model `ultravox-v0.6-llama3.3-70b`). It screens inbound callers for fraud and compliance, verifies their location, pre-qualifies legitimate callers, and hands off to a licensed agent.

> [!IMPORTANT]
> **This is a TEST-LINE CANDIDATE, not campaign production.** It runs only on the test line below, not on any live Calls-to-Convert campaign. Do not promote or merge it into the production call flow without a human reviewing the final prompt diff.

## Live test

- **Call:** 970-409-1156
- **Ultravox agent:** https://app.ultravox.ai/agents/a5e4eedc-be3f-4104-82cd-30ec8bc7ce2e
- **Prompt:** [`compliance_joe_gepa_r1_20260730.md`](compliance_joe_gepa_r1_20260730.md)
- **Prompt SHA-256 (LF):** `d17acdb000c18fd03f55f1a85b6269d4805118ad9f03dcbe1feaf57fb5da6656`
- **Previous:** [`compliance_joe_v7.md`](compliance_joe_v7.md), SHA-256 (LF) `dc9b162ddfbe9f56bd14bb1def41ecac33c3cf7678477192c5d905a050c6543d`

---

## What changed, and how it was produced

This prompt addresses the feedback Jamie Sutton and Carlos Medina gave on 2026-07-29. It was **written by GEPA**, not by hand: the text is the winner of an offline reflective-optimization run, and no line of it was hand-edited afterwards.

| | |
| :--- | :--- |
| Optimizer | GEPA via `staging_and_telephony/voice-prompt-fix`, `src/gepa_optimize.py` |
| Run | `offline_r1`, 2026-07-30T11:55:47Z, 125 rollouts, 8 candidates |
| Source artifact | `prompts_out/offline_r1_gepa_20260730T115547Z_best.md` |
| Rollouts | Sonnet-on-Bedrock text role-play against the behavioral predicates in `src/scorer.py` |
| Offline score | **9 of 12 personas fully clean, mean 0.971**, against 0 of 12 and mean 0.875 for the prior prompt |

A second round was run and produced no improvement (9 of 12, mean 0.967), which is the stopping condition: the offline predicates stopped discriminating.

---

## Live evidence (2026-07-30)

**16 real PSTN calls** into 970-409-1156 from `+1 725-444-9511` (which Trestle resolves to Nevada, so a caller claiming another state mismatches for real). Every call was verified against the agent's own Ultravox call record before being scored: `agentId`, the full `systemPrompt` hashed against the file above, `model`, `vadSettings` and `inactivityMessages`. **16 of 16 verified.** The harness aborts on the first mismatch rather than scoring a call whose config drifted.

### Fixed and confirmed live

| Item | Result |
| :--- | :--- |
| The abrupt `"We're unable to move forward"` exit line | 🟢 Gone, 0 of 16 |
| State asked only once | 🟢 Exactly 1 ask on every call that reached the state step |
| Internal narration of the state check | 🟢 0 of 16 |
| Tool call spoken aloud (Carlos's report) | 🟢 0 of 16 |
| Disclosure exact wording survives interruption | 🟢 12 of 12, including 3 of 3 under deliberate talk-over |

### Still open

| Item | Result | Owner |
| :--- | :--- | :--- |
| Warm goodbye actually spoken on a mismatch | 🟡 **3 of 5.** Better than v7's roughly 2 of 9, still not reliable | Model or backend, see limitations |
| Concierge opening heard | 🟡 **6 of 12.** GEPA front-loads the disclosure, so the concierge phrase is what an interrupting caller clips. 0 of 3 under deliberate talk-over | Ordering or telephony |
| Recording disclosure heard twice | 🔴 **Confirmed live.** Two causes: the `<Say>` in the `/inbound-call` TwiML, and a prompt-side repeat where Joe re-states it mid-call | Charlotte (TwiML), plus a further prompt round |
| Caller answers the state question mid-question | 🔴 Not reproduced by the rig | Jamie, see limitations |

> [!NOTE]
> **Root cause of Jamie's double state ask, from her own call `2d1020dc`:** Joe asks the state, receives the answer, then fires `getCallerState`, and when the tool result returns he re-enters the ask-the-state step and asks a second time. It is not barge-in truncation, which was the earlier assumption. The offline simulator does not reproduce it because it supplies the tool result in the system prompt rather than as a mid-conversation tool result, so the optimizer has no gradient on it yet.

---

## State verification (the double check)

Two independent signals must agree before a caller can proceed:

1. The state tied to the caller's **phone number**, via Trestle (`getCallerState`).
2. The state the **caller says** when asked.

If they do not match, the call is dropped. A matching state proceeds into pre-qualification and transfer.

## What it does

- Opens with the exact recorded-line disclosure ("This call may be recorded for quality and training purposes.").
- Leads with what the service is, not only a name ("a concierge service for insurance coverage"), per Jamie's request.
- The two-way state check above.
- Drops incentivized, transferred ("you called me"), survey and off-topic callers at the reason-for-calling step.
- Never-infer: re-asks instead of assuming an unstated pre-qual answer.
- On a mismatch, exits warmly without stating a reason, instead of the previous abrupt line.
- Does not speak its internal verification logic aloud.

---

## Known limitations

> [!WARNING]
> - **The spoken goodbye is still not reliable.** Measured at 3 of 5 on this prompt, and about 2 of 9 on v7. `ultravox-v0.6-llama3.3-70b` frequently hangs up before the line completes, so treat the goodbye as advisory. If the graceful exit has to be guaranteed, it needs backend help rather than more prompt work.
> - **Background noise is untested.** The rig speaks clean synthesized audio down a clean SIP leg. It can produce an unintelligible caller, but not a clear caller with a television behind them, which is the specific case Jamie reported. Human re-testing is the acceptance test for that item.
> - **Caller ID cannot be varied.** All rig calls originate from one DID resolving to Nevada, so only the "caller claims a non-Nevada state" case is covered. A Nevada-matching caller, a number Trestle cannot resolve, and a ported number resolving to the wrong state are all still unverified on a real phone.
> - **Possible false drops from inaccurate or ported-number state data.** Trestle's registered state can differ from where the caller actually is (in testing a 305 / Miami number resolved to PA), so a legitimate caller can be dropped as a mismatch.
> - **Not gated.** This candidate has not been through the fail-closed promotion gate against a sealed holdout. It is a test-line candidate only.

## For Charlotte

Three items are on your side:

1. **The duplicate recording disclosure.** The first one is a `<Say>` in the `/inbound-call` TwiML, before Joe speaks. Removing it is a one-line backend change. Joe's own prompt-side repeat is a separate defect and is being handled here.
2. **Whether the silent mismatch drop is acceptable**, or whether the goodbye needs backend help given the reliability above.
3. **Whether to tolerate ported or inaccurate Trestle state data**, which can drop a legitimate caller.

Integration into the production call flow (the Geico-replica for L2C) remains yours and Raghu's side.
