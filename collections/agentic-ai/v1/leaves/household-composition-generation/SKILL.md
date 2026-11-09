---
name: household-composition-generation
description: Use when synthetic household composition records (num_people, house_type,
  composition) must be generated for energy-demand simulation and need cultural/physical
  validation under explicit constraints.
when_to_use_negative:
- When the goal is weather profiles or hourly consumption (Stages 2 and 3), which
  build on Stage 1 output rather than producing household compositions
- When real socio-economic household data (e.g., the CER survey data) is already available
  and synthetic generation is unnecessary
- When only one LLM is desired without governance — this skill's value depends on
  the multi-role council with auditors and CEO arbitration
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: WattCouncil
  role: Framework orchestrating the 3-stage pipeline and the LLM council roles for
    Stage 1 family generation
  repo: https://github.com/Singularity-AI-Lab/wattcouncil
- name: google/gemini-2.5-flash
  role: Primary Generator producing candidate household composition JSON records via
    DeepInfra API
  repo: https://github.com/Singularity-AI-Lab/wattcouncil
- name: meta-llama/Llama-4-Maverick-17B
  role: Cultural Auditor flagging cultural/narrative inconsistencies in generated
    families
  repo: https://github.com/Singularity-AI-Lab/wattcouncil
- name: Qwen/Qwen2.5-72B-Instruct
  role: CEO final arbiter issuing ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL decisions
    and guidance
  repo: https://github.com/Singularity-AI-Lab/wattcouncil
- name: DeepInfra API
  role: LLM provider used for council model calls (OpenRouter optional fallback)
- name: Python 3.11
  role: Runtime environment for the pipeline (conda env llm_council)
  repo: https://github.com/Singularity-AI-Lab/wattcouncil
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1145/3744256.3812595
    title: 'WattCouncil: Context-Aware Household Energy Scenario Generation With Governed
      LLMs'
  source_content_hash: sha256:c83fcaa230869a515bd64dd6c53411f8f1fc4a94dc8d595803e7e88c41f567a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Singularity-AI-Lab/wattcouncil
  tools:
  - DeepInfra API
  - Python
  - Python 3.11
  - Qwen/Qwen2.5-72B-Instruct
  - WattCouncil
  - google/gemini-2.5-flash
  - meta-llama/Llama-4-Maverick-17B
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# household-composition-generation

## Summary

Stage 1 of the WattCouncil pipeline: generate culturally appropriate household compositions and work regimes as structured JSON records using a governed LLM council (Primary Generator, Cultural Auditor, Physical Auditor, Editor, Approver, CEO) rather than a single LLM.

## When to use

Use when synthetic household composition records (num_people, house_type, composition) must be generated for energy-demand simulation and need cultural/physical validation under explicit constraints.

## When NOT to use

- When the goal is weather profiles or hourly consumption (Stages 2 and 3), which build on Stage 1 output rather than producing household compositions
- When real socio-economic household data (e.g., the CER survey data) is already available and synthetic generation is unnecessary
- When only one LLM is desired without governance — this skill's value depends on the multi-role council with auditors and CEO arbitration

## Inputs

- config/runtime.yaml (country, year, num_families, num_variants, resume_from_checkpoint)
- config/runtime_targeted.yaml with representative_combos lists like [1, 5, 1] for targeted CER-matching generation
- Stage 1 prompt templates from prompts/stage1_family/
- JSON validation schema from schemas/stage1_family
- knowledge_base.txt persistent rules and CEO guidance (on retry iterations)
- DeepInfra API key (or OpenRouter fallback) in .env

## Outputs

- Per-record household composition JSON files written to the run's raw/ checkpoint directory
- Consolidated stage1_families_* file under outputs/run_YYYYMMDD_HHMMSS/
- Council decision trail (ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL with guidance)
- Updated knowledge_base.txt persistent rules extracted from critical failures

## How to apply

Load Stage 1 runtime parameters from config/runtime.yaml (country, year, num_families, num_variants) or config/runtime_targeted.yaml when matching a reference dataset such as CER via representative_combos filtered by (num_people, house_type, composition). Run the council: the Primary Generator (google/gemini-2.5-flash via DeepInfra API) produces candidate household JSON records, the Cultural Auditor (meta-llama/Llama-4-Maverick-17B) flags cultural/narrative inconsistencies, and the Physical Auditor flags numerical/physical constraint violations; the CEO (Qwen/Qwen2.5-72B-Instruct) issues ACCEPT, REGENERATE_PARTIAL (with Editor fixes verified by the Approver) or REGENERATE_FULL decisions. On retries, inject persistent rules from knowledge_base.txt and CEO guidance into the Generator prompt so the council learns from critical failures. Write each accepted household record to the run's raw/ checkpoint directory and emit the consolidated stage1_families_* file under outputs/run_YYYYMMDD_HHMMSS/.

## Related tools

- **WattCouncil** (Framework orchestrating the 3-stage pipeline and the LLM council roles for Stage 1 family generation) — https://github.com/Singularity-AI-Lab/wattcouncil
- **google/gemini-2.5-flash** (Primary Generator producing candidate household composition JSON records via DeepInfra API) — https://github.com/Singularity-AI-Lab/wattcouncil
- **meta-llama/Llama-4-Maverick-17B** (Cultural Auditor flagging cultural/narrative inconsistencies in generated families) — https://github.com/Singularity-AI-Lab/wattcouncil
- **Qwen/Qwen2.5-72B-Instruct** (CEO final arbiter issuing ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL decisions and guidance) — https://github.com/Singularity-AI-Lab/wattcouncil
- **DeepInfra API** (LLM provider used for council model calls (OpenRouter optional fallback))
- **Python 3.11** (Runtime environment for the pipeline (conda env llm_council)) — https://github.com/Singularity-AI-Lab/wattcouncil

## Examples

```
python main.py --config config/runtime_targeted.yaml --weather-config config/weather.yaml
```

## Evaluation signals

- Every generated record parses as JSON and validates against schemas/stage1_family
- The council CEO has issued an ACCEPT decision (cultural/physical auditor issues resolved) for each household record
- Each accepted item is checkpointed to raw/ and the consolidated stage1_families_* file exists under outputs/run_YYYYMMDD_HHMMSS/
- In targeted mode, generated families match the representative_combos (num_people, house_type, composition) tuples from config/runtime_targeted.yaml
- scripts/analyze_logs.py reports nonzero calls, tokens, and cost for Generator, Cultural Auditor, Physical Auditor, and CEO roles

## Limitations

- Corruption detection skips invalid checkpoint files, so corrupt raw/ records are silently regenerated rather than repaired
- Knowledge-base rules are run-scoped accumulation; persistent rules from one run's failures may over-constrain subsequent generations if too broad
- If the Approver REJECTS an Editor fix, a full regeneration is triggered, increasing API cost and latency

## Evidence

- [methods] stage1 prompts and schema: [evidence span withheld — anchor status: not_found]
- [methods] council decision procedure: [evidence span withheld — anchor status: not_found]
- [readme] readme architecture: "If a failure is critical or reveals a general rule (e.g., "Always ensure humidity is < 100%"), the CEO can extract a **persistent rule**."
- [readme] readme targeted config: "# Representative combinations filtered by (num_people, house_type, composition)"
- [abstract] abstract finding: "operating in specialized roles to generate, audit, and validate structured energy scenarios under explicit cultural, temporal, and physical constraints"
