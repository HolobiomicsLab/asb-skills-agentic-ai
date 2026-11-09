---
name: household-energy-scenario-generation
description: Use when you need to synthesize realistic hourly household energy consumption
  profiles tied to family composition, seasonal weather, and cultural context (e.g.,
  UAE-style scenarios), or benchmark such profiles against measured data like the
  CER dataset.
when_to_use_negative:
- You need measured consumption data rather than synthetic scenarios — WattCouncil
  outputs are LLM-generated profiles, only evaluated against the real CER dataset
- You need deterministic, non-LLM load modeling — the pipeline depends on LLM agents
  and API availability (DeepInfra/OpenRouter)
- Your scenario domain requires different governance roles or constraint regimes not
  expressible in the council's fixed prompts and JSON schemas
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3855
tools:
- name: WattCouncil
  role: Core framework implementing the governed LLM council and 3-stage generation
    pipeline
  repo: github.com/Singularity-AI-Lab/wattcouncil
- name: google/gemini-2.5-flash
  role: Primary Generator producing candidate structured JSON outputs
- name: anthropic/claude-4-sonnet
  role: Physical Auditor flagging numerical/physical constraint violations
- name: meta-llama/Llama-4-Maverick-17B
  role: Cultural Auditor flagging cultural/narrative inconsistencies
- name: meta-llama/Llama-3.3-70B-Instruct
  role: Editor making targeted fixes on partial regeneration
- name: mistralai/Mistral-Small-3.2-24B
  role: Approver verifying editor changes; rejection triggers full regeneration
- name: Qwen/Qwen2.5-72B-Instruct
  role: CEO final arbiter issuing ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL with
    guidance
- name: DeepInfra API
  role: Primary LLM inference provider for council calls
- name: OpenRouter
  role: Optional fallback LLM provider
- name: scripts/analyze_logs.py
  role: Aggregates per-role API calls, execution time, tokens, and USD cost per run
  repo: github.com/Singularity-AI-Lab/wattcouncil
- name: TMY (Typical Meteorological Year) weather data
  role: Real-data alternative to LLM-generated seasonal weather profiles in Stage
    2
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1145/3744256.3812595
    title: 'WattCouncil: Context-Aware Household Energy Scenario Generation With Governed
      LLMs'
  source_content_hash: sha256:0860a3a182c59eeaa5dd7bc8db469a4b1a8fc1b1e8a7dfc85d16be701c3eec0a
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
  - mistralai/Mistral-Small-3.2-24B
  - pip
  - scripts/analyze_logs.py
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3855
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.47368421052631576
n_claims: 19
---

# household-energy-scenario-generation

## Summary

Generates context-aware synthetic household electricity demand scenarios using WattCouncil, a role-specialized LLM council framework where agents generate, audit, and validate structured energy scenarios under explicit cultural, temporal, and physical constraints across a 3-stage pipeline.

## When to use

Use when you need to synthesize realistic hourly household energy consumption profiles tied to family composition, seasonal weather, and cultural context (e.g., UAE-style scenarios), or benchmark such profiles against measured data like the CER dataset.

## When NOT to use

- You need measured consumption data rather than synthetic scenarios — WattCouncil outputs are LLM-generated profiles, only evaluated against the real CER dataset
- You need deterministic, non-LLM load modeling — the pipeline depends on LLM agents and API availability (DeepInfra/OpenRouter)
- Your scenario domain requires different governance roles or constraint regimes not expressible in the council's fixed prompts and JSON schemas

## Inputs

- config/runtime.yaml (country, year, stage toggles, num_families, num_variants)
- config/models.yaml (fixed council role-to-model assignments)
- config/weather.yaml (capital coordinates, TMY weather settings, southern-hemisphere list)
- .env with DEEPINFRA_API_KEY (and optional OPENROUTER_API_KEY)
- Optional: config/runtime_targeted.yaml with representative (num_people, house_type, composition) combos to match the CER dataset

## Outputs

- Timestamped run directory with stage1_families_*, stage2_hourly_*, and stage3_consumption_* outputs
- Structured JSON scenario artifacts per household (families, weather profiles, hourly consumption)
- knowledge_base.txt with persistent rules extracted by the CEO from critical failures
- usage_metrics.csv / analyze_logs.py report of per-role calls, time, tokens, and cost

## How to apply

Set up a conda environment (Python 3.11), install requirements via pip, and configure a DeepInfra API key (OpenRouter fallback) in .env. Run the 3-stage pipeline with 'python main.py' using config/runtime.yaml (country: United Arab Emirates, year: 2024): Stage 1 generates culturally appropriate household compositions and work regimes, Stage 2 generates seasonal weather profiles (via LLM or TMY real data), and Stage 3 generates hourly consumption conditioned on family and weather. The council roles are fixed in config/models.yaml — google/gemini-2.5-flash as Primary Generator producing candidate JSON, anthropic/claude-4-sonnet as Physical Auditor flagging numerical/physical constraint violations, meta-llama/Llama-4-Maverick-17B as Cultural Auditor, meta-llama/Llama-3.3-70B-Instruct as Editor, mistralai/Mistral-Small-3.2-24B as Approver, and Qwen/Qwen2.5-72B-Instruct as CEO issuing ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL decisions. Every generated item is checkpointed to raw/ directories so runs can resume with 'python main.py --resume run_<timestamp>' without regenerating completed work; for REGENERATE_FULL, CEO guidance is injected back into the Generator's prompt, and persistent rules are accumulated in knowledge_base.txt across generations.

## Related tools

- **WattCouncil** (Core framework implementing the governed LLM council and 3-stage generation pipeline) — github.com/Singularity-AI-Lab/wattcouncil
- **google/gemini-2.5-flash** (Primary Generator producing candidate structured JSON outputs)
- **anthropic/claude-4-sonnet** (Physical Auditor flagging numerical/physical constraint violations)
- **meta-llama/Llama-4-Maverick-17B** (Cultural Auditor flagging cultural/narrative inconsistencies)
- **meta-llama/Llama-3.3-70B-Instruct** (Editor making targeted fixes on partial regeneration)
- **mistralai/Mistral-Small-3.2-24B** (Approver verifying editor changes; rejection triggers full regeneration)
- **Qwen/Qwen2.5-72B-Instruct** (CEO final arbiter issuing ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL with guidance)
- **DeepInfra API** (Primary LLM inference provider for council calls)
- **OpenRouter** (Optional fallback LLM provider)
- **scripts/analyze_logs.py** (Aggregates per-role API calls, execution time, tokens, and USD cost per run) — github.com/Singularity-AI-Lab/wattcouncil
- **TMY (Typical Meteorological Year) weather data** (Real-data alternative to LLM-generated seasonal weather profiles in Stage 2)

## Examples

```
python main.py --config config/runtime.yaml --weather-config config/weather.yaml && python scripts/analyze_logs.py outputs/run_20260206_090431
```

## Evaluation signals

- analyze_logs.py reference totals reproduce: 20 total calls, 215.2 s cumulative time, 75,782 input tokens, 18,321 output tokens, $0.1506 total cost, $0.0075 average cost per call, 10.76 s average time per call
- Generator role (google/gemini-2.5-flash) row shows 5 calls, 186.8 s, 10,991 input tokens, 17,739 output tokens, $0.0476
- Run directory contains complete stage1_families_*, stage2_hourly_*, and stage3_consumption_* outputs with no unregenerated checkpoint gaps
- All generated JSON validates against the schemas/ definitions and passes Physical Auditor constraint checks (e.g., humidity < 100%)
- Generated profiles evaluated against the CER dataset (over a year of load measurements for 4232 households) show plausible distributional match

## Limitations

- Physical Auditor uses anthropic/claude-4-sonnet, which dominated cost ($0.0760 of $0.1506 total) despite few output tokens — model choice drives cost asymmetry
- Editor and Approver made 0 calls in the default run (all outputs accepted without partial regeneration), so those pathways are untested at default settings
- Corrupted checkpoint files are auto-detected and skipped; runs must be resumed with --resume to avoid regenerating completed items and paying duplicate API costs
- Weather stage quality depends on LLM generation unless TMY real data is supplied via --weather-config

## Evidence

- [abstract] This work presents WattCouncil, a data-generation framework in which household electricity demand is generated by a council of Large Language Model (LLM)–based agents: "This work presents WattCouncil, a data-generation framework in which household electricity demand is generated by a council of Large Language Model (LLM)–based agents"
- [methods] Generate hourly household energy consumption based on family and weather: "Generate hourly household energy consumption based on family and weather"
- [abstract] We evaluate the generated profiles against the detailed CER dataset, which contains over a year of load measurements for 4232 households together with survey-based socio-economic information.: "We evaluate the generated profiles against the detailed CER dataset, which contains over a year of load measurements for 4232 households together with survey-based socio-economic information."
- [readme] Every generated item is immediately saved to `raw/` directories: "1. **Automatic Checkpointing** - Every generated item is immediately saved to `raw/` directories"
- [readme] Final arbiter: ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL: "└── Final arbiter: ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL    │"
- [readme] This rule is saved to `knowledge_base.txt` and is injected into the system prompt for *all future generations*: "This rule is saved to `knowledge_base.txt` and is injected into the system prompt for *all future generations* in that run and subsequent runs."
