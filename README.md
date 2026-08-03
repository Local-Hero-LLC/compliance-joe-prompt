# Compliance Joe (Inbound Pre-Qual + Compliance Voice Agent)

Inbound voice-agent prompts on Ultravox (`ultravox-v0.6-llama3.3-70b`). Screens inbound callers for fraud and compliance, verifies their location against Trestle, pre-qualifies, and hands off to a licensed agent.

> [!IMPORTANT]
> Test-line candidates. Neither has been through the fail-closed promotion gate. Do not merge into the production call flow without a human reviewing the prompt diff.

## The two prompts

| | Unbranded | Find Quality Insurance |
| :--- | :--- | :--- |
| File | [`compliance_joe_v9.md`](compliance_joe_v9.md) | [`compliance_joe_v9_fqi.md`](compliance_joe_v9_fqi.md) |
| Call | 970-409-1156 | 970-489-7023 |
| Agent | `a5e4eedc-be3f-4104-82cd-30ec8bc7ce2e` | `300574ed-5c31-4f3c-8bd1-c121c2a8ccfa` |
| SHA-256 (LF) | `d17acdb000c18fd0…` | `f79e6f4f126df9d4…` |

Behaviour is identical. The only difference is identity, and it is a **`{{brandName}}` merge token**, not a hardcoded name: the FQI prompt contains the token 10 times and the company name 0 times, so one file serves any brand by supplying the token at call time. That avoids a near-identical prompt per brand, each needing its own testing.

## Where `compliance_joe_v9.md` is deployed

Charlotte created one Ultravox agent per L2C environment on 2026-08-03 for the CALLS-3410 delivery. **All three carry this repo's `compliance_joe_v9.md` byte for byte** (sha `d17acdb000c1`, 16,837 chars, verified 2026-08-03), so the file here is the prompt that ships.

| Agent | Ultravox id | Endpoint |
| :--- | :--- | :--- |
| `Compliance-Joe-Dev` | `e20e3520-cdbb-45c0-a6eb-1e9f14261fbe` | `dev.voice-inbound-compliance.callstoconvert.com` |
| `Compliance-Joe-Stage` | `0ae8e70b-6b9d-4893-bbec-80614f411f38` | `staging.voice-inbound-compliance.callstoconvert.com` |
| `Compliance-Joe-Prod` | `33b95124-1b94-4b97-9ce5-658861044328` | `voice-inbound-compliance.callstoconvert.com` |

They differ from `Compliance-Joe-V1` only in their environment-suffixed tools (`getCallerState_<env>`, `sendQualification_<env>`), which is how each posts qualification details to its own L2C environment.

> [!CAUTION]
> **V1, the agent on the 970-409-1156 test line, now has PRODUCTION routing attached.** Its routing tool changed from `sendQualification` (`febc8639…`) to `sendQualification_prod` (`df7b3c14…`) between 7/30 and 8/03, and that is the same tool id `Compliance-Joe-Prod` uses. A cooperative test caller that reaches the handoff on that line will fire production routing. The prompt is unchanged; the tool is not. Confirm this is deliberate before running transfer-reaching tests there.

## Provenance

**Both prompts were written by GEPA and not hand-edited.**

| | Unbranded | FQI |
| :--- | :--- | :--- |
| Run | `offline_r1`, 125 rollouts, 8 candidates | `fqi_r1`, 120 rollouts, 3 candidates |
| Offline score | 9 of 12 personas clean, mean 0.971 (prior prompt: 0 of 12, 0.875) | 8 of 8 clean, mean 1.000 |

Rollouts are a Sonnet-on-Bedrock text simulation scored against the predicates in `voice-prompt-fix/src/scorer.py`. Each winner has a provenance record in `voice-prompt-fix/runs_seal/`, and the write path refuses a prompt that has none.

## Live evidence (2026-07-30)

**16 PSTN calls** on the unbranded line, plus smoke tests on the FQI line. Every call verified against the agent's own Ultravox record before scoring: agent id, prompt hash, model, VAD, inactivity. 16 of 16 verified; the harness aborts on the first mismatch rather than scoring a drifted call.

**Fixed and confirmed:** the abrupt `"We're unable to move forward"` exit is gone (0 of 16); the state is asked once; no internal narration (0 of 16); no tool call spoken aloud (0 of 16). On the FQI line the brand is spoken when asked, with no `{{token}}` braces reaching speech and no carrier impersonation.

**Open:**

| Item | Status |
| :--- | :--- |
| Recorded-line disclosure completes | 🔴 **Joe starts it and is cut off** when a caller talks over him. 1 of 3 got it out in full. See below |
| Warm goodbye on a mismatch | 🟡 Improved and often reliable now, but the model can hang up before the line finishes |
| Real background noise | ⬜ Untestable from the rig; needs a human |
| Caller answers mid-question | ⬜ Not reproducible from the rig; needs a human |

> [!WARNING]
> **The disclosure is the one to watch.** The carrier-side `<Say>` that used to play before Joe answered has been removed, which correctly fixed callers hearing the notice twice. The side effect is that Joe's own line is now the only one, and on real calls it is frequently truncated by an interrupting caller. Score this off the **agent's own** transcript, never the dialer's: the dialer's transcript merges the telephony audio with Joe's, and its ASR reported the disclosure as absent on calls where Joe's own record shows it spoken.

## Known limitations

> [!CAUTION]
> - **Caller ID cannot be varied on the rig.** All automated calls originate from one number resolving to Nevada, so only "caller claims a different state" is covered. A matching caller, an unresolvable number, and a ported number resolving wrong are unverified on a real phone.
> - **Ported or inaccurate Trestle data can false-drop a legitimate caller** (in testing a 305 / Miami number resolved to PA).
> - **The FQI line cannot exercise the state check or the transfer.** `getCallerState` and `sendQualification` look a call up in the backend by call ID, and calls on that line are created by a different router, so both return `call_not_found`. Joe degrades gracefully. Those two behaviours are unchanged from the unbranded prompt and are covered on the other line.

## For Charlotte

1. **The disclosure truncation above** is the open compliance question now that the `<Say>` is gone.
2. **Whether the silent mismatch drop is acceptable**, or the goodbye needs backend help.
3. **Whether to tolerate ported or inaccurate Trestle state data**, which can drop a legitimate caller.
4. **To test state and transfer on the FQI line**, 970-489-7023 would need registering in the backend.

Production integration (the Geico-replica for L2C) remains yours and Raghu's.
