# Outbound Marco prompts

The live outbound voice-agent prompts, copied verbatim from production (CtC `ai_agents.agent_prompt`) on 2026-09-02, with the calls and transfers each one generated.

Result (Aug 1 – Sep 1, campaigns RT 1693 + CoReg 1764, by each agent's live prompt):

| Prompt | Fleet / states | Calls (attempts) | Dialed | Transfers |
|---|---|---|---|---|
| [`73734f` — CoReg gift-card "Identity"](coreg_firstdial_giftcard_ga-il-oh-pa.md) | CoReg first-dial GA/IL/OH/PA | 621 | 563 | 7 |
| [`5aae82` — "Persona & Tone"](firstdial_persona_shared.md) | CoReg first-dial, 14 other states | 699 | 487 | 5 |
| [`923b51` — CoReg gift-card redial](coreg_redial_giftcard_ga-il-oh-pa.md) | CoReg redial GA/IL/OH/PA | 348 | 5 | 0 |
| [`443968`](rt_firstdial_warm_il-pa-tn-va.md) / [`d0717d`](rt_redial_warm_il-pa-tn-va.md) — RT warm "Identity" | RT first-dial + redial IL/PA/TN/VA | 0 | 0 | 0 |

Scope: 2026-08-01 → 2026-09-01, campaigns 1693 (Real Time) + 1764 (CoReg), all statuses; each call attributed to the current prompt on its agent (all prompts predate August, so attribution is exact). RT shows 0 because the RT campaign did not dial in this window.
