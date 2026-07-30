# Compliance Joe (Inbound Pre-Qual + Compliance Voice Agent)

Inbound voice-agent prompt on Ultravox (`ultravox-v0.6-llama3.3-70b`). Screens inbound callers for fraud and compliance, verifies their location against Trestle, pre-qualifies, and hands off to a licensed agent.

> [!IMPORTANT]
> **Test-line candidate, not campaign production.** Not gated against a sealed holdout. Do not merge into the production call flow without a human reviewing the prompt diff.

## Live test

- **Call:** 970-409-1156
- **Agent:** https://app.ultravox.ai/agents/a5e4eedc-be3f-4104-82cd-30ec8bc7ce2e
- **Prompt:** [`compliance_joe_v9.md`](compliance_joe_v9.md)
- **SHA-256 (LF):** `d17acdb000c18fd03f55f1a85b6269d4805118ad9f03dcbe1feaf57fb5da6656`

---

## Provenance

v9 follows v7 in this repo and v8 (the prompt this replaces on the test line). The internal suite also has v9 to v11 files from the earlier narration work; those are unrelated experiment artifacts. The authoritative identity of this file is the source path and SHA above.

Addresses the Jamie Sutton / Carlos Medina feedback of 2026-07-29. **The prompt text was written by GEPA and not hand-edited.**

| | |
| :--- | :--- |
| Run | `offline_r1`, 2026-07-30T11:55:47Z, 125 rollouts, 8 candidates |
| Source | `voice-prompt-fix/prompts_out/offline_r1_gepa_20260730T115547Z_best.md` |
| Offline | **9 of 12 personas clean, mean 0.971** vs 0 of 12 and 0.875 prior |

A second round gained nothing (9 of 12, 0.967), which is the stopping condition.

---

## Live evidence, 2026-07-30

16 PSTN calls into 970-409-1156 from `+1 725-444-9511` (Trestle resolves it to Nevada, so a caller claiming another state mismatches for real). Each call was verified against the agent's own Ultravox call record before scoring (`agentId`, `systemPrompt` hashed against the committed file, `model`, `vadSettings`, `inactivityMessages`). **16 of 16 verified**; the harness aborts on the first mismatch rather than scoring a drifted call.

**Fixed**

| Item | Result |
| :--- | :--- |
| Abrupt `"We're unable to move forward"` exit | 🟢 0 of 16 |
| State asked once | 🟢 1 ask on every call reaching that step |
| Internal narration of the state check | 🟢 0 of 16 |
| Tool call spoken aloud (Carlos) | 🟢 0 of 16 |
| Disclosure wording survives interruption | 🟢 12 of 12 |

**Open**

| Item | Result | Owner |
| :--- | :--- | :--- |
| Goodbye actually spoken on mismatch | 🟡 3 of 5 (v7 was ~2 of 9) | Backend, see limitations |
| Concierge opening heard | 🟡 6 of 12; 0 of 3 under talk-over | Ordering / telephony |
| Disclosure heard twice | 🔴 TwiML `<Say>` **plus** a prompt-side repeat | Charlotte, plus a further round |
| Caller answers mid-question | 🔴 Not reproduced by the rig | Jamie |

> [!NOTE]
> **Root cause of the double state ask,** from Jamie's own call `2d1020dc`: Joe asks the state, gets the answer, fires `getCallerState`, and on the tool result re-enters the ask-the-state step and asks again. Not barge-in truncation, which was the earlier assumption. The offline simulator supplies the tool result in the system prompt rather than mid-conversation, so the optimizer has no gradient on it yet.

---

## Known limitations

> [!WARNING]
> - **The goodbye is not reliable.** 3 of 5 here, ~2 of 9 on v7. The model often hangs up before the line completes. Guaranteeing it needs backend help, not more prompt work.
> - **Background noise untested.** The rig speaks clean audio down a clean SIP leg. It can produce an unintelligible caller, not a clear caller with a TV behind them, which is the case Jamie reported. Human re-testing is the acceptance test.
> - **Caller ID cannot be varied.** All rig calls originate from one Nevada-resolving DID, so only "caller claims a non-Nevada state" is covered. A matching caller, an unresolvable number, and a ported number resolving wrong are unverified on a real phone.
> - **False drops from ported or inaccurate Trestle data** remain possible (in testing a 305/Miami number resolved to PA).

## For Charlotte

1. **Duplicate recording disclosure.** The first is a `<Say>` in the `/inbound-call` TwiML, before Joe picks up. One-line removal. Joe's own repeat is separate and handled here.
2. **Is the silent mismatch drop acceptable**, or does the goodbye need backend help given the reliability above?
3. **Tolerate ported or inaccurate Trestle state data?** It can drop a legitimate caller.

Production integration (the Geico-replica for L2C) remains yours and Raghu's.
