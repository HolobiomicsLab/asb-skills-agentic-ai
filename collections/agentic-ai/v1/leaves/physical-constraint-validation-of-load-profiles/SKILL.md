---
name: physical-constraint-validation-of-load-profiles
description: Use when validating that generated hourly household electricity consumption
  JSON respects physical/numerical constraints (e.g., non-negative power, plausible
  magnitudes, weather-consistent usage) before accepting synthetic load profiles.
when_to_use_negative:
- When the data are real measured loads (e.g., CER dataset meter readings) rather
  than LLM-generated candidate profiles requiring audit before acceptance.
- When the issue at stake is cultural or narrative consistency of the household story
  — that is the Cultural Auditor's remit, not physical constraint validation.
- When only structural conformance to schemas/stage3_consumption. is needed (schema
  validation alone does not substitute for the physical audit in the council loop).
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3855
tools:
- name: anthropic/claude-4-sonnet
  role: Physical Auditor LLM that flags numerical/physical constraint violations in
    candidate consumption JSON
  repo: github.com/Singularity-AI-Lab/wattcouncil
- name: google/gemini-2.5-flash
  role: Primary Generator producing the candidate hourly consumption JSON that is
    audited
  repo: github.com/Singularity-AI-Lab/wattcouncil
- name: Qwen/Qwen2.5-72B-Instruct
  role: CEO final arbiter that consumes the Physical Auditor's flags and issues ACCEPT
    / REGENERATE_PARTIAL / REGENERATE_FULL
  repo: github.com/Singularity-AI-Lab/wattcouncil
- name: DeepInfra API
  role: Provider serving the council's LLM calls (OpenRouter as optional fallback)
  repo: github.com/Singularity-AI-Lab/wattcouncil
- name: WattCouncil
  role: Overall governed LLM council framework hosting the physical validation step
    in Stage 3
  repo: github.com/Singularity-AI-Lab/wattcouncil
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1145/3744256.3812595
    title: 'WattCouncil: Context-Aware Household Energy Scenario Generation With Governed
      LLMs'
  source_content_hash: sha256:ffa356b0ba80b5124f8f6dafb5ea43e047449253ca172779e5ba23b188705dee
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3855
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# physical-constraint-validation-of-load-profiles

## Summary

Audits LLM-generated hourly household energy consumption profiles for numerical and physical constraint violations using a dedicated Physical Auditor LLM role (anthropic/claude-4-sonnet) inside the WattCouncil council loop, gating acceptance of the synthetic load data.

## When to use

Use when validating that generated hourly household electricity consumption JSON respects physical/numerical constraints (e.g., non-negative power, plausible magnitudes, weather-consistent usage) before accepting synthetic load profiles.

## When NOT to use

- When the data are real measured loads (e.g., CER dataset meter readings) rather than LLM-generated candidate profiles requiring audit before acceptance.
- When the issue at stake is cultural or narrative consistency of the household story — that is the Cultural Auditor's remit, not physical constraint validation.
- When only structural conformance to schemas/stage3_consumption. is needed (schema validation alone does not substitute for the physical audit in the council loop).

## Inputs

- Candidate structured JSON of hourly household energy consumption from the Primary Generator (stage3 candidate output)
- Stage 1 family JSON (household composition and work regime)
- Stage 2 hourly weather JSON (LLM-generated or TMY-derived)
- JSON schema schemas/stage3_consumption.
- Prompts from prompts/stage3_consumption/ and accumulated knowledge_base.txt rules

## Outputs

- Physical Auditor report flagging numerical/physical constraint violations
- Feedback to the CEO decision (ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL)
- Accepted, physically consistent hourly consumption profiles saved as stage3_consumption_*. in outputs/raw (timestamped run directory)
- API call and prompt logs via utils/api_logger.py

## How to apply

Within the WattCouncil Stage 3 (Consumption Patterns) pipeline, the Primary Generator (google/gemini-2.5-flash) first produces candidate structured JSON of hourly consumption conditioned on the Stage 1 family JSON and Stage 2 hourly weather JSON. The Physical Auditor, assigned to anthropic/claude-4-sonnet via config/models.yaml and using prompts from prompts/stage3_consumption/, reviews the candidate and flags numerical/physical constraint violations. Its findings feed the CEO (Qwen/Qwen2.5-72B-Instruct), which issues ACCEPT, REGENERATE_PARTIAL, or REGENERATE_FULL; on partial regeneration the Editor (meta-llama/Llama-3.3-70B) makes targeted fixes verified by the Approver, while critical failures can be distilled into persistent rules in knowledge_base.txt that are injected into future Generator prompts. Rationale: separating physical validation from cultural/narrative review (Cultural Auditor, meta-llama/Llama-4-Maverick-17B) enforces separation of concerns so numerical realism is judged by a specialist role rather than diluted across general-purpose checks.

## Related tools

- **anthropic/claude-4-sonnet** (Physical Auditor LLM that flags numerical/physical constraint violations in candidate consumption JSON) — github.com/Singularity-AI-Lab/wattcouncil
- **google/gemini-2.5-flash** (Primary Generator producing the candidate hourly consumption JSON that is audited) — github.com/Singularity-AI-Lab/wattcouncil
- **Qwen/Qwen2.5-72B-Instruct** (CEO final arbiter that consumes the Physical Auditor's flags and issues ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL) — github.com/Singularity-AI-Lab/wattcouncil
- **DeepInfra API** (Provider serving the council's LLM calls (OpenRouter as optional fallback)) — github.com/Singularity-AI-Lab/wattcouncil
- **WattCouncil** (Overall governed LLM council framework hosting the physical validation step in Stage 3) — github.com/Singularity-AI-Lab/wattcouncil

## Examples

```
python main.py --config config/runtime.yaml --weather-config config/weather.yaml
```

## Evaluation signals

- The candidate hourly consumption JSON passes schema validation against schemas/stage3_consumption.
- The Physical Auditor reports no unaddressed numerical/physical constraint violations.
- The CEO issues an ACCEPT decision after auditor review (not REGENERATE_PARTIAL or REGENERATE_FULL).
- Accepted profile is saved as stage3_consumption_*. in outputs/raw with API calls and prompts logged via utils/api_logger.py.
- scripts/analyze_logs.py shows nonzero Physical Auditor calls/tokens for the run, confirming the audit role executed.

## Limitations

- Physical auditing is LLM-based, not a deterministic numerical checker — violations are flagged by claude-4-sonnet under prompt guidance, so subtle physical errors may pass and require full-regeneration guidance or persistent knowledge_base.txt rules to catch across runs.
- If the Approver REJECTS editor changes, the framework triggers a full regeneration (start over), incurring additional API cost and latency (observed ~$0.0075 average cost per call, ~10.76 s per call).
- Audit quality depends on the quality of upstream Stage 1 family and Stage 2 weather inputs; inconsistent weather or household context can induce violations the auditor must repeatedly flag.

## Evidence

- [methods] Physical Auditor flags constraint violations: [evidence span withheld — anchor status: not_found]
- [abstract] Council roles generate, audit, validate: "operating in specialized roles to generate, audit, and validate structured energy scenarios under explicit cultural, temporal, and physical constraints"
- [readme] README role description: "📐 PHYSICAL AUDITOR
     └── Flags numerical/physical constraint violations"
- [readme] CEO final decision: "Final arbiter: ACCEPT / REGENERATE_PARTIAL / REGENERATE_FULL"
- [methods] Validation criterion from source card: [evidence span withheld — anchor status: not_found]
