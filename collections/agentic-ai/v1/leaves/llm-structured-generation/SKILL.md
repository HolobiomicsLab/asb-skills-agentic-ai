---
name: llm-structured-generation
description: Use when a scientific pipeline needs LLM-produced structured JSON outputs
  (e.g., hourly consumption profiles conditioned on a family JSON and weather JSON)
  that must pass schema validation and multi-role constraint auditing before acceptance.
when_to_use_negative:
- When the target output is raw measured data (e.g., CER-style smart-meter load measurements)
  rather than schema-valid synthetic profiles — use measurement datasets directly
  instead
- When a single unconstrained LLM call suffices and no cultural/temporal/physical
  constraint auditing or schema enforcement is required
- When inputs are already validated consumption profiles (e.g., post-CEO ACCEPT outputs)
  — downstream analysis or CSV conversion is the appropriate next step, not regeneration
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3071
tools:
- name: WattCouncil
  role: Framework orchestrating the role-specialized LLM council for governed structured
    generation
  repo: github.com/Singularity-AI-Lab/wattcouncil
- name: google/gemini-2.5-flash
  role: Primary Generator producing candidate structured JSON
- name: anthropic/claude-4-sonnet
  role: Physical Auditor flagging numerical/physical constraint violations
- name: meta-llama/Llama-4-Maverick-17B
  role: Cultural Auditor flagging cultural/narrative inconsistencies
- name: Qwen/Qwen2.5-72B-Instruct
  role: CEO final arbiter issuing ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL
- name: meta-llama/Llama-3.3-70B-Instruct
  role: Editor making targeted fixes on partial regeneration
- name: DeepInfra API
  role: Primary LLM inference provider for council calls
- name: OpenRouter
  role: Optional fallback provider for LLM inference
- name: TMY (Typical Meteorological Year) weather data
  role: Real-data alternative weather input conditioning Stage 3 consumption generation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1145/3744256.3812595
    title: 'WattCouncil: Context-Aware Household Energy Scenario Generation With Governed
      LLMs'
  source_content_hash: sha256:f7333300c7a5019c5aa9e4eb3fe6706de105297cf2d10496a67359582a52844d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepInfra API
  - OpenRouter
  - Python
  - Qwen/Qwen2.5-72B-Instruct
  - TMY (Typical Meteorological Year) weather data
  - WattCouncil
  - anthropic/claude-4-sonnet
  - conda
  - google/gemini-2.5-flash
  - meta-llama/Llama-3.3-70B-Instruct
  - meta-llama/Llama-4-Maverick-17B
  - pip
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3071
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# llm-structured--generation

## Summary

Generate schema-constrained structured JSON (hourly household energy consumption profiles) with a council of role-specialized LLMs — Generator, Cultural Auditor, Physical Auditor, Editor, Approver, and a CEO arbiter — so that synthetic outputs satisfy cultural, temporal, and physical constraints.

## When to use

Use when a scientific pipeline needs LLM-produced structured JSON outputs (e.g., hourly consumption profiles conditioned on a family JSON and weather JSON) that must pass schema validation and multi-role constraint auditing before acceptance.

## When NOT to use

- When the target output is raw measured data (e.g., CER-style smart-meter load measurements) rather than schema-valid synthetic profiles — use measurement datasets directly instead
- When a single unconstrained LLM call suffices and no cultural/temporal/physical constraint auditing or schema enforcement is required
- When inputs are already validated consumption profiles (e.g., post-CEO ACCEPT outputs) — downstream analysis or CSV conversion is the appropriate next step, not regeneration

## Inputs

- Stage 1 family JSON (household composition and work regime)
- Stage 2 hourly weather JSON (LLM-generated or TMY-derived)
- JSON schema schemas/stage3_consumption.*
- Prompts from prompts/stage3_consumption/
- knowledge_base.txt persistent rules (if present)
- config/runtime.yaml pipeline settings and config/models.yaml role-model assignments

## Outputs

- Candidate and accepted structured JSON of hourly household energy consumption (stage3_consumption_*. in outputs/raw)
- CEO decision (ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL) with guidance
- Auditor flag reports and API/prompt logs via utils/api_logger.py
- Optionally CSV conversion of accepted JSON via scripts/convert_outputs.py

## How to apply

Compose the Generator prompt from stage-specific prompt files (e.g., prompts/stage3_consumption/) and inject any accumulated knowledge_base.txt rules, then run the Primary Generator (google/gemini-2.5-flash via DeepInfra, with OpenRouter as optional fallback) against the Stage 1 family JSON and Stage 2 hourly weather JSON to produce a candidate structured JSON. Route the candidate through the council: the Cultural Auditor (meta-llama/Llama-4-Maverick-17B) flags cultural/narrative inconsistencies, the Physical Auditor (anthropic/claude-4-sonnet) flags numerical/physical constraint violations, and the CEO (Qwen/Qwen2.5-72B-Instruct) issues ACCEPT, REGENERATE_PARTIAL, or REGENERATE_FULL. On REGENERATE_FULL, inject the CEO guidance back into the Generator prompt; on REGENERATE_PARTIAL, the Editor (meta-llama/Llama-3.3-70B) makes targeted fixes verified by the Approver (mistralai/Mistral-Small-3.2-24B) before retry. For critical or generalizable failures, extract a persistent rule into knowledge_base.txt so it is injected into all future generations, preventing repeated mistakes across families and seasons.

## Related tools

- **WattCouncil** (Framework orchestrating the role-specialized LLM council for governed structured generation) — github.com/Singularity-AI-Lab/wattcouncil
- **google/gemini-2.5-flash** (Primary Generator producing candidate structured JSON)
- **anthropic/claude-4-sonnet** (Physical Auditor flagging numerical/physical constraint violations)
- **meta-llama/Llama-4-Maverick-17B** (Cultural Auditor flagging cultural/narrative inconsistencies)
- **Qwen/Qwen2.5-72B-Instruct** (CEO final arbiter issuing ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL)
- **meta-llama/Llama-3.3-70B-Instruct** (Editor making targeted fixes on partial regeneration)
- **DeepInfra API** (Primary LLM inference provider for council calls)
- **OpenRouter** (Optional fallback provider for LLM inference)
- **TMY (Typical Meteorological Year) weather data** (Real-data alternative weather input conditioning Stage 3 consumption generation)

## Examples

```
python main.py --config config/runtime.yaml --weather-config config/weather.yaml
```

## Evaluation signals

- Accepted JSON must pass schema validation against schemas/stage3_consumption.*
- CEO decision must be ACCEPT after the auditor review loop (no outstanding cultural or physical violations)
- Physical Auditor reports no numerical/physical constraint violations (e.g., humidity < 100% style rules promoted to knowledge_base.txt)
- scripts/analyze_logs.py reports nonzero calls for Generator, Cultural Auditor, Physical Auditor, and CEO roles and total pipeline cost within expected range (example run: $0.1506 total)
- Regeneration behavior matches CEO verdict: REGENERATE_FULL triggers a fresh Generator attempt with injected guidance; REGENERATE_PARTIAL routes through Editor and Approver

## Limitations

- Quality depends on the council loop converging; Approver REJECTED verdicts trigger full regeneration and restart, incurring additional API cost and time
- Knowledge base rules accumulate per run; without them the council can repeat the same mistakes across different families or seasons
- Requires a DeepInfra API key (or OpenRouter fallback); generation is model- and provider-dependent (Python 3.11 conda env 'llm_council')

## Evidence

- [methods] Primary Generator produces candidate JSON outputs: "Produces candidate structured outputs (JSON)"
- [methods] CEO final decision: "Final arbiter: ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL"
- [methods] Stage 3 consumption conditioned on family and weather: "Generate hourly household energy consumption based on family and weather"
- [readme] Knowledge base persistent rules: "This rule is saved to `knowledge_base.txt` and is injected into the system prompt for *all future generations* in that run and subsequent runs."
- [readme] Immediate feedback loop on full regeneration: "When the CEO rejects an output with `REGENERATE_FULL`, the specific *guidance* is injected back into the Generator's prompt for the next attempt."
- [readme] Council roles and separation of concerns: "The framework uses **multiple LLMs with fixed roles**, ensuring separation of concerns and quality control"
