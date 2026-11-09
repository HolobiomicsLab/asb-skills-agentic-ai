---
name: thrust-action-label-extraction
description: Use when recorded KSPDG human or navball-bot gameplay logs must be converted
  into supervised (observation -> thrust action) pairs in OpenAI fine-tuning JSONL
  format for imitation learning of spacecraft maneuvering.
when_to_use_negative:
- The logs are already in OpenAI chat/JSONL fine-tuning format with all three role
  fields present
- The task requires an RL training environment rather than supervised imitation data
  — KSPDG explicitly provides evaluation environments, NOT training environments for
  reinforcement learning
- The agent strategy is prompt engineering or few-shot prompting rather than fine-tuning,
  since no JSONL label dataset is needed
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: KSPDG
  role: Source of the recorded differential-game (pursuit-evasion) gameplay logs whose
    observations and thrust actions form the training pairs
  repo: https://github.com/mit-ll/spacegym-kspdg/tree/main
- name: Hugging Face
  role: Hosts the resulting fine-tuning datasets (LLaMA 10/25/50 file variants) and
    trained models
  repo: https://huggingface.co
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:90bc37cbd9ca7022936e5e98b9193737dfafd51d689386d1c8ecc12ee59c4df8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Hugging Face
  - KSPDG
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# thrust-action-label-extraction

## Summary

Extracts discrete throttle (thrust) action labels from recorded KSPDG gameplay logs and pairs them with rendered observation prompts to build OpenAI fine-tuning JSONL examples for an LLM pursuit agent.

## When to use

Use when recorded KSPDG human or navball-bot gameplay logs must be converted into supervised (observation -> thrust action) pairs in OpenAI fine-tuning JSONL format for imitation learning of spacecraft maneuvering.

## When NOT to use

- The logs are already in OpenAI chat/JSONL fine-tuning format with all three role fields present
- The task requires an RL training environment rather than supervised imitation data — KSPDG explicitly provides evaluation environments, NOT training environments for reinforcement learning
- The agent strategy is prompt engineering or few-shot prompting rather than fine-tuning, since no JSONL label dataset is needed

## Inputs

- Recorded KSPDG gameplay episode logs (human gameplay and navball-bot data) from the kspdg agents_data folder
- Per-timestep observation dictionaries (relative positions, velocities, fuel, prograde) sampled every 0.5 seconds
- Discrete thrust action labels relative to the spacecraft's reference frame

## Outputs

- fine_tuning_data.l: JSONL file with one line per timestep, each containing 'system', 'user', and 'assistant' role fields
- Assistant responses embedding a perform_action(ft, rt, dt) thrust call

## How to apply

Load episode logs from the kspdg agents_data folder (human gameplay and navball-bot runs), where observations (relative positions, velocities, fuel, prograde) and thrust actions were recorded every 0.5 seconds during gameplay. For each logged timestep, render the observation dictionary into a user prompt of the form 'Given these observations ..., what is the best throttle to capture evader?' with step-by-step reasoning requested, and prepend a fixed system prompt describing the autonomous pursuit agent and the vessel-frame throttle axes (x right, y toward target, z up, max throttle 1). Format the paired thrust action as the assistant response ending in a perform_action(ft, rt, dt) call, delimited with HUMAN/ASSISTANT keywords as shown in the prompt templates. Serialize each timestep as one JSON line containing 'system', 'user', and 'assistant' role fields, saving the result as fine_tuning_data.l; these files were collected using the navball bot and used to fine-tune LLaMA models (10, 25, and 50 file variants).

## Related tools

- **KSPDG** (Source of the recorded differential-game (pursuit-evasion) gameplay logs whose observations and thrust actions form the training pairs) — https://github.com/mit-ll/spacegym-kspdg/tree/main
- **Hugging Face** (Hosts the resulting fine-tuning datasets (LLaMA 10/25/50 file variants) and trained models) — https://huggingface.co

## Examples

```
python scripts/build_fine_tuning_data.py --agents_data agents_data --out fine_tuning_data.l
```

## Evaluation signals

- Every output line in fine_tuning_data.l parses as valid JSON
- Each JSON line contains all three role fields: 'system', 'user', and 'assistant'
- Each assistant field embeds a well-formed perform_action(ft, rt, dt) thrust label
- Timestep sampling interval in the source logs matches the 0.5 s recording cadence of the original gameplay
- System prompt consistently declares the vessel-frame throttle axes (x right, y toward target, z up, max throttle 1)

## Limitations

- All dataset files were collected using the navball bot, so label diversity reflects bot behavior rather than varied expert strategies
- Fine-tuning hyperparameters (learning rate, epochs) are not reported, limiting reproducibility of the conversion-to-model pipeline
- Training on only 2 human gameplays caused validation loss to worsen (1.3e-4 train vs 4.4e-1 validation), indicating tiny human datasets overfit

## Evidence

- [methods] The fine-tuning dataset consists of observations and thrust actions recorded every 0.5 seconds during gameplay. Thrust actions are logged as discrete labels relative to the spacecraft's reference frame: [evidence span withheld — anchor status: not_found]
- [methods] It is noteworthy that all dataset files were collected using the navball bot: [evidence span withheld — anchor status: not_found]
- [readme] KSPDG provides evaluations environments, _NOT_ training environments for reinforcement learning algorithms.: [evidence span withheld — anchor status: not_found]
- [results] + 2 human gameplays 1.3 × 10−4 4.4 × 10−1: "+ 2 human gameplays 1.3 × 10−4 4.4 × 10−1"
- [methods] This section houses all the data generated from human gameplay. It is crucial for understanding human strategies and behaviors: "This section houses all the data generated from human gameplay. It is crucial for understanding human strategies and behaviors"
