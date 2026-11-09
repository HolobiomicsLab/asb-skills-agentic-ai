---
name: flight-log-to-alpaca-dataset-conversion
description: Use when raw per-step agent/gameplay logs from KSPDG pursuit missions
  must be transformed into instruction-tuning files (Alpaca schema) for LLaMA fine-tuning,
  including a 'none' action label for steps with no valid throttle action.
when_to_use_negative:
- The logs come from human gameplay rather than the navball bot — the paper states
  all LLaMA dataset files were collected using the navball bot
- The target model is GPT via the OpenAI API fine-tuning path rather than LLaMA Alpaca-format
  training files
- You need a trainable/RL environment — the KSPDG library provides evaluation environments,
  NOT training environments for reinforcement learning
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Hugging Face
  role: Hosts the converted Alpaca-format LLaMA fine-tuning datasets (10/25/50-file
    variants) for reuse
- name: LLaMA
  role: Target model family whose fine-tuning consumes the converted Alpaca-format
    JSONL training files
- name: KSPDG
  role: Source of the navball-bot flight logs in scenario E3 used as raw input for
    conversion
  repo: https://github.com/mit-ll/spacegym-kspdg/tree main
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:58faeff3f094243ac8aa9455185938281d0e4e13076b5605e6ae749143e0d79d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Hugging Face
  - KSPDG
  - LLaMA
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# flight-log-to-alpaca-dataset-conversion

## Summary

Converts navball-bot pursuit-evasion flight logs (KSPDG scenario E3) into synthetic Alpaca-format JSONL records with dynamic chain-of-thought prompts, used to fine-tune all LLaMA models in the KSPDG spacecraft-operator study.

## When to use

Use when raw per-step agent/gameplay logs from KSPDG pursuit missions must be transformed into instruction-tuning files (Alpaca schema) for LLaMA fine-tuning, including a 'none' action label for steps with no valid throttle action.

## When NOT to use

- The logs come from human gameplay rather than the navball bot — the paper states all LLaMA dataset files were collected using the navball bot
- The target model is GPT via the OpenAI API fine-tuning path rather than LLaMA Alpaca-format training files
- You need a trainable/RL environment — the KSPDG library provides evaluation environments, NOT training environments for reinforcement learning

## Inputs

- Navball-bot gameplay log files (LLaMA fine-tuning dataset files collected with the navball bot in scenario E3)
- Observation augmentation definition (prograde direction) from the LLM Agent pseudocode
- Autonomous pursuit-agent system prompt specification (vessel-frame throttles, max throttle 1, 'Reason step-by-step.')

## Outputs

- Alpaca-format JSONL dataset file with instruction, output, system, and history fields per record
- Discrete action vocabulary including throttle labels plus the new no-action label 'none'

## How to apply

Load the navball-bot gameplay log files collected in scenario E3 from the kspdg agents-data directory and parse each per-step record into an observation-action pair, augmenting observations with the prograde direction as in the LLM Agent pseudocode. Map each pair to an Alpaca record: 'instruction' is the HUMAN prompt asking for the best throttle to capture the evader given the observations, 'output' is the assistant throttle decision, 'system' is the autonomous pursuit-agent prompt (throttles in vessel frame, x right, y toward target, z up, maximum throttle 1, 'Reason step-by-step.'), and 'history' holds prior dialogue turns. Assign the discrete 'none' action label to steps where no valid throttle action was produced (the failed/invalid action-invocation path), then serialize all records to a single JSONL dataset file. This produces the dataset files used for the LLaMA 10/25/50-file fine-tuning variants.

## Related tools

- **Hugging Face** (Hosts the converted Alpaca-format LLaMA fine-tuning datasets (10/25/50-file variants) for reuse)
- **LLaMA** (Target model family whose fine-tuning consumes the converted Alpaca-format JSONL training files)
- **KSPDG** (Source of the navball-bot flight logs in scenario E3 used as raw input for conversion) — https://github.com/mit-ll/spacegym-kspdg/tree main

## Examples

```
python convert_navball_logs.py --input kspdg/agents-data/e3_navball_logs --output llama_finetune.l --schema alpaca --action-vocab forward,left,right,up,down,none
```

## Evaluation signals

- Field-presence check: every JSONL record contains all four required fields — 'instruction', 'output', 'system', 'history'
- Action-label vocabulary check: every 'output' label is drawn from the throttle vocabulary including the added 'none' label
- Schema conformance: resulting dataset matches the Alpaca schema used for the LLaMA 10/25/50-file fine-tuning variants
- Provenance check: all source logs trace to the navball bot in scenario E3, not other agents or human gameplay
- Steps with failed/invalid action invocations are consistently labeled 'none' rather than dropped or mapped to a throttle

## Limitations

- Fine-tuning hyperparameters (learning rate, epochs, LoRA rank) are not reported, limiting reproducibility of models trained on these datasets
- Hardware specifications for local LLaMA training/inference are not specified; outcomes are described as highly hardware-dependent
- KSP is not designed for massively parallel, faster-than-real-time, headless execution needed for large-scale RL training, so logs are finite scripted/bot runs

## Evidence

- [methods] navball-bot logs to Alpaca: [evidence span withheld — anchor status: not_found]
- [methods] Alpaca record fields: [evidence span withheld — anchor status: not_found]
- [methods] system prompt spec: [evidence span withheld — anchor status: not_found]
- [intro] RL loop interface: "we design the classic RL loop by interfacing the simulation environment (KSDPG) with a LLM, transforming the real-time observations (or state) of the mission as textual user prompts"
- [readme] KSPDG evaluation-only scope: [evidence span withheld — anchor status: not_found]
