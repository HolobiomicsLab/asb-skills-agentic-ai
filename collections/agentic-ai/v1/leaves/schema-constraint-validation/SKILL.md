---
name: schema-constraint-validation
description: Use when a generated candidate JSON (e.g. Stage 2 hourly weather or Stage
  3 hourly consumption) must be checked against its schemas/ JSON schema before acceptance,
  with failures routed to the council regeneration loop.
when_to_use_negative:
- The input is real measured data (TMY meteorological data for the capital coordinates
  or CER dataset load measurements) processed outside the LLM generation loop, so
  there is no LLM candidate to gate.
- The goal is statistical evaluation of generated profiles against the CER dataset
  (4232 households) or ablation studies, rather than structural/constraint conformance
  of a single candidate.
- No governed council loop exists (no schemas/ files, no CEO arbiter) — a plain one-shot
  validator cannot implement the REGENERATE_PARTIAL / REGENERATE_FULL semantics this
  skill assumes.
tools:
- name: WattCouncil
  role: 'Governed LLM council whose control loop hosts the validation: candidates
    are checked against schemas/ and only CEO-accepted JSON is emitted'
  repo: github.com/Singularity-AI-Lab/wattcouncil
- name: meta-llama/Llama-4-Maverick-17B
  role: Cultural Auditor flagging cultural/narrative inconsistencies in the candidate
    JSON
- name: anthropic/claude-4-sonnet
  role: Physical Auditor flagging numerical/physical constraint violations against
    the schema constraints
- name: Qwen/Qwen2.5-72B-Instruct
  role: CEO final arbiter emitting ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL plus
    GUIDANCE text
- name: meta-llama/Llama-3.3-70B-Instruct
  role: Editor applying targeted fixes when the CEO requests partial regeneration,
    verified by the Approver before full regeneration
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_007
  source_papers:
  - doi: 10.1145/3744256.3812595
    title: 'WattCouncil: Context-Aware Household Energy Scenario Generation With Governed
      LLMs'
  source_content_hash: sha256:1faaa913dd6c266dc339c38808cb06dabf35a90f9d0eb534f6a14524cc828030
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepInfra API
  - OpenRouter
  - Python
  - Qwen/Qwen2.5-72B-Instruct
  - WattCouncil
  - anthropic/claude-4-sonnet
  - conda
  - google/gemini-2.5-flash
  - meta-llama/Llama-3.3-70B-Instruct
  - meta-llama/Llama-4-Maverick-17B
  - pip
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# -schema-constraint-validation

## Summary

WattCouncil's schema-constraint validation checks each LLM-generated structured JSON (family composition, hourly weather, hourly consumption) against the stage-specific JSON schemas in schemas/ and routes failures through the governed council loop rather than dropping them. It guarantees that only schema-conforming candidates that survive auditor review and a CEO ACCEPT become stage2_hourly_* / stage3_consumption_* outputs.

## When to use

Use when a generated candidate JSON (e.g. Stage 2 hourly weather or Stage 3 hourly consumption) must be checked against its schemas/ JSON schema before acceptance, with failures routed to the council regeneration loop.

## When NOT to use

- The input is real measured data (TMY meteorological data for the capital coordinates or CER dataset load measurements) processed outside the LLM generation loop, so there is no LLM candidate to gate.
- The goal is statistical evaluation of generated profiles against the CER dataset (4232 households) or ablation studies, rather than structural/constraint conformance of a single candidate.
- No governed council loop exists (no schemas/ files, no CEO arbiter) — a plain one-shot validator cannot implement the REGENERATE_PARTIAL / REGENERATE_FULL semantics this skill assumes.

## Inputs

- Candidate structured JSON from the Primary Generator (Stage 2 seasonal hourly weather profile or Stage 3 hourly household consumption conditioned on Stage 1 family and Stage 2 weather)
- Stage-specific JSON schema files from the schemas/ directory (schemas/stage2_hourly_weather, schemas/stage3_consumption)
- Stage 1 family composition JSON and Stage 2 weather JSON (LLM-generated or TMY-derived) as conditioning inputs for Stage 3
- Accumulated rules in knowledge_base.txt injected into Generator system prompts

## Outputs

- CEO ACCEPT verdict on schema-valid candidate JSON
- Saved accepted profiles: stage2_hourly_* / stage3_consumption_* JSON files in the timestamped run outputs/raw directory, checkpointed to raw/
- Auditor flag reports (cultural/narrative inconsistencies and numerical/physical constraint violations)
- CEO decision (ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL) with GUIDANCE text, and persistent rules appended to knowledge_base.txt

## How to apply

Validate every candidate JSON produced by the Primary Generator (google/gemini-2.5-flash via DeepInfra) against the stage-specific schema loaded from the schemas/ directory — schemas/stage2_hourly_weather for Stage 2 weather and schemas/stage3_consumption for Stage 3 hourly consumption. Failed candidates are not discarded silently: the Cultural Auditor (meta-llama/Llama-4-Maverick-17B) flags cultural/narrative inconsistencies and the Physical Auditor (anthropic/claude-4-sonnet) flags numerical/physical constraint violations, after which the CEO (Qwen/Qwen2.5-72B-Instruct) emits exactly one enumerated decision — ACCEPT, REGENERATE_PARTIAL, or REGENERATE_FULL — plus GUIDANCE text. REGENERATE_PARTIAL routes the candidate to the Editor (meta-llama/Llama-3.3-70B-Instruct) for targeted fixes that the Approver must verify against the guidance, and an Approver REJECTED verdict restarts full generation from the Generator. On REGENERATE_FULL, the CEO's guidance is reinjected directly into the Generator's prompt, and critical or generalizable failures are extracted as persistent rules appended to knowledge_base.txt for injection into all future Generator system prompts. Save only CEO-accepted, schema-valid profiles as stage2_hourly_* / stage3_consumption_* in the timestamped run outputs/raw directory, checkpointing each item to raw/ for smart resume.

## Related tools

- **WattCouncil** (Governed LLM council whose control loop hosts the validation: candidates are checked against schemas/ and only CEO-accepted JSON is emitted) — github.com/Singularity-AI-Lab/wattcouncil
- **meta-llama/Llama-4-Maverick-17B** (Cultural Auditor flagging cultural/narrative inconsistencies in the candidate JSON)
- **anthropic/claude-4-sonnet** (Physical Auditor flagging numerical/physical constraint violations against the schema constraints)
- **Qwen/Qwen2.5-72B-Instruct** (CEO final arbiter emitting ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL plus GUIDANCE text)
- **meta-llama/Llama-3.3-70B-Instruct** (Editor applying targeted fixes when the CEO requests partial regeneration, verified by the Approver before full regeneration)

## Examples

```
python main.py --config config/runtime.yaml --weather-config config/weather.yaml
```

## Evaluation signals

- Candidate JSON passes schema validation against schemas/stage2_hourly_weather (Stage 2) or schemas/stage3_consumption (Stage 3).
- Each council cycle terminates only on a CEO ACCEPT, and every emitted decision is one of the three enumerated actions (ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL).
- An Approver REJECTED verdict restarts generation from the Primary Generator instead of emitting unapproved edited output.
- Only accepted profiles are saved as stage2_hourly_* / stage3_consumption_* in the timestamped run outputs/raw directory, with each item checkpointed to raw/.
- scripts/analyze_logs.py over the run directory shows per-role call/token/cost rows (e.g. Generator google/gemini-2.5-flash 5 calls, CEO Qwen/Qwen2.5-72B-Instruct 5 calls, $0.1506 total in the README example), confirming the loop ran and closed.

## Limitations

- Schema conformance alone does not guarantee plausibility — cultural/narrative and numerical/physical checks require the dedicated Auditor roles, and acceptance occurs only on CEO ACCEPT.
- Each failed round re-runs paid LLM calls (DeepInfra API with OpenRouter fallback), so heavy regeneration inflates token and cost totals reported by scripts/analyze_logs.py.
- Persistent knowledge_base.txt rules are injected into all future generations in that run and subsequent runs, so over-broad extracted rules can over-constrain later families or seasons.
- The Stage 2 TMY real-data path bypasses LLM generation, so the council validation loop is not exercised in that configuration.

## Evidence

- [abstract] Structured energy scenarios are generated, audited, and validated under explicit constraints: "operating in specialized roles to generate, audit, and validate structured energy scenarios under explicit cultural, temporal, and physical constraints"
- [methods] The CEO is the final arbiter with exactly three enumerated decisions: "Final arbiter: ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL"
- [methods] The Physical Auditor checks numerical/physical constraint violations of candidate JSON: "Flags numerical/physical constraint violations"
- [methods] The Cultural Auditor checks cultural/narrative inconsistencies of candidate JSON: "Flags cultural/narrative inconsistencies"
- [readme] An Approver REJECTED verdict restarts the loop at full regeneration: "If REJECTED -> Triggers full regeneration (start over)"
- [readme] Critical failures become persistent rules injected into all future generator prompts: "This rule is saved to `knowledge_base.txt` and is injected into the system prompt for *all future generations* in that run and subsequent runs."
- [readme] REGENERATE_FULL guidance is reinjected into the Generator prompt on the next attempt: "When the CEO rejects an output with `REGENERATE_FULL`, the specific *guidance* is injected back into the Generator's prompt for the next attempt."
