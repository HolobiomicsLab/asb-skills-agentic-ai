---
name: telemetry-to-prompt-rendering
description: Use when you have KSPDG gameplay logs or real-time observation dictionaries
  from a pursuit-evasion episode and need to convert each timestep's telemetry into
  a natural-language prompt and discrete vessel-frame thrust label for an LLM agent.
when_to_use_negative:
- KSPDG environments are evaluation, not training, environments — do not use this
  rendering pipeline to train RL policies via massively parallel KSP rollouts (KSP
  was not designed for headless faster-than-real-time execution)
- The task output is not discrete vessel-frame throttle actions (e.g. continuous control
  or non-pursuit scenarios like sun-blocking without a capture target)
- Observations are already in a structured agent-ready format (e.g. feeding a Lambert-MPC
  or navball bot that consumes numeric state directly)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3398
tools:
- name: KSPDG
  role: Source of the observation dictionaries and episode logs; the RL-loop environment
    whose real-time observations are rendered into textual prompts and whose actions
    the LLM's perform_action output is plugged back into
  repo: https://github.com/mit-ll/spacegym-kspdg
- name: Kerbal Space Program (KSP)
  role: Game engine running the simulation that produces the telemetry (via kRPC)
    sampled every 0.5 seconds during gameplay
- name: tiktoken
  role: Token counting of rendered prompts to manage context/token budgets for GPT/LLaMA
    input
  repo: https://github.com/openai/tiktoken
- name: Hugging Face
  role: Hosts the resulting LLaMA fine-tuning datasets (10/25/50 files, navball bot)
    produced by this rendering workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:dfc2db9e35451400d62e023e84f771dcea124005a2fa1c9255a6e62f7d53dd0c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mit-ll/spacegym-kspdg
  tools:
  - Hugging Face
  - KSPDG
  - Kerbal Space Program (KSP)
  - tiktoken
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3398
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# telemetry-to-prompt-rendering

## Summary

Renders KSPDG simulation observations (relative positions, velocities, fuel, prograde) recorded every 0.5 seconds into structured textual user prompts and paired thrust-action assistant responses for LLM prompt-engineering or fine-tuning, forming the classic RL loop between KSPDG and GPT-3.5/LLaMA models.

## When to use

Use when you have KSPDG gameplay logs or real-time observation dictionaries from a pursuit-evasion episode and need to convert each timestep's telemetry into a natural-language prompt and discrete vessel-frame thrust label for an LLM agent.

## When NOT to use

- KSPDG environments are evaluation, not training, environments — do not use this rendering pipeline to train RL policies via massively parallel KSP rollouts (KSP was not designed for headless faster-than-real-time execution)
- The task output is not discrete vessel-frame throttle actions (e.g. continuous control or non-pursuit scenarios like sun-blocking without a capture target)
- Observations are already in a structured agent-ready format (e.g. feeding a Lambert-MPC or navball bot that consumes numeric state directly)

## Inputs

- Recorded human gameplay and navball-bot episode logs from the kspdg agents_data folder (observations and thrust actions sampled every 0.5 seconds)
- KSPDG observation dictionaries with relative positions, velocities, fuel, and prograde fields
- Prompt templates including the fixed system prompt and the user prompt template with HUMAN/ASSISTANT delimiters

## Outputs

- JSONL fine-tuning dataset where each line is a JSON object with 'system', 'user', and 'assistant' role fields (e.g. fine_tuning_data.l)
- Rendered textual user prompts embedding observation values for real-time inference or few-shot prompting
- Assistant response strings embedding perform_action(ft, rt, dt) thrust labels

## How to apply

For each logged timestep, render the observation dictionary (relative positions, velocities, fuel, prograde) into the user prompt template 'Given these observations ..., what is the best throttle to capture evader?', optionally requesting step-by-step reasoning for Chain of Thought evaluation. Prepend a fixed system prompt describing the autonomous pursuit agent and the vessel-frame throttle axes (x right, y toward target, z up, max throttle 1). Pair the rendered observation with the recorded thrust action formatted as an assistant response ending in a perform_action(ft, rt, dt) call, delimited with the HUMAN/ASSISTANT keywords from the prompt templates. Serialize each timestep as one JSON line containing 'system', 'user', and 'assistant' role fields, suitable for OpenAI fine-tuning (JSONL) or direct few-shot prompting. The rationale is to let the LLM rely solely on natural-language telemetry, per the paper's design of transforming real-time observations into textual user prompts and plugging the model's action back into KSPDG.

## Related tools

- **KSPDG** (Source of the observation dictionaries and episode logs; the RL-loop environment whose real-time observations are rendered into textual prompts and whose actions the LLM's perform_action output is plugged back into) — https://github.com/mit-ll/spacegym-kspdg
- **Kerbal Space Program (KSP)** (Game engine running the simulation that produces the telemetry (via kRPC) sampled every 0.5 seconds during gameplay)
- **tiktoken** (Token counting of rendered prompts to manage context/token budgets for GPT/LLaMA input) — https://github.com/openai/tiktoken
- **Hugging Face** (Hosts the resulting LLaMA fine-tuning datasets (10/25/50 files, navball bot) produced by this rendering workflow)

## Examples

```
python render_dataset.py --agents_data kspdg/arclab_mit/agents_data --template pursuit --out fine_tuning_data.l
```

## Evaluation signals

- Every output line parses as valid JSON
- Each line contains all three role fields: 'system', 'user', and 'assistant'
- The assistant field embeds a well-formed perform_action(ft, rt, dt) thrust label
- The user field contains the rendered observation values (relative positions, velocities, fuel, prograde) matching the logged timestep
- The system prompt is the fixed autonomous-pursuit-agent description with vessel-frame throttle axes (x right, y toward target, z up, max throttle 1)

## Limitations

- KSP game time does not pause while agents make decisions (non-blocking step function), so slow LLM inference incurs an inherent penalty — rendered prompts must support fast decisions
- All LLaMA fine-tuning dataset files were collected using the navball bot, so the data reflects navball-bot behavior rather than diverse human strategies
- Fine-tuning on as few as 2 human gameplays achieved very low training loss (1.3e-4) but validation loss worsened to 4.4e-1, indicating overfitting on small rendered datasets
- Prompt-rendered token counts are hardware/model dependent; token budgets were checked with tiktoken

## Evidence

- [intro] RL loop interface: "we design the classic RL loop by interfacing the simulation environment (KSDPG) with a LLM, transforming the real-time observations (or state) of the mission as textual user prompts"
- [intro] prompt strategies: [evidence span withheld — anchor status: not_found]
- [results] CoT improvement: "The application of the Chain of Thought approach demonstrates a significant improvement in the generalization of spacecraft piloting techniques"
- [methods] navball bot data source: [evidence span withheld — anchor status: not_found]
- [readme] non-blocking env: [evidence span withheld — anchor status: not_found]
