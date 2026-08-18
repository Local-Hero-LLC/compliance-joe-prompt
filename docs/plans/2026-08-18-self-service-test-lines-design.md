# Self-Service Test Lines for the Joe Inventory

## Goal

Give every prompt variant in the repository inventory a permanent, independently dialable test method so non-technical stakeholders can test the correct Joe without asking someone to repoint a shared line.

## Design

- Create six Ultravox sandbox agents, one for each inventory row.
- Load each agent from its prompt of record. For the branded agent, supply `brandName = Find Quality Insurance` at the router.
- Keep every sandbox isolated from production routing. Qualification and transfer tools must be non-production/no-op tools.
- Buy six dedicated Twilio local voice numbers through the Twilio API.
- Route all six numbers through the existing `inbound-agent-router`, with one immutable number-to-agent mapping per variant.
- Do not change `+1 970-409-1156`, `+1 970-489-7023`, the DL Gate agent, Charlotte's service, or any production campaign route.
- Verify each route from the live Twilio configuration, deployed Lambda `ROUTES`, and an actual call fingerprint.

## Documentation

- Create the dated local inventory `VOICE_AGENT_INVENTORY_2026-08-18.md` with a `Test phone number` column.
- Update the repository README inventory with the same column and live phone numbers.
- Record sandbox agent IDs and explicitly label the numbers as non-production test lines.

## Cost and rollback

Six Twilio local numbers add approximately $7/month before usage. Rollback is recoverable: remove the six router mappings, release only the six newly purchased numbers, and retire only the six newly created sandbox agents.
