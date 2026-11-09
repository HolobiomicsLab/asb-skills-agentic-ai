---
name: peft-hyperparameter-specification
description: Use when a LLaMA-style LLM must be fine-tuned for an agent/control task
  and the full training configuration (optimizer, scheduler, PEFT/DoRA block, attention
  implementation, epochs, batch size) must be specified or reproduced.
when_to_use_negative:
- When fine-tuning GPT via the OpenAI API, where only the Learning Rate Multiplier
  (e.g. default LRM 2 vs LRM 0.2) is adjustable — not LoRA/DoRA parameters
- When the goal is prompt-engineering or few-shot prompting of a frozen model rather
  than parameter-efficient fine-tuning
- When KSPDG is used only as an evaluation environment — the README states it provides
  evaluation environments, NOT training environments for RL
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: LLaMA
  role: Base LLM architecture (LLaMA-3-8B) being fine-tuned as the spacecraft-control
    agent
- name: Hugging Face
  role: Hosts the base LLaMA-3-8B model identifier and the resulting fine-tuned models
    and datasets
- name: Weights & Biases
  role: Experiment tracking of all fine-tuning runs so hyperparameter variants can
    be compared
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:933cb87547775ec7ae3c30107e850b967e9c2d838aa7ce6e3ee1265cce720cbd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Hugging Face
  - LLaMA
  - Weights & Biases
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# peft-hyperparameter-specification

## Summary

Specify the complete parameter-efficient fine-tuning (PEFT) configuration for LLaMA-3-8B agents used as spacecraft operators in the KSPDG challenge: AdamW with lr 1e-4 and cosine scheduler, LoRA (r=16, alpha=8, dropout=0.05) with DoRA enabled, Flash Attention 2, 3 epochs, last-2-layers trainable, batch size 2, gradient accumulation 2, no quantization. This makes LLM-based agent training reproducible and comparable against the published baselines.

## When to use

Use when a LLaMA-style LLM must be fine-tuned for an agent/control task and the full training configuration (optimizer, scheduler, PEFT/DoRA block, attention implementation, epochs, batch size) must be specified or reproduced.

## When NOT to use

- When fine-tuning GPT via the OpenAI API, where only the Learning Rate Multiplier (e.g. default LRM 2 vs LRM 0.2) is adjustable — not LoRA/DoRA parameters
- When the goal is prompt-engineering or few-shot prompting of a frozen model rather than parameter-efficient fine-tuning
- When KSPDG is used only as an evaluation environment — the README states it provides evaluation environments, NOT training environments for RL

## Inputs

- Base model identifier (Hugging Face LLaMA-3-8B)
- Fine-tuning dataset: navball-bot-collected trajectory files (10/25/50 file variants)
- Table 1 fine-tuning specification (hyperparameter values to encode)

## Outputs

- llama3_finetune_config.yaml with all hyperparameters matching Table 1
- Weights & Biases tracked training run with logged metrics
- Fine-tuned LLaMA checkpoint hosted on Hugging Face

## How to apply

Define the optimization block first: AdamW optimizer with learning rate 1e-4 and a cosine scheduler over 3 epochs, batch size 2 and gradient accumulation 2. Then define the PEFT block: LoRA with r=16, alpha=8, dropout=0.05, DoRA enabled, only the last 2 model layers set as trainable, and no quantization applied. Set Flash Attention 2 as the attention implementation and attach the Hugging Face LLaMA-3-8B base model identifier as the training target. Point the training data paths at the navball-bot-collected trajectory files (10/25/50 file variants) and enable Weights & Biases tracking so every run is logged for comparison. Serialize all fields into llama3_finetune_config.yaml and validate that the YAML parses and every hyperparameter exactly matches the Table 1 reported values.

## Related tools

- **LLaMA** (Base LLM architecture (LLaMA-3-8B) being fine-tuned as the spacecraft-control agent)
- **Hugging Face** (Hosts the base LLaMA-3-8B model identifier and the resulting fine-tuned models and datasets)
- **Weights & Biases** (Experiment tracking of all fine-tuning runs so hyperparameter variants can be compared)

## Examples

```
from peft import LoraConfig; cfg = LoraConfig(r=16, lora_alpha=8, lora_dropout=0.05, use_dora=True)
```

## Evaluation signals

- The YAML configuration parses successfully and every hyperparameter field exactly matches the Table 1 reported values (AdamW, lr 1e-4, cosine scheduler, LoRA r=16/alpha=8/dropout=0.05, DoRA enabled, Flash Attention 2, 3 epochs, last-2-layers trainable, batch size 2, gradient accumulation 2, no quantization)
- Weights & Biases run logs exist and record the training processes, metrics, and visualizations for the run
- Only the last 2 model layers plus LoRA adapters are trainable — trainable parameter count is a small fraction of the full 8B model
- Fine-tuned LLaMA model and its training dataset are uploaded and resolvable on Hugging Face repositories
- Performance of the fine-tuned model is at or above the navball bot baseline (LLaMA moderately outperforms the navball bot) on KSPDG metrics

## Limitations

- Outcomes are highly hardware-dependent; enhancing the model's optimization or upgrading technical resources may lead to improved results, but hardware specifications for local LLaMA training/inference are not specified
- Fine-tuning hyperparameters (learning rate, epochs, LoRA rank) are reported only for the final Table 1 configuration; experimental models were trained during a phase with varying hyperparameters and dataset sizes
- KSPDG provides evaluation environments, NOT training environments — training data must be collected separately (e.g. via the navball bot)

## Evidence

- [intro] In this work, we leverage GPT-3.5 and LLaMA models to develop an intelligent agent capable of controlling a spacecraft in real time.: "In this work, we leverage GPT-3.5 and LLaMA models to develop an intelligent agent capable of controlling a spacecraft in real time."
- [methods] all dataset files were collected using the navball bot: [evidence span withheld — anchor status: not_found]
- [results] All experiments carried out during this investigation have been tracked using Weights & Biases (wandb).: "All experiments carried out during this investigation have been tracked using Weights & Biases (wandb)."
- [results] the experimental models, which were trained during a phase where we experimented with variations of hyperparameters: [evidence span withheld — anchor status: not_found]
- [readme] KSPDG provides evaluations environments, _NOT_ training environments for reinforcement learning algorithms.: [evidence span withheld — anchor status: not_found]
