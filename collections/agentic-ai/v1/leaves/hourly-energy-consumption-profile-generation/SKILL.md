---
name: hourly-energy-consumption-profile-generation
description: Use when synthetic hourly household electricity demand profiles are needed
  that are context-sensitive to household composition, seasonal weather, and cultural
  constraints, rather than purely statistical load models.
when_to_use_negative:
- When measured household load data is available (e.g. CER-style meter readings) —
  this skill generates synthetic profiles, not forecasts calibrated to real measurements.
- When only aggregate or daily load levels are needed without hourly resolution conditioned
  on household composition and weather.
- When a deterministic physics-based or statistical load model suffices and LLM-based
  narrative/cultural constraint checking is unnecessary.
edam_operation: http://edamontology.org/operation_2426
tools:
- name: WattCouncil
  role: 3-stage LLM council framework whose Stage 3 generates hourly household energy
    consumption based on family and weather
  repo: https://github.com/Singularity-AI-Lab/wattcouncil
- name: google/gemini-2.5-flash
  role: Primary Generator producing candidate structured JSON of hourly household
    energy consumption via DeepInfra
  repo: https://github.com/Singularity-AI-Lab/wattcouncil
- name: anthropic/claude-4-sonnet
  role: Physical Auditor flagging numerical/physical constraint violations in consumption
    profiles
  repo: https://github.com/Singularity-AI-Lab/wattcouncil
- name: meta-llama/Llama-4-Maverick-17B
  role: Cultural Auditor flagging cultural/narrative inconsistencies
  repo: https://github.com/Singularity-AI-Lab/wattcouncil
- name: Qwen/Qwen2.5-72B-Instruct
  role: CEO final arbiter issuing ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL
  repo: https://github.com/Singularity-AI-Lab/wattcouncil
- name: meta-llama/Llama-3.3-70B-Instruct
  role: Editor making targeted fixes on partial regeneration
  repo: https://github.com/Singularity-AI-Lab/wattcouncil
- name: TMY (Typical Meteorological Year) weather data
  role: Alternative real-data source for Stage 2 hourly weather conditioning Stage
    3 generation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1145/3744256.3812595
    title: 'WattCouncil: Context-Aware Household Energy Scenario Generation With Governed
      LLMs'
  source_content_hash: sha256:39ec14aa773849ec632214c9b1c9b74c256b17ba94b56b43d4fc161a7c11959b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Singularity-AI-Lab/wattcouncil
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# hourly-energy-consumption-profile-generation

## Summary

WattCouncil's Stage 3 generates hourly household energy consumption JSON profiles by conditioning an LLM Generator on the Stage 1 family composition and Stage 2 weather profiles, with a governed council of auditors, editor, and CEO validating cultural, physical, and numerical constraints before acceptance.

## When to use

Use when synthetic hourly household electricity demand profiles are needed that are context-sensitive to household composition, seasonal weather, and cultural constraints, rather than purely statistical load models.

## When NOT to use

- When measured household load data is available (e.g. CER-style meter readings) — this skill generates synthetic profiles, not forecasts calibrated to real measurements.
- When only aggregate or daily load levels are needed without hourly resolution conditioned on household composition and weather.
- When a deterministic physics-based or statistical load model suffices and LLM-based narrative/cultural constraint checking is unnecessary.

## Inputs

- Stage 1 family JSON (household composition and work regime)
- Stage 2 hourly weather JSON (LLM-generated or TMY-derived)
- JSON schema schemas/stage3_consumption
- Prompts from prompts/stage3_consumption/
- knowledge_base.txt accumulated rules
- config/runtime.yaml (country, year, enabled stages)
- config/models.yaml (model role assignments)

## Outputs

- Hourly household energy consumption JSON profiles (stage3_consumption_*) in outputs/raw under a timestamped run directory
- Optional CSV conversion via scripts/convert_outputs.py
- API call and prompt logs from utils/api_logger.py

## How to apply

After configuring the llm_council conda environment (Python 3.11, DeepInfra/OpenRouter keys), enable stage3 in config/runtime.yaml and compose the Generator prompt from prompts/stage3_consumption/ using the Stage 1 family JSON, the Stage 2 hourly weather JSON (LLM-generated or TMY-derived), and the JSON schema schemas/stage3_consumption, injecting any accumulated knowledge_base.txt rules. Run google/gemini-2.5-flash as Primary Generator to produce candidate hourly consumption JSON, then the council review loop: meta-llama/Llama-4-Maverick-17B (Cultural Auditor) and anthropic/claude-4-sonnet (Physical Auditor) flag violations, and Qwen/Qwen2.5-72B-Instruct as CEO issues ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL, with meta-llama/Llama-3.3-70B editing targeted fixes. Accepted profiles are saved as stage3_consumption_* JSON in the timestamped run outputs/raw directory; CEO guidance on rejection is re-injected into the Generator prompt and critical failures are persisted as rules in knowledge_base.txt for future generations.

## Related tools

- **WattCouncil** (3-stage LLM council framework whose Stage 3 generates hourly household energy consumption based on family and weather) — https://github.com/Singularity-AI-Lab/wattcouncil
- **google/gemini-2.5-flash** (Primary Generator producing candidate structured JSON of hourly household energy consumption via DeepInfra) — https://github.com/Singularity-AI-Lab/wattcouncil
- **anthropic/claude-4-sonnet** (Physical Auditor flagging numerical/physical constraint violations in consumption profiles) — https://github.com/Singularity-AI-Lab/wattcouncil
- **meta-llama/Llama-4-Maverick-17B** (Cultural Auditor flagging cultural/narrative inconsistencies) — https://github.com/Singularity-AI-Lab/wattcouncil
- **Qwen/Qwen2.5-72B-Instruct** (CEO final arbiter issuing ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL) — https://github.com/Singularity-AI-Lab/wattcouncil
- **meta-llama/Llama-3.3-70B-Instruct** (Editor making targeted fixes on partial regeneration) — https://github.com/Singularity-AI-Lab/wattcouncil
- **TMY (Typical Meteorological Year) weather data** (Alternative real-data source for Stage 2 hourly weather conditioning Stage 3 generation)

## Examples

```
python main.py --config config/runtime.yaml --weather-config config/weather.yaml
```

## Evaluation signals

- Produced hourly consumption JSON passes schema validation against schemas/stage3_consumption
- CEO issues an ACCEPT decision (not REGENERATE_PARTIAL/FULL) after Cultural and Physical Auditor review
- Physical Auditor reports no numerical/physical constraint violations (e.g. humidity < 100% class of physical rules)
- Cultural Auditor finds no cultural/narrative inconsistencies between consumption routine and family composition
- Profiles land as stage3_consumption_* files in outputs/raw with API calls and prompts logged by utils/api_logger.py

## Limitations

- Output quality depends on Stage 1 and Stage 2 upstream outputs; consumption is conditioned on family and weather JSON, so errors propagate from those stages.
- Council review incurs repeated LLM calls and cost per run (README example: $0.1506 total, 20 calls, ~10.76 s average per call for 5 generated items).
- Requires a DeepInfra API key (OpenRouter as optional fallback); runs are provider-dependent.
- Generated profiles are synthetic and validated against the CER dataset only for evaluation; they are not measured consumption.

## Evidence

- [methods] Stage 3 conditions generation on family and weather: "Generate hourly household energy consumption based on family and weather"
- [abstract] Council roles generate, audit, validate: "operating in specialized roles to generate, audit, and validate structured energy scenarios under explicit cultural, temporal, and physical constraints"
- [methods] Generator produces candidate JSON: "Produces candidate structured outputs (JSON)"
- [readme] Knowledge base persistence: "This rule is saved to `knowledge_base.txt` and is injected into the system prompt for *all future generations*"
- [methods] CEO decision loop: "Final arbiter: ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL"
