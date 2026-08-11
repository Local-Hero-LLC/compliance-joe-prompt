# Inbound Voice-Agent Prompts

Source-of-truth repo for the inbound voice-agent prompt variants authored by Mark Cetola, plus the inventory that maps every variant the team talks about to what is actually running. Variants are named by the number of qualifying questions they ask. Legacy nicknames appear once, in the mapping column below, and nowhere else.

Every file in this repo was authored by Mark (all commits: hud63 / mlh-mjc). The 3Q Intake (Original) prompt was never touched by Mark: zero commits from any Mark identity across all three branches of its repo (~300 commits, all Charlotte Hauke / charhauke / Esosa Sosa).

## Inventory (verified against production 2026-08-11, CtC read replica + Ultravox call fingerprints)

| # | Name | Previously called | Qualifying questions | Author | Status as of 2026-08-11 | Prompt of record |
|---|------|-------------------|----------------------|--------|-------------------------|------------------|
| 1 | 3Q Intake (Original) | "Original Joe", "Prequal Joe" | Are you currently insured? / Who is your current provider? / Insured 6+ months, no gaps? | Charlotte | LIVE. The majority AI agent: CtC ivr 14, 933 calls/30d (87% of AI-agent inbound). Live campaigns last 7d: 1326 Found Performance - Auto, 1624 RevX - Auto Pre-qual. Prompt unchanged since 2026-07-14. | [`prequalification-voice-agent` `index.js` line 362](https://github.com/Local-Hero-LLC/prequalification-voice-agent/blob/prod_prequal_agent/index.js#L362) (hardcoded in the service) |
| 2 | 3Q Intake Rework | "improved / more conversational Prequal Joe" | Same 3 questions, plus a warm opener and an upfront recording disclosure | Mark | NEVER DEPLOYED. Candidate only (GEPA it2, 2026-07-24). The belief that it went live ~Aug 6 is wrong; ivr 14's prompt predates it and has not changed. | Local harness artifact `prompts_out/prequal_auto_gepa_it2_best.md` (not in this repo) |
| 3 | 4Q Full Battery | "Compliance Joe", "Prequal + Compliance Joe" | What state are you calling from? / Are you currently insured? / Who's your current provider? / Insured 6+ months, no gaps? | Mark | LIVE since 2026-08-06 on CtC ivr 16 (+1 737-637-9256), 7 campaigns including three Data Lot (1859, 833, 1867) and VoiceAIQ (1944, 1945). | [`4q_full_battery.md`](4q_full_battery.md) |
| 4 | 4Q Full Battery, Branded | "FQI Joe" | Same 4 questions; brand identity via `{{brandName}}` merge token (10 occurrences, nothing hardcoded) | Mark | STAGED, not live. 1 test call ever (2026-08-07). Charlotte's three `*_auto_compliance_fqi` agents (2026-08-10) carry 0 calls and have the defects listed under Cautions. | [`4q_full_battery_branded.md`](4q_full_battery_branded.md) |
| 5 | 2Q Transfer Handoff | "DL gate", "Data Lot handoff", "apologetic handoff" | Who are you currently insured with? (confirm) / Allstate-match accept? | Mark | ON HOLD per the 2026-08-08 decision (Data Lot data-passthrough gets explored first). Demo line only; never took a production call. | [`2q_transfer_handoff.md`](2q_transfer_handoff.md) |

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

## Where each prompt physically lives

- Variants 3, 4, 5: the runtime prompt lives on pre-created Ultravox agents; the services ([`compliance-voice-agent`](https://github.com/Local-Hero-LLC/compliance-voice-agent), [`auto-compliance-fqi`](https://github.com/Local-Hero-LLC/auto-compliance-fqi)) call the agent by ID with `systemPrompt` commented out. The files in THIS repo are therefore the prompt of record for those agents.
- Variant 1: the prompt is hardcoded inside Charlotte's Node service ([`prequalification-voice-agent`](https://github.com/Local-Hero-LLC/prequalification-voice-agent)); changing it means a commit + deploy there. This repo intentionally holds no copy.
- CtC's `ivrs` table has no prompt column. Its `welcome_message` field is stale documentation text (ivr 16 still shows the old 3Q greeting); the Ultravox call object is the only prompt ground truth.

## Deployment map

| Ultravox agent | Created | Carries | Where |
|----------------|---------|---------|-------|
| Compliance-Joe-Prod / -Stage / -Dev (`33b95124…` / `0ae8e70b…` / `e20e3520…`) | 2026-08-03 | `4q_full_battery.md` byte-for-byte | Production ivr 16 traffic since 2026-08-06 (CALLS-3410) |
| Compliance-Joe-V1 (`a5e4eedc…`) | 2026-07 | 4Q Full Battery lineage | Test line +1 970-409-1156 (Charlotte's webhook; full tool support) |
| Compliance-Joe-FQI (`300574ed…`) | 2026-07-30 | `4q_full_battery_branded.md` | Demo line via Mark's router; `getCallerState`/`sendQualification` return `call_not_found` by design |
| prod/stage/dev_auto_compliance_fqi (`637e640f…` etc.) | 2026-08-10 | Modified copy of the branded file (see Cautions) | Charlotte's FQI service, pre-launch, 0 calls |
| DL-Gate-BEST-r9-20260808 (`c96eb3bc…`) | 2026-08-09 | `2q_transfer_handoff.md` | Demo line +1 970-489-7023, routing tool stubbed |

## Provenance

All three prompts here were written by GEPA optimization, not hand-edited:

- `4q_full_battery.md`: run `offline_r1`, 125 rollouts, 8 candidates, 9/12 personas clean, mean 0.971. Live-verified byte-for-byte against production calls.
- `4q_full_battery_branded.md`: run `fqi_r1`, 120 rollouts, 3 candidates, 8/8 personas clean, mean 1.000. Live-verified on PSTN (brand spoken when asked, no literal `{{token}}` braces in speech, no carrier impersonation).
- `2q_transfer_handoff.md`: GEPA live round-9 winner (2026-08-08), seals `20260808T212155Z` / `20260808T222712Z`. Honest status: BLOCKED at the fail-closed gate. Safety pack 25/25, sealed holdout ~0.78 against a 0.80 acceptance bar; never promoted. Ships only as a contingency if the Data Lot escalation (Tom Summerfield to Clint) fails to get data passed through correctly.

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
