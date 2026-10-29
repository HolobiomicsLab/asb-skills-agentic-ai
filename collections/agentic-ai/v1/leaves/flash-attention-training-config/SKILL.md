---
name: flash-attention-training-config
description: Use when fine-tuning a LLaMA model as a KSPDG spacecraft-control agent
  and you need the paper's exact PEFT setup — LoRA r=16/alpha=8/dropout=0.05 with
  DoRA, Flash Attention 2, lr 1e-4, 3 epochs — serialized into a training YAML.
when_to_use_negative:
- KSPDG itself is an evaluation-only environment — per the README it 'provides evaluations
  environments, _NOT_ training environments for reinforcement learning algorithms',
  so do not use this config to train inside KSPDG
- Fine-tuning GPT-3.5 via the OpenAI API, where only the Learning Rate Multiplier
  is adjustable and LoRA/DoRA/Flash Attention 2 are not applicable
- Prompt-engineering or few-shot prompting experiments, which require no PEFT or training
  configuration
edam_operation: http://edamontology.org/operation_3714
edam_topics:
- http://edamontology.org/topic_3377
tools:
- name: LLaMA
  role: Base model family fine-tuned as the KSPDG spacecraft-control agent; configuration
    targets LLaMA-3-8B
- name: Hugging Face
  role: Source of the LLaMA-3-8B base model identifier used as the training target;
    hosts the trained models and datasets
- name: Weights & Biases
  role: Experiment tracking enabled for the fine-tuning run; detailed training logs,
    metrics and visualizations are published there
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:6e5e82ac1e3e5c4c1e79058e30fe2ed91144291dd89d66523390efd63c313e0d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Hugging Face
  - LLaMA
  - Weights & Biases
  edam_operation: http://edamontology.org/operation_3714
  edam_topics:
  - http://edamontology.org/topic_3377
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# flash-attention-training-config

## Summary

Reconstruct and serialize the exact LLaMA-3-8B fine-tuning configuration (LoRA/DoRA with Flash Attention 2) used for the KSPDG spacecraft-control agent into a reproducible YAML config matching the paper's Table 1 values.

## When to use

Use when fine-tuning a LLaMA model as a KSPDG spacecraft-control agent and you need the paper's exact PEFT setup — LoRA r=16/alpha=8/dropout=0.05 with DoRA, Flash Attention 2, lr 1e-4, 3 epochs — serialized into a training YAML.

## When NOT to use

- KSPDG itself is an evaluation-only environment — per the README it 'provides evaluations environments, _NOT_ training environments for reinforcement learning algorithms', so do not use this config to train inside KSPDG
- Fine-tuning GPT-3.5 via the OpenAI API, where only the Learning Rate Multiplier is adjustable and LoRA/DoRA/Flash Attention 2 are not applicable
- Prompt-engineering or few-shot prompting experiments, which require no PEFT or training configuration

## Inputs

- Table 1 fine-tuning specification for the LLaMA-3-8B KSPDG agent
- Navball-bot-collected trajectory dataset files (10/25/50 file variants)
- Hugging Face LLaMA-3-8B base model identifier

## Outputs

- llama3_finetune_config.yaml containing all hyperparameters (optimizer, scheduler, PEFT block, attention implementation, data paths, wandb tracking)

## How to apply

Load the Table 1 specification for the LLaMA-3-8B KSPDG agent: AdamW optimizer, learning rate 1e-4 with cosine scheduler, 3 epochs, batch size 2, gradient accumulation 2, and no quantization. Define the parameter-efficient fine-tuning block using LoRA with r=16, alpha=8, dropout=0.05, DoRA enabled, and only the last 2 model layers set as trainable. Set Flash Attention 2 as the attention implementation and attach the Hugging Face LLaMA-3-8B base model identifier as the training target. Point the training-data paths at the navball-bot-collected trajectory files (10/25/50 file variants) and enable Weights & Biases experiment tracking for the run, since the paper tracked all experiments there. Finally serialize every field into llama3_finetune_config.yaml so the config can be diffed field-by-field against Table 1.

## Related tools

- **LLaMA** (Base model family fine-tuned as the KSPDG spacecraft-control agent; configuration targets LLaMA-3-8B)
- **Hugging Face** (Source of the LLaMA-3-8B base model identifier used as the training target; hosts the trained models and datasets)
- **Weights & Biases** (Experiment tracking enabled for the fine-tuning run; detailed training logs, metrics and visualizations are published there)

## Examples

```
import yaml; yaml.safe_dump({'optimizer':'AdamW','lr':1e-4,'scheduler':'cosine','lora':{'r':16,'alpha':8,'dropout':0.05,'dora':True},'flash_attention_2':True,'epochs':3,'batch_size':2}, open('llama3_finetune_config.yaml','w'))
```

## Evaluation signals

- The generated YAML parses successfully as valid configuration
- Every hyperparameter field exactly matches Table 1: AdamW, lr 1e-4, cosine scheduler, LoRA r=16/alpha=8/dropout=0.05, DoRA enabled, Flash Attention 2, 3 epochs, last-2-layers trainable, batch size 2, gradient accumulation 2, no quantization
- Training-data paths resolve to the navball-bot-collected 10/25/50-file dataset variants
- A Weights & Biases run is initialized and logging metrics for the training process
- The config drives a run whose fine-tuned model performance can be compared against the navball bot baseline (30.52/51.53 for 10 files, 13.54/29.44 for 25, 11.86/29.76 for 50)

## Limitations

- Outcomes are highly hardware-dependent: the paper notes that 'enhancing the model's optimization or upgrading the technical resources may lead to improved results, as these outcomes are highly hardware-dependent'
- Some paper sections do not report the fine-tuning hyperparameters (a flagged missing signal: 'Fine-tuning hyperparameters (learning rate, epochs, LoRA rank) not reported'), so Table 1 is the authoritative source
- The experimental LLaMA models were trained 'during a phase where we experimented with variations of hyperparameters', so not all published checkpoints share this exact configuration

## Evidence

- [methods] Dataset provenance and evaluation baseline: [evidence span withheld — anchor status: not_found]
- [results] Hyperparameter variation across experimental models: [evidence span withheld — anchor status: not_found]
- [abstract] Weights & Biases tracking of all training runs: "experiment tracking and detailed results can be reviewed on Weights & Biases"
- [intro] Fine-tuning is one of the compared strategies alongside prompt engineering: "Our approach leverages prompt engineering, few-shot prompting, and fine-tuning techniques"
- [results] Trained models and datasets are published on Hugging Face: "the models developed and used in this research are hosted on Hugging Face"
