---
name: weather-conditioned-household-scenario-modeling
description: Use when you need synthetic, context-aware hourly household electricity
  demand profiles tied to a specific country, year, and weather scenario (e.g., UAE
  2024) rather than generic statistical load profiles.
when_to_use_negative:
- You need measured/real load data for households — use an empirical dataset such
  as the CER dataset (over a year of load measurements for 4232 households with survey-based
  socio-economic information), which WattCouncil itself uses only for evaluation.
- Your task is Stage 1 family composition or Stage 2 weather generation — this skill
  covers only Stage 3 consumption patterns, which consumes those stages' outputs.
- You lack the required council components (auditors, CEO, schema); running only the
  Generator without audit would violate the framework's constraint-driven governance
  model.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: WattCouncil
  role: 3-stage pipeline and LLM council framework that generates, audits, and validates
    the hourly consumption profiles
  repo: github.com/Singularity-AI-Lab/wattcouncil
- name: google/gemini-2.5-flash
  role: Primary Generator producing candidate structured JSON of hourly consumption
    conditioned on family and weather
- name: anthropic/claude-4-sonnet
  role: Physical Auditor flagging numerical/physical constraint violations in the
    candidate profile
- name: meta-llama/Llama-4-Maverick-17B
  role: Cultural Auditor flagging cultural/narrative inconsistencies
- name: Qwen/Qwen2.5-72B-Instruct
  role: CEO issuing ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL decisions with regeneration
    guidance
- name: meta-llama/Llama-3.3-70B-Instruct
  role: Editor making targeted fixes when the CEO requests partial regeneration
- name: DeepInfra API
  role: LLM inference provider used for council model calls (OpenRouter as optional
    fallback)
- name: TMY (Typical Meteorological Year) weather data
  role: Real-data alternative source for the Stage 2 seasonal weather profile conditioning
    Stage 3
- name: Python 3.11 / conda
  role: Runtime environment (conda env 'llm_council') hosting the pipeline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1145/3744256.3812595
    title: 'WattCouncil: Context-Aware Household Energy Scenario Generation With Governed
      LLMs'
  source_content_hash: sha256:7b7cdaffd00d54c1577daa78811aadb09daea5e43bfa89369b03fb390ab11e46
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepInfra API
  - OpenRouter
  - Python
  - Python 3.11 / conda
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
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# weather-conditioned-household-scenario-modeling

## Summary

Generates hourly household energy consumption profiles by conditioning an LLM-based generation pipeline on a culturally composed household (family JSON from Stage 1) and a seasonal weather profile (LLM-generated or TMY-derived from Stage 2), within WattCouncil's governed council architecture.

## When to use

Use when you need synthetic, context-aware hourly household electricity demand profiles tied to a specific country, year, and weather scenario (e.g., UAE 2024) rather than generic statistical load profiles.

## When NOT to use

- You need measured/real load data for households — use an empirical dataset such as the CER dataset (over a year of load measurements for 4232 households with survey-based socio-economic information), which WattCouncil itself uses only for evaluation.
- Your task is Stage 1 family composition or Stage 2 weather generation — this skill covers only Stage 3 consumption patterns, which consumes those stages' outputs.
- You lack the required council components (auditors, CEO, schema); running only the Generator without audit would violate the framework's constraint-driven governance model.

## Inputs

- Stage 1 family JSON (culturally appropriate household composition and work regime)
- Stage 2 hourly weather JSON (LLM-generated or TMY-derived seasonal profile)
- JSON schema schemas/stage3_consumption
- Generator prompts from prompts/stage3_consumption/
- knowledge_base.txt accumulated rules (if present)

## Outputs

- Accepted hourly household energy consumption profile JSON (stage3_consumption_*) in the timestamped run outputs/raw directory
- Optional CSV conversion of the accepted JSON via scripts/convert_outputs.py
- API call and prompt logs via utils/api_logger.py

## How to apply

Configure config/runtime.yaml with the target country and year (e.g., United Arab Emirates, 2024) and enable the Stage 3 consumption stage; model assignments come from config/models.yaml (google/gemini-2.5-flash as Generator, anthropic/claude-4-sonnet as Physical Auditor, Qwen/Qwen2.5-72B-Instruct as CEO). Load the Stage 1 family JSON and the Stage 2 hourly weather JSON (LLM-generated or TMY real data), compose the Generator prompt from prompts/stage3_consumption/ with any accumulated knowledge_base.txt rules injected, and run the Generator to produce candidate structured JSON of hourly consumption conditioned on family and weather. Subject the candidate to the council review loop: the Cultural Auditor (meta-llama/Llama-4-Maverick-17B) flags cultural/narrative inconsistencies, the Physical Auditor (anthropic/claude-4-sonnet) flags numerical/physical constraint violations, and the CEO issues ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL, with the Editor (meta-llama/Llama-3.3-70B) and Approver handling targeted fixes before any full regeneration. Save accepted profiles as stage3_consumption_* files in the timestamped run outputs/raw directory, logging all API calls and prompts via utils/api_logger.py. Rationale: the framework produces context-sensitive daily routines by incorporating household composition, temporal factors, and environmental conditions, with separation of concerns across specialized council roles preventing single-model errors from propagating.

## Related tools

- **WattCouncil** (3-stage pipeline and LLM council framework that generates, audits, and validates the hourly consumption profiles) — github.com/Singularity-AI-Lab/wattcouncil
- **google/gemini-2.5-flash** (Primary Generator producing candidate structured JSON of hourly consumption conditioned on family and weather)
- **anthropic/claude-4-sonnet** (Physical Auditor flagging numerical/physical constraint violations in the candidate profile)
- **meta-llama/Llama-4-Maverick-17B** (Cultural Auditor flagging cultural/narrative inconsistencies)
- **Qwen/Qwen2.5-72B-Instruct** (CEO issuing ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL decisions with regeneration guidance)
- **meta-llama/Llama-3.3-70B-Instruct** (Editor making targeted fixes when the CEO requests partial regeneration)
- **DeepInfra API** (LLM inference provider used for council model calls (OpenRouter as optional fallback))
- **TMY (Typical Meteorological Year) weather data** (Real-data alternative source for the Stage 2 seasonal weather profile conditioning Stage 3)
- **Python 3.11 / conda** (Runtime environment (conda env 'llm_council') hosting the pipeline)

## Examples

```
python main.py --config config/runtime.yaml --weather-config config/weather.yaml
```

## Evaluation signals

- Produced hourly consumption JSON passes schema validation against schemas/stage3_consumption
- CEO issues a final ACCEPT decision after Cultural and Physical Auditor review
- Physical Auditor reports no numerical/physical constraint violations (e.g., energy values consistent with the conditioned weather and family context)
- Cultural Auditor flags no cultural/narrative inconsistencies between consumption routine and household composition/work regime
- Accepted files appear as stage3_consumption_* in outputs/raw with API calls and prompts logged via utils/api_logger.py

## Limitations

- Quality depends on upstream Stage 1 family and Stage 2 weather JSON; invalid or culturally implausible inputs propagate into Stage 3.
- Council iteration costs API budget and time — per-run costs are tracked via model_costs.yaml and scripts/analyze_logs.py (an example run totaled ~$0.15 for 20 calls), and failures can trigger REGENERATE_FULL loops.
- Validation against the CER dataset is external: generated profiles must still be compared to measured data to establish realism; ablation studies assess framework consistency rather than guarantee per-profile accuracy.

## Evidence

- [methods] Hourly consumption conditioned on family and weather is the core Stage 3 operation: [evidence span withheld — anchor status: not_found]
- [abstract] Framework rationale: context-sensitive routines from composition, temporal, and environmental factors: "our framework produces context-sensitive daily routines through a guided reasoning process that incorporates household composition, temporal factors, and environmental conditions"
- [readme] Weather profiles can come from LLM generation or TMY real data: "Generate seasonal weather profiles (using **LLM** or **TMY** real data)"
- [readme] CEO governs acceptance with regeneration decisions: "Final arbiter: ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL"
- [abstract] Evaluation basis for generated profiles: "We evaluate the generated profiles against the detailed CER dataset, which contains over a year of load measurements for 4232 households together with survey-based socio-economic information."
