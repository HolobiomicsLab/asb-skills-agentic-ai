---
name: llama-lora-finetuning-configuration
description: Use when reproducing or adapting the paper's LLaMA-3-8B fine-tuning setup
  for the KSPDG pursuit-evasion agent, i.e. when a PEFT (LoRA/DoRA) configuration
  must match Table 1's reported hyperparameters exactly.
when_to_use_negative:
- Fine-tuning GPT via the OpenAI API — that path uses the Learning Rate Multiplier
  (LRM) hyperparameter, not a LoRA/DoRA configuration.
- Training agents directly on KSPDG environments — the README states KSPDG 'provides
  evaluations environments, NOT training environments for reinforcement learning algorithms'.
- Workloads requiring large-scale parallel RL training — KSP/KSPDG was not designed
  for massively parallel, faster-than-real-time, headless execution.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: LLaMA
  role: Base LLM (LLaMA-3-8B) fine-tuned with LoRA/DoRA to act as the spacecraft-control
    agent
- name: Hugging Face
  role: Source of the LLaMA-3-8B base model identifier and host of the trained models
    and 10/25/50-file fine-tuning datasets
- name: Weights & Biases
  role: Experiment tracking of the fine-tuning runs, with publicly accessible logs,
    metrics and visualizations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:1d8e5de3bb8d5f62b707d4b7790223cfed49b3d3207171b42f7d5e045ca240fc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Hugging Face
  - LLaMA
  - Weights & Biases
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# llama-lora-finetuning-configuration

## Summary

Defines the LoRA/DoRA fine-tuning configuration used to adapt LLaMA-3-8B into a spacecraft-control agent for the KSPDG challenge: AdamW with lr 1e-4 and cosine scheduler, LoRA r=16/alpha=8/dropout=0.05, Flash Attention 2, DoRA, last 2 trainable layers, 3 epochs, batch size 2. Reproducing these validated settings enables an LLM agent to operate a spacecraft from raw telemetry text.

## When to use

Use when reproducing or adapting the paper's LLaMA-3-8B fine-tuning setup for the KSPDG pursuit-evasion agent, i.e. when a PEFT (LoRA/DoRA) configuration must match Table 1's reported hyperparameters exactly.

## When NOT to use

- Fine-tuning GPT via the OpenAI API — that path uses the Learning Rate Multiplier (LRM) hyperparameter, not a LoRA/DoRA configuration.
- Training agents directly on KSPDG environments — the README states KSPDG 'provides evaluations environments, NOT training environments for reinforcement learning algorithms'.
- Workloads requiring large-scale parallel RL training — KSP/KSPDG was not designed for massively parallel, faster-than-real-time, headless execution.

## Inputs

- LLaMA-3-8B base model identifier (Hugging Face)
- Navball-bot-collected trajectory files (10/25/50 file variants) used as the LLaMA fine-tuning dataset
- Table 1 fine-tuning hyperparameter specification (optimizer, scheduler, LoRA block, epochs, batch size)

## Outputs

- llama3_finetune_config.yaml containing the validated hyperparameter set
- Fine-tuned LLaMA-3-8B checkpoints (10/25/50-file variants) hosted on Hugging Face
- Weights & Biases run logs with training metrics and visualizations

## How to apply

Serialize the fine-tuning specification into llama3_finetune_config.yaml: set the optimizer to AdamW with learning rate 1e-4 and a cosine scheduler over 3 epochs with batch size 2 and gradient accumulation 2. Define the parameter-efficient block as LoRA with r=16, alpha=8, dropout=0.05 and DoRA enabled, marking only the last 2 model layers trainable and applying no quantization. Set Flash Attention 2 as the attention implementation and attach the Hugging Face LLaMA-3-8B base model identifier as the training target. Point the training data paths at the navball-bot-collected trajectory files (the 10/25/50 file variants published on Hugging Face) and enable Weights & Biases tracking so metrics and visualizations are logged in the publicly accessible project. Validate by parsing the YAML and checking every field against the Table 1 reported values before launching the run.

## Related tools

- **LLaMA** (Base LLM (LLaMA-3-8B) fine-tuned with LoRA/DoRA to act as the spacecraft-control agent)
- **Hugging Face** (Source of the LLaMA-3-8B base model identifier and host of the trained models and 10/25/50-file fine-tuning datasets)
- **Weights & Biases** (Experiment tracking of the fine-tuning runs, with publicly accessible logs, metrics and visualizations)

## Examples

```
from peft import LoraConfig; cfg = LoraConfig(r=16, lora_alpha=8, lora_dropout=0.05, use_dora=True)  # pair with lr=1e-4 cosine, 3 epochs, bs=2, flash_attention_2
```

## Evaluation signals

- llama3_finetune_config.yaml parses successfully and every hyperparameter field exactly matches Table 1: AdamW, lr 1e-4, cosine scheduler, LoRA r=16/alpha=8/dropout=0.05, DoRA enabled, Flash Attention 2, 3 epochs, last-2-layers trainable, batch size 2, gradient accumulation 2, no quantization.
- The run appears in the publicly accessible Weights & Biases project with logged training metrics.
- Resulting fine-tuned model checkpoints and the 10/25/50-file datasets are uploaded to their Hugging Face repositories.
- The fine-tuned LLaMA agent outperforms the Navball (bot) baseline on the KSPDG evaluation metrics (e.g. fine-tune 50 files: 11.86 vs. Navball bot 34.34).
- No quantization is applied — the config must not contain any quantization block.

## Limitations

- Hardware specifications for local LLaMA training/inference are not specified; outcomes are 'highly hardware-dependent' per the article.
- The experimental models were 'trained during a phase where we experimented with variations of hyperparameters', so Table 1 values correspond to the final configuration and intermediate runs may differ.
- Results are tied to navball-bot-collected data; the article notes the fine-tuned LLaMA only 'moderately outperforms the navball bot's performance'.

## Evidence

- [results] Fine-tuning hyperparameters from finding: [evidence span withheld — anchor status: not_found]
- [methods] Dataset source for fine-tuning: [evidence span withheld — anchor status: not_found]
- [results] Experiment tracking: "All experiments carried out during this investigation have been tracked using Weights & Biases (wandb)."
- [results] Model hosting: "the models developed and used in this research are hosted on Hugging Face"
- [readme] KSPDG is evaluation-only: [evidence span withheld — anchor status: not_found]
