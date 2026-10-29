---
name: llm-multi-agent-orchestration
description: Use when generating structured JSON outputs (household compositions,
  weather profiles, hourly consumption) that must satisfy cultural, temporal, and
  physical constraints and require adversarial multi-model review rather than a single
  LLM call.
when_to_use_negative:
- The task needs only a single forward LLM call with no schema or constraint validation
  — the council's auditors, Editor/Approver, and CEO loop add cost and latency
- Deterministic ground-truth data exists (e.g. real TMY meteorological records), where
  processing measured data directly beats LLM generation
- The candidate outputs cannot be expressed as JSON validated against a fixed schema,
  since auditors and the CEO operate on schema-checked structured candidates
edam_operation: http://edamontology.org/operation_3766
tools:
- name: WattCouncil
  role: Framework implementing the governed council pipeline (generator -> auditors
    -> editor -> approver -> CEO)
  repo: github.com/Singularity-AI-Lab/wattcouncil
- name: google/gemini-2.5-flash
  role: Primary Generator producing candidate structured JSON
  repo: github.com/Singularity-AI-Lab/wattcouncil
- name: meta-llama/Llama-4-Maverick-17B
  role: Cultural Auditor flagging cultural/narrative inconsistencies
  repo: github.com/Singularity-AI-Lab/wattcouncil
- name: anthropic/claude-4-sonnet
  role: Physical Auditor flagging numerical/physical constraint violations
  repo: github.com/Singularity-AI-Lab/wattcouncil
- name: meta-llama/Llama-3.3-70B-Instruct
  role: Editor applying targeted fixes on REGENERATE_PARTIAL
  repo: github.com/Singularity-AI-Lab/wattcouncil
- name: mistralai/Mistral-Small-3.2-24B
  role: Approver verifying editor changes; REJECTED triggers full regeneration
  repo: github.com/Singularity-AI-Lab/wattcouncil
- name: Qwen/Qwen2.5-72B-Instruct
  role: CEO final arbiter issuing ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL plus
    GUIDANCE
  repo: github.com/Singularity-AI-Lab/wattcouncil
- name: DeepInfra API
  role: Primary LLM provider, with OpenRouter as fallback
  repo: github.com/Singularity-AI-Lab/wattcouncil
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_005
  - task_001
  - task_007
  source_papers:
  - doi: 10.1145/3744256.3812595
    title: 'WattCouncil: Context-Aware Household Energy Scenario Generation With Governed
      LLMs'
  source_content_hash: sha256:22b584b2db72531a830629b13037a41ff6ef15d8a30776ae4a12903444e17048
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
  - mistralai/Mistral-Small-3.2-24B
  - pip
  - scripts/analyze_logs.py
  edam_operation: http://edamontology.org/operation_3766
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# llm-multi-agent-orchestration

## Summary

A governed multi-LLM council pattern in which a Primary Generator (google/gemini-2.5-flash) produces candidate structured JSON that is audited by Cultural (meta-llama/Llama-4-Maverick-17B) and Physical (anthropic/claude-4-sonnet) Auditors, optionally repaired by an Editor/Approver pair, and accepted only by a CEO arbiter (Qwen/Qwen2.5-72B-Instruct). Used in WattCouncil to generate culturally and physically valid household energy scenarios under explicit constraints.

## When to use

Use when generating structured JSON outputs (household compositions, weather profiles, hourly consumption) that must satisfy cultural, temporal, and physical constraints and require adversarial multi-model review rather than a single LLM call.

## When NOT to use

- The task needs only a single forward LLM call with no schema or constraint validation — the council's auditors, Editor/Approver, and CEO loop add cost and latency
- Deterministic ground-truth data exists (e.g. real TMY meteorological records), where processing measured data directly beats LLM generation
- The candidate outputs cannot be expressed as JSON validated against a fixed schema, since auditors and the CEO operate on schema-checked structured candidates

## Inputs

- config/models.yaml role-to-model assignments
- config/runtime.yaml pipeline parameters (country, year, num_families, num_variants)
- Stage prompt templates from prompts/stage1_family/, stage2_weather/, stage3_consumption/
- JSON validation schemas from schemas/
- knowledge_base.txt persistent rules
- Prior stage outputs (stage1_families_*. and stage2_hourly_*. JSON)

## Outputs

- Schema-validated structured JSON (stage1_families_*, stage2_hourly_*, stage3_consumption_*) with a CEO ACCEPT decision
- knowledge_base.txt updated with extracted failure rules
- Per-role usage logs aggregated by scripts/analyze_logs.py (calls, tokens, time, USD cost)

## How to apply

Read role-to-model assignments from config/models.yaml and initialize the provider-agnostic client (council/client.py) against the DeepInfra API with OpenRouter as fallback. Run the Primary Generator to emit candidate JSON validated against stage schemas (e.g. schemas/stage1_family., schemas/stage2_hourly_weather., schemas/stage3_consumption.); on failure the candidate enters a review loop where the Cultural Auditor flags narrative inconsistencies, the Physical Auditor flags numerical/physical violations, and the CEO emits one of ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL with GUIDANCE text. For REGENERATE_PARTIAL, the Editor (meta-llama/Llama-3.3-70B-Instruct) applies targeted fixes verified by the Approver (mistralai/Mistral-Small-3.2-24B); an Approver REJECTED verdict triggers REGENERATE_FULL back at the Generator. Inject knowledge_base.txt rules and prior CEO guidance into the Generator's system prompt on every retry so persistent failures become durable rules. The loop must terminate only on a CEO ACCEPT; every intermediate item is checkpointed to raw/ for smart resume.

## Related tools

- **WattCouncil** (Framework implementing the governed council pipeline (generator -> auditors -> editor -> approver -> CEO)) — github.com/Singularity-AI-Lab/wattcouncil
- **google/gemini-2.5-flash** (Primary Generator producing candidate structured JSON) — github.com/Singularity-AI-Lab/wattcouncil
- **meta-llama/Llama-4-Maverick-17B** (Cultural Auditor flagging cultural/narrative inconsistencies) — github.com/Singularity-AI-Lab/wattcouncil
- **anthropic/claude-4-sonnet** (Physical Auditor flagging numerical/physical constraint violations) — github.com/Singularity-AI-Lab/wattcouncil
- **meta-llama/Llama-3.3-70B-Instruct** (Editor applying targeted fixes on REGENERATE_PARTIAL) — github.com/Singularity-AI-Lab/wattcouncil
- **mistralai/Mistral-Small-3.2-24B** (Approver verifying editor changes; REJECTED triggers full regeneration) — github.com/Singularity-AI-Lab/wattcouncil
- **Qwen/Qwen2.5-72B-Instruct** (CEO final arbiter issuing ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL plus GUIDANCE) — github.com/Singularity-AI-Lab/wattcouncil
- **DeepInfra API** (Primary LLM provider, with OpenRouter as fallback) — github.com/Singularity-AI-Lab/wattcouncil

## Examples

```
conda activate llm_council; python main.py --config config/runtime_targeted.yaml --weather-config config/weather.yaml
```

## Evaluation signals

- Each council cycle terminates only on a CEO ACCEPT; no other exit path exists
- Every CEO decision is one of the three enumerated actions: ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL
- An Approver REJECTED verdict restarts generation from the Primary Generator (full regeneration)
- Every emitted record parses as JSON and validates against its stage schema (e.g. schemas/stage1_family., schemas/stage3_consumption.)
- Usage totals reconcile via scripts/analyze_logs.py to the reference table: 20 calls, 215.2 s, 75,782 in / 18,321 out tokens, $0.1506 total ($0.0075 avg/call, 10.76 s avg)

## Limitations

- Model assignments are fixed in config/models.yaml; the default run incurs measurable cost per role (e.g. Generator $0.0476, Physical Auditor $0.0760, total $0.1506 for 20 calls)
- Critical failures must be extracted by the CEO into knowledge_base.txt; without persistent rule accumulation the council can repeat the same mistakes across families or seasons
- Resume relies on the checkpoint system: every item must be saved to raw/ directories, and corrupted checkpoint files are detected and skipped before smart resume

## Evidence

- [readme] CEO roles and decision set: "Final arbiter: ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL"
- [readme] Knowledge base persistence: "This rule is saved to `knowledge_base.txt` and is injected into the system prompt for *all future generations* in that run and subsequent runs."
- [readme] Approver rejection semantics: "Verifies editor changes meet the guidance requirements"
- [methods] Generator role usage: "Generator          google/gemini-2.5-flash             5      186.8      10,991       17,739       0.0476"
- [methods] Council setup: "conda create -n llm_council python=3.11 -y"
