---
name: fine-tuning-data-format-validation
description: Use when converting recorded human or navball-bot KSPDG gameplay logs
  into a fine-tuning dataset and you must verify each timestep line parses as JSON
  with well-formed role fields and a valid perform_action thrust label.
when_to_use_negative:
- Input is the raw KSPDG evaluation environment output or mission save files (ksp_files/saves/missions)
  rather than recorded gameplay logs — KSPDG provides evaluation environments, not
  training data
- The target is reinforcement-learning training in KSPDG itself, which the upstream
  library explicitly does not support (evaluation, not training)
- The logs come from a non-navball agent whose action labels do not match the vessel-frame
  throttle axes (x right, y toward target, z up) assumed by the prompt templates
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: KSPDG
  role: Provides the pursuit-evasion environment and observation schema from which
    gameplay logs and thrust actions are recorded
  repo: https://github.com/ARCLab-MIT/kspdg
- name: Hugging Face
  role: Hosts the validated fine-tuning datasets and resulting models for reuse
- name: GPT-3.5
  role: Target model fine-tuned on the validated JSONL records via the OpenAI API
- name: LLaMA
  role: Target model family fine-tuned on the 10/25/50-file dataset variants produced
    by this workflow
- name: openai/tiktoken
  role: BPE tokeniser used to count/verify token budgets of the serialized prompts
    against OpenAI model limits
  repo: https://github.com/openai/tiktoken
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:f22204cea4b5f728a2a36aad4f4fad83607d29defbca91dffb925c9899cfb430
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ARCLab-MIT/kspdg
  tools:
  - GPT-3.5
  - Hugging Face
  - KSPDG
  - LLaMA
  - openai/tiktoken
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# fine-tuning-data-format-validation

## Summary

Validates that gameplay-logged observation/thrust-action pairs were correctly serialized into OpenAI-style JSONL fine-tuning records (system/user/assistant roles) for imitating human KSPDG spacecraft maneuvering, before the file is submitted to fine-tuning.

## When to use

Use when converting recorded human or navball-bot KSPDG gameplay logs into a fine-tuning dataset and you must verify each timestep line parses as JSON with well-formed role fields and a valid perform_action thrust label.

## When NOT to use

- Input is the raw KSPDG evaluation environment output or mission save files (ksp_files/saves/missions) rather than recorded gameplay logs — KSPDG provides evaluation environments, not training data
- The target is reinforcement-learning training in KSPDG itself, which the upstream library explicitly does not support (evaluation, not training)
- The logs come from a non-navball agent whose action labels do not match the vessel-frame throttle axes (x right, y toward target, z up) assumed by the prompt templates

## Inputs

- Recorded human gameplay episode logs from the kspdg agents_data folder
- Navball-bot gameplay logs (all dataset files were collected using the navball bot)
- Observation dictionaries per timestep (relative positions, velocities, fuel, prograde) sampled every 0.5 s
- Discrete thrust action labels in the vessel reference frame (ft, rt, dt)

## Outputs

- JSONL fine-tuning file (fine_tuning_data) with one line per timestep, each containing 'system', 'user', and 'assistant' role fields
- Assistant responses embedding perform_action(ft, rt, dt) thrust calls delimited with HUMAN/ASSISTANT keywords
- Dataset suitable for OpenAI GPT-3.5 and LLaMA fine-tuning (10/25/50 file variants hosted on Hugging Face)

## How to apply

Load the recorded gameplay episode logs (human gameplay and navball-bot data) from the kspdg agents_data folder and iterate over timesteps sampled every 0.5 seconds. For each timestep, render the observation dictionary (relative positions, velocities, fuel, prograde) into the user prompt template 'Given these observations ..., what is the best throttle to capture evader?' with step-by-step reasoning requested, and prepend the fixed system prompt describing the autonomous pursuit agent and vessel-frame throttle axes (x right, y toward target, z up, max throttle 1). Pair each observation with its thrust action label relative to the spacecraft reference frame, formatted as an assistant response ending in a perform_action(ft, rt, dt) call and delimited with HUMAN/ASSISTANT keywords. Serialize each timestep as one JSON line containing 'system', 'user', and 'assistant' role fields, saved as fine_tuning_data. Validation requires that every output line parses as JSON, contains all three role fields, and embeds a well-formed perform_action thrust label; the resulting dataset (collected with the navball bot) is used to fine-tune LLaMA and GPT-3.5 models whose success is later measured by reduced pursuit metrics versus the navball bot baseline.

## Related tools

- **KSPDG** (Provides the pursuit-evasion environment and observation schema from which gameplay logs and thrust actions are recorded) — https://github.com/ARCLab-MIT/kspdg
- **Hugging Face** (Hosts the validated fine-tuning datasets and resulting models for reuse)
- **GPT-3.5** (Target model fine-tuned on the validated JSONL records via the OpenAI API)
- **LLaMA** (Target model family fine-tuned on the 10/25/50-file dataset variants produced by this workflow)
- **openai/tiktoken** (BPE tokeniser used to count/verify token budgets of the serialized prompts against OpenAI model limits) — https://github.com/openai/tiktoken

## Evaluation signals

- Every output line parses as valid JSON
- Each line contains all three role fields: 'system', 'user', and 'assistant'
- Each assistant field embeds a well-formed perform_action(ft, rt, dt) thrust label
- Timesteps are sampled at the recorded 0.5 s cadence and observations match the logged relative-position/velocity/fuel/prograde values
- Downstream fine-tuning achieves decreasing loss (e.g. GPT LRM 0.2 reached training loss 2.3e-2 / validation 2.8e-2) and improved pursuit metrics vs. the Navball bot baseline (34.34)

## Limitations

- All dataset files were collected using the navball bot, so the fine-tuned model at best imitates navball-bot-like behavior (though it moderately outperformed it)
- Adding only 2 human gameplays to GPT fine-tuning caused validation loss to worsen (4.4e-1) despite training loss of 1.3e-4, indicating overfitting risk on small human-data additions
- KSP game engine is not designed for massively parallel, faster-than-real-time, headless execution, limiting data-generation throughput
- Fine-tuning hyperparameters (learning rate, epochs, LoRA rank) are not reported in the paper, reducing reproducibility of the data-to-model pipeline

## Evidence

- [methods] The fine-tuning dataset consists of observations and thrust actions recorded every 0.5 seconds during gameplay. Thrust actions are logged as discrete labels relative to the spacecraft's reference frame: [evidence span withheld — anchor status: not_found]
- [methods] It is noteworthy that all dataset files were collected using the navball bot, and the LLaMA fine-tuned model moderately outperforms the navball bot's performance.: [evidence span withheld — anchor status: not_found]
- [readme] KSPDG provides evaluations environments, _NOT_ training environments for reinforcement learning algorithms.: [evidence span withheld — anchor status: not_found]
- [results] + 2 human gameplays 1.3 × 10−4 4.4 × 10−1: "+ 2 human gameplays 1.3 × 10−4 4.4 × 10−1"
- [results] the models developed and used in this research are hosted on Hugging Face: "the models developed and used in this research are hosted on Hugging Face"
