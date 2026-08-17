# Inbound Voice-Agent Prompts

Source-of-truth repo for the inbound voice-agent prompt variants authored by Mark Cetola, plus the inventory that maps every variant the team talks about to what is actually running. Variants are named by the number of qualifying questions they ask. Legacy nicknames appear once, in the mapping column below, and nowhere else.

Every file in this repo was authored by Mark (all commits: hud63 / mlh-mjc). The 3Q Intake (Original) prompt was never touched by Mark: zero commits from any Mark identity across all three branches of its repo (~300 commits, all Charlotte Hauke / charhauke / Esosa Sosa).

## Inventory (verified against production 2026-08-11, CtC read replica + Ultravox call fingerprints)

| # | Name | Previously called | Qualifying questions | “How can I help you today?” | Author | Status | Prompt of record | Where it runs |
|---|------|-------------------|----------------------|----------------------------------|--------|--------|------------------|---------------|
| 1 | 3Q Intake (Original) | "Original Joe", "Prequal Joe" | Are you currently insured? / Who is your current provider? / Insured 6+ months, no gaps? | No. Opens: “Hi this is Joe. I just have a few questions…” | Charlotte | LIVE. Prompt unchanged since 2026-07-14. | [`prequalification-voice-agent` `index.js` line 362](https://github.com/Local-Hero-LLC/prequalification-voice-agent/blob/prod_prequal_agent/index.js#L362) (hardcoded in the service) | CtC ivr 14. Campaigns: Found Performance - Auto (1326), RevX - Auto Pre-qual (1624). |
| 2 | 3Q Intake Rework | "improved / more conversational Prequal Joe" | Same 3 questions, plus a warm opener and an upfront recording disclosure | **No. It goes directly into “Are you currently insured?”** | Mark | NEVER DEPLOYED. Candidate only (GEPA it2, 2026-07-24). | [`prequal_auto_gepa_it2_best.md`](https://github.com/mlh-mjc/general-reorg/blob/073df6d250f1aef9f32a81d8a807621ef8711c7d/current_state/staging_and_telephony/voice-prompt-fix/prompts_out/prequal_auto_gepa_it2_best.md) (validated winner archived in GitHub) | Nowhere. |
| 3 | 4Q Full Battery | "Compliance Joe", "Prequal + Compliance Joe" | What state are you calling from? / Are you currently insured? / Who's your current provider? / Insured 6+ months, no gaps? | **Yes. Exact wording: “How can I help you today?”** | Mark | LIVE since 2026-08-06. The provider-mapping correction is validated and awaiting Charlotte's backend test and deployment. | [`4q_full_battery.md`](4q_full_battery.md) | **Production prompt editor: [Compliance-Joe-Prod](https://app.ultravox.ai/agents/33b95124-1b94-4b97-9ce5-658861044328/edit) (`33b95124-1b94-4b97-9ce5-658861044328`), serving CtC ivr 16 on +1 737-637-9256. Campaign routing is shared and varies by call.** |
| 4 | 4Q Full Battery, Branded | "FQI Joe" | Same 4 questions; brand identity via `{{brandName}}` merge token (10 occurrences, nothing hardcoded) | **Yes. Exact wording: “How can I help you today?”** | Mark | UPDATED + TARGET-TESTED 2026-08-17. Charlotte's insured-only branch is merged with the provider-mapping correction. EVA confirmed an uninsured caller skips provider and coverage and reaches `sendQualification`; the same run separately flagged a missing recording disclosure when the caller spoke first, so backend/full-flow retest remains required. | [`4q_full_battery_branded.md`](4q_full_battery_branded.md) | Charlotte's prod/stage/dev `*_auto_compliance_fqi` agents. Live source: [`prod_auto_compliance_fqi`](https://app.ultravox.ai/agents/637e640f-645b-4081-abb7-6a4f90398189). |
| 5 | 2Q Transfer Handoff | "DL gate", "Data Lot handoff", "apologetic handoff" | Who are you currently insured with? (confirm) / Allstate-match accept? | No. Opens by acknowledging that the caller already answered questions. | Mark | ON HOLD per the 2026-08-08 decision. | [`2q_transfer_handoff.md`](2q_transfer_handoff.md) | Demo line +1 970-489-7023, transfer routing stubbed; never took a production call. |
| 6 | 5Q Full Battery | "Compliance Joe + 3 questions", "the MDP-314 prompt" | And what state do you currently live in? / Are you currently insured? / Who's your current provider? / Insured 6+ months, no gaps? / Compliance advisory then say your state again | **Yes. Exact wording: “How can I help you today?”** | Mark | TESTED. Final state wording approved by Hannah on 2026-08-17; awaiting deployment. Derived from variant 3 for MDP-314 / CALLS-3479. | [`5q_full_battery.md`](5q_full_battery.md) | Nowhere. Not deployed. |

## Provider-mapping release handoff

The 2026-08-17 correction is present in 4Q, branded 4Q, and re-derived 5Q. It carries Charlotte's full live illustrative off-list roster, uses the scripted response “Got it, thanks.” after every carrier answer, and prevents Joe from guessing or speaking an internal classification.

EVA tested fixed 4Q directly and passed 152/152 local tests, Plymouth Rock 3/3, and Progressive holdout 3/3. Cekura then validated fixed 4Q directly in scenario 322012, run 3682320, result 815705. Joe heard “Plymouth Rock,” replied exactly “Got it, thanks.”, and progressed to the transfer attempt. Fixed 5Q independently passed Cekura run 3682319, result 815704. Production Ultravox agents were not modified.

Charlotte's next step is to load the updated 4Q prompt into a backend-connected non-production agent, verify `getCallerState` and `sendQualification` end to end, then deploy it to the three 4Q environment agents. Production traffic reaches [Compliance-Joe-Prod](https://app.ultravox.ai/agents/33b95124-1b94-4b97-9ce5-658861044328/edit) through CtC ivr 16 on +1 737-637-9256. Campaign routing is shared and varies by call. A production Ultravox write still requires explicit approval.

## What each variant says (verbatim)

### 3Q Intake (Original)
Opening: "Hi this is Joe. I just have a few questions to ask before I connect you."
No recording disclosure, no state question, no fraud or transfer screening. Questions: "Are you currently insured?" then "Got it. Now who is your current provider?" then "Great… and have you been insured for at least 6 months with no breaks or gaps?" On completion: "Thanks for that. I'm going to get you over to an agent now." then `sendQualification`.

### 4Q Full Battery
Opening: "Hi there, thanks so much for calling — this is Joe with a service that helps connect you with licensed insurance agents. This call may be recorded for quality and training purposes. How can I help you today?"
Adds silent state verification via `getCallerState` (mismatch gets a warm no-availability close, never a stated failure), the recording disclosure, and fraud screening, then the same insured/provider/tenure questions. Transfers via `sendQualification` in-turn; never hangs up after a successful transfer.

The "How can I help you today?" opener is deliberate: this variant is built for raw inbound callers who dial in cold. It is not built for callers a rep already interviewed; that scenario belongs to the 2Q Transfer Handoff.

### 4Q Full Battery, Branded
Byte-identical behavior to 4Q Full Battery. Opening names the brand: "Hi there, thanks so much for calling — this is Joe with {{brandName}}, a service that helps connect you with licensed insurance agents…". The `{{brandName}}` value is public by design (share freely when asked); only instructions/tools stay confidential. One file serves any brand.

### 2Q Transfer Handoff
Opening: "Hi there, this is Joe. I know you just answered some questions with the other representative, so I'll be really quick. This call may be recorded for quality and training purposes. Bear with me for just two quick questions so I can make sure we get you to the right person."
Confirms the current carrier, offers the Allstate match, and transfers. On an Allstate decline it still transfers but silently sets `current_provider` to `ALLSTATE` as a routing-exclusion tag. Fraud/gift-card/survey callers get one warm send-off line then `hangUp`.

### 5Q Full Battery
Opening: identical to 4Q Full Battery. The state question is reworded from "Great, and what state are you calling from?" to "And what state do you currently live in?" This asks for residence, not location, which is the substantive change MDP-314 requested and which changes who the `getCallerState` check declines.
Adds a fifth ask before any transfer: a fixed compliance advisory ("if you were promised any money, or a gift card, or if someone called you first without you asking for it, that is illegal and we cannot help you") closing with "please say again the state where you currently live now". The advisory is delivered to every caller who reaches a transfer, is never paraphrased, and always precedes `sendQualification`.
Adds an answer-recording procedure: repeat each answer back using the caller's own words, re-ask if there is nothing to repeat, and after three asks of the same question close the call with the existing warm no-availability line rather than transferring. The re-confirmed state is collected but NOT compared against the earlier answer; comparison remains a separate anti-fraud concern.

## Where each prompt physically lives

- Variants 3, 4, 5, 6: the runtime prompt lives on pre-created Ultravox agents; the services ([`compliance-voice-agent`](https://github.com/Local-Hero-LLC/compliance-voice-agent), [`auto-compliance-fqi`](https://github.com/Local-Hero-LLC/auto-compliance-fqi)) call the agent by ID with `systemPrompt` commented out. The files in THIS repo are therefore the prompt of record for those agents.
- Variant 1: the prompt is hardcoded inside Charlotte's Node service ([`prequalification-voice-agent`](https://github.com/Local-Hero-LLC/prequalification-voice-agent)); changing it means a commit + deploy there. This repo intentionally holds no copy.
- CtC's `ivrs` table has no prompt column. Its `welcome_message` field is stale documentation text (ivr 16 still shows the old 3Q greeting); the Ultravox call object is the only prompt ground truth.

## Deployment map

| Ultravox agent | Created | Carries | Where |
|----------------|---------|---------|-------|
| Compliance-Joe-Prod / -Stage / -Dev (`33b95124…` / `0ae8e70b…` / `e20e3520…`) | 2026-08-03 | Original `4q_full_battery.md` before the provider-mapping correction | Production ivr 16 traffic since 2026-08-06 (CALLS-3410). The production agent was not changed by the 2026-08-17 fix work. |
| Compliance-Joe-V1 (`a5e4eedc…`) | 2026-07 | 4Q Full Battery lineage | Test line +1 970-409-1156 (Charlotte's webhook; full tool support) |
| Compliance-Joe-FQI (`300574ed…`) | 2026-07-30 | `4q_full_battery_branded.md` | Demo line via Mark's router; `getCallerState`/`sendQualification` return `call_not_found` by design |
| prod/stage/dev_auto_compliance_fqi (`637e640f…` etc.) | 2026-08-10 | Modified copy of the branded file (see Cautions) | Charlotte's FQI service, pre-launch, 0 calls |
| DL-Gate-BEST-r9-20260808 (`c96eb3bc…`) | 2026-08-09 | `2q_transfer_handoff.md` | Demo line +1 970-489-7023, routing tool stubbed |
| Provider-mapping fix test agents (`14fce552…`, `3b17eb9e…`, `6886ebc1…`) | 2026-08-16 to 2026-08-17 | Test copies of `4q_full_battery.md` and `5q_full_battery.md` | RETIRED. All disposable agents were deleted. +1 970-489-7023 was restored to sealed DL-gate agent `c96eb3bc…`. |

## Provenance

Variants 3, 4 and 5 were written by GEPA optimization, not hand-edited. Variant 6 was not: it is a scripted derivation of variant 3, and it is the only prompt here whose wording came from the business rather than from a search.


- `4q_full_battery.md`: run `offline_r1`, 125 rollouts, 8 candidates, 9/12 personas clean, mean 0.971. Live-verified byte-for-byte against production calls.
- `4q_full_battery_branded.md`: run `fqi_r1`, 120 rollouts, 3 candidates, 8/8 personas clean, mean 1.000. Live-verified on PSTN (brand spoken when asked, no literal `{{token}}` braces in speech, no carrier impersonation).
- `2q_transfer_handoff.md`: GEPA live round-9 winner (2026-08-08), seals `20260808T212155Z` / `20260808T222712Z`. Honest status: BLOCKED at the fail-closed gate. Safety pack 25/25, sealed holdout ~0.78 against a 0.80 acceptance bar; never promoted. Ships only as a contingency if the Data Lot escalation (Tom Summerfield to Clint) fails to get data passed through correctly.
- `5q_full_battery.md`: NOT a GEPA product. Derived from `4q_full_battery.md` by script (`voice_eval/runs/mdp314/derive_candidate.py`), so every change is a literal string replacement against the seed and the diff is reproducible. The spoken wording is the business's: the advisory is Hannah Clack's 2026-08-14 text on MDP-314, reproduced verbatim apart from "(short pause)" rendered as an ellipsis and "NOW" lowercased so TTS does not shout it. Hannah finalized the state question on CALLS-3479 on 2026-08-17 as "And what state do you currently live in?"
  Measured on 8 authored personas, 3 repeats a side, against `4q_full_battery.md` as the control: gate INCONCLUSIVE (every metric inside a noise floor measured on the same eval set, so no collateral damage detected); gift-card caller transferred 3/3 by the control and dropped 3/3 by this prompt; home-insurance, uninsured and misheard-question personas all at or above the control on the holdout split. Full run: `voice_eval/runs/mdp314/`.
- Provider-mapping correction, 2026-08-17: `4q_full_battery.md` was fixed first, the same correction was ported to `4q_full_battery_branded.md`, and `5q_full_battery.md` was re-derived. EVA tested the fixed 4Q directly: 152/152 tests passed, Plymouth Rock candidate 3/3, and Progressive holdout 3/3. The EVA metric gate was INCONCLUSIVE because all deltas were inside the measured noise floors, with no detected collateral regression. Cekura scenario 322012 then validated fixed 5Q in run 3682319/result 815704 and fixed 4Q directly in run 3682320/result 815705. In both transcripts, Joe answered Plymouth Rock with exactly "Got it, thanks.", did not repeat or guess a carrier, and progressed to transfer. No production prompt was patched. Branded 4Q retains its separate `{{brandName}}` behavior and still requires Charlotte's backend-connected test before deployment.
- FQI insured-branch correction, 2026-08-17: Charlotte deployed identical prompt SHA `038663d31873` to the prod, stage, and dev FQI agents so provider and continuous-coverage questions are skipped when the caller says they are uninsured. The repository file ports that explicit `If YES` / `If NO: Skip all of Step 2` structure while retaining the provider-mapping correction her deployed copies do not yet contain.
  EVA run `2026-08-17_21-55-59.092916_insurance-qual-inbound_en_ultravox-v0.6-llama3.3-70b`, record 7, succeeded 1/1 on its first attempt. After the caller said “No, I let it lapse,” Joe asked neither provider nor continuous coverage and immediately called `sendQualification`; conciseness and conversation progression were both 1.0. The generic faithfulness judge scored 0 because the caller spoke before Joe's opening and Joe never recovered the required recording disclosure. That separate disclosure defect means this is a targeted branch pass, not a clean full-call approval.

## Cautions

> [!CAUTION]
> The "test" line +1 970-409-1156 carries PRODUCTION routing (`sendQualification_prod`, tool `df7b3c14…`, same ID as Compliance-Joe-Prod) since ~2026-08-03. A successful transfer from that line is a real transfer.

> [!WARNING]
> Charlotte's three `*_auto_compliance_fqi` agents (2026-08-10) differ from `4q_full_battery_branded.md` in exactly two ways, both defects:
> 1. The persona line was edited to hardcode `{{brandName}} = 'Find Quality Insurance'` while the service's `templateContext` still passes no `brandName` value. Note the spelling conflict: the demo router used "Fine Quality Insurance". Confirm the real brand before launch.
> 2. All 26 em dashes in the prompt were corrupted to mojibake bytes (encoding mishap during copy-paste). Fix by re-pasting from the raw file in this repo.

> [!NOTE]
> `compliance_stage` ran 62 calls on a real DID (+1 252-590-4591) on 2026-08-04 with the production prompt. If any of those were live callers, they are labeled stage in call metadata.

## History note

Files were renamed on 2026-08-11: `compliance_joe_v9.md` is now `4q_full_battery.md`, `compliance_joe_v9_fqi.md` is now `4q_full_battery_branded.md`, `dl_gate_v1.md` is now `2q_transfer_handoff.md`. Old deep links to those paths are dead; git history is intact (`git log --follow`).

`5q_full_battery.md` was added on 2026-08-16 for MDP-314 / CALLS-3479. It does not replace `4q_full_battery.md`, which is still the production variant; the two are separate variants. The 4Q, branded 4Q, and 5Q files received the provider-mapping correction on 2026-08-17, with 5Q re-derived from the corrected 4Q source. The name follows the question count: the fifth ask is the compliance advisory and the state re-confirmation that MDP-314 added.
