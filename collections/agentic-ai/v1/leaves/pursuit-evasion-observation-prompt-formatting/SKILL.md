---
name: pursuit-evasion-observation-prompt-formatting
description: Use when converting KSPDG per-step vessel observations into textual user/system
  prompts for an LLM pursuit agent, or when building Alpaca-format records whose instruction
  must embed the observations for throttle decisions.
when_to_use_negative:
- The task is training a control policy via reinforcement learning — KSPDG explicitly
  provides evaluation environments, NOT training environments for RL algorithms
- The observations target non-pursuit KSPDG scenarios (e.g. Lady-Bandit-Guard or Sun-blocking)
  whose observation semantics differ from 1-v-1 pursuit-evasion
- The source data is human gameplay logs not collected with the navball bot — the
  LLaMA fine-tuning dataset files were all collected with the navball bot
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: KSPDG
  role: Provides the pursuit-evasion simulation environment whose per-step observations
    are serialized into textual prompts and whose actions receive the model's throttle
    output
  repo: https://github.com/mit-ll/spacegym-kspdg/tree main
- name: tiktoken
  role: Tokenization of the textual observation prompts for LLM input length accounting
  repo: https://github.com/openai/tiktoken
- name: Hugging Face
  role: Hosts the resulting models and the Alpaca-format fine-tuning datasets (LLaMA
    10/25/50-file variants)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:8694c2d841e86b7ee5dc3eb70ca9f2dc7a3ae9477533c32cdc24595568bcb826
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Hugging Face
  - KSPDG
  - tiktoken
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# pursuit-evasion-observation-prompt-formatting

## Summary

Formats KSPDG pursuit-evasion telemetry (real-time observations/state) into textual LLM user prompts, with observation augmentation using the prograde direction, so that GPT-3.5/LLaMA agents can operate a pursuing spacecraft in the classic RL loop.

## When to use

Use when converting KSPDG per-step vessel observations into textual user/system prompts for an LLM pursuit agent, or when building Alpaca-format records whose instruction must embed the observations for throttle decisions.

## When NOT to use

- The task is training a control policy via reinforcement learning — KSPDG explicitly provides evaluation environments, NOT training environments for RL algorithms
- The observations target non-pursuit KSPDG scenarios (e.g. Lady-Bandit-Guard or Sun-blocking) whose observation semantics differ from 1-v-1 pursuit-evasion
- The source data is human gameplay logs not collected with the navball bot — the LLaMA fine-tuning dataset files were all collected with the navball bot

## Inputs

- KSPDG per-step vessel observation/state records (e.g. from scenario E3 navball-bot gameplay logs in the kspdg agents-data directory)
- Pursuit-agent system prompt template (vessel-frame throttle axes, max throttle 1, 'Reason step-by-step.')
- Prior dialogue turns (for few-shot history fields)

## Outputs

- Textual user prompts embedding mission observations (with prograde-direction augmentation) suitable for GPT-3.5 or LLaMA input
- Alpaca-format records with 'instruction', 'output', 'system', and 'history' fields, serializable to JSONL

## How to apply

Interface the KSPDG simulation environment with the LLM in a classic RL loop by transforming each real-time observation (or state) of the mission into a textual user prompt and plugging the model's textual action back into KSPDG. Augment observations with the prograde direction as defined in the LLM Agent pseudocode, and present relative-state telemetry on the navball convention (throttles in the vessel frame: x right, y toward target, z up). Pair the user prompt with a system prompt specifying maximum throttle 1 and the directive 'Reason step-by-step.' so chain-of-thought reasoning precedes the throttle decision. When the agent's action invocation fails or is invalid, map the step to the added 'none' action label rather than forcing a throttle. These prompt conventions feed both the few-shot GPT-3.5 strategy and the synthetic Alpaca-format records (instruction/output/system/history) used to fine-tune the LLaMA 10/25/50-file variants.

## Related tools

- **KSPDG** (Provides the pursuit-evasion simulation environment whose per-step observations are serialized into textual prompts and whose actions receive the model's throttle output) — https://github.com/mit-ll/spacegym-kspdg/tree main
- **tiktoken** (Tokenization of the textual observation prompts for LLM input length accounting) — https://github.com/openai/tiktoken
- **Hugging Face** (Hosts the resulting models and the Alpaca-format fine-tuning datasets (LLaMA 10/25/50-file variants))

## Examples

```
git clone git@github.com:mit-ll/spacegym-kspdg.git
```

## Evaluation signals

- Field-presence check: every serialized record contains all four required fields — 'instruction', 'output', 'system', 'history'
- Action-label vocabulary check: output labels come from the throttle vocabulary {forward/left/right/up/down throttles ..., 'none'}, with 'none' assigned to steps where no valid throttle action was produced
- Prompt-level check: system prompt specifies throttles in the vessel frame (x right, y toward target, z up) with maximum throttle 1 and the 'Reason step-by-step.' directive
- Observation augmentation check: prompts include the prograde direction as defined in the LLM Agent pseudocode
- Schema check: resulting dataset matches the Alpaca schema used for the LLaMA 10/25/50-file fine-tuning variants

## Limitations

- KSPDG uses a non-blocking step function — game time does not pause while agents make decisions, so long-computed prompts inherently penalize the control policy
- Fine-tuning/inference outcomes are highly hardware-dependent and hardware specifications for local LLaMA training were not specified
- Steps with failed or invalid action invocations must fall back to the 'none' label; forcing a throttle action there would corrupt the training vocabulary
- The fine-tuned LLaMA model only moderately outperforms the navball bot itself, so prompt formatting alone does not exceed the scripted expert baseline

## Evidence

- [intro] RL loop interface: "we design the classic RL loop by interfacing the simulation environment (KSDPG) with a LLM, transforming the real-time observations (or state) of the mission as textual user prompts"
- [methods] system prompt conventions: [evidence span withheld — anchor status: not_found]
- [methods] navball-bot data source: [evidence span withheld — anchor status: not_found]
- [methods] none action label: [evidence span withheld — anchor status: not_found]
- [readme] evaluation not training: [evidence span withheld — anchor status: not_found]
