---
name: dora-low-rank-adaptation-setup
description: Use when fine-tuning an LLaMA-3-8B agent for the KSPDG differential-game
  challenge from navball-bot trajectory data (10/25/50 file variants) and the exact
  Table 1 hyperparameter set is required for replication.
when_to_use_negative:
- The task requires the prompt-engineering / few-shot GPT strategy instead of fine-tuning
  — the paper compares these two strategies separately
- 'KSPDG itself is needed as the environment: the README states KSPDG provides evaluation
  environments, NOT training environments for RL algorithms'
- Hardware cannot support Flash Attention 2 or non-quantized LLaMA-3-8B training;
  the authors note outcomes are highly hardware-dependent
edam_operation: http://edamontology.org/operation_3434
edam_topics:
- http://edamontology.org/topic_0218
tools:
- name: LLaMA
  role: Base LLaMA-3-8B model fine-tuned as the spacecraft-control agent with DoRA/LoRA
- name: Hugging Face
  role: Hosts the base LLaMA model, the fine-tuned checkpoints, and the training datasets
- name: Weights & Biases
  role: Experiment tracking of the fine-tuning runs, with public project logs, metrics,
    and visualizations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:c3715dfa83f5ccd7932ce2ace12db67fddd31c39361294950e545a647c17f9e7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Hugging Face
  - LLaMA
  - Weights & Biases
  edam_operation: http://edamontology.org/operation_3434
  edam_topics:
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# dora-low-rank-adaptation-setup

## Summary

Configures parameter-efficient fine-tuning of the LLaMA-3-8B spacecraft-control agent in the KSPDG challenge using LoRA augmented with DoRA, as reported in Table 1 of the Advances in Space Research paper (DOI 10.1016/j.asr.2025.06.034). It standardizes the exact optimizer, scheduler, LoRA/DoRA, attention, and batching settings used to produce the fine-tuned models hosted on Hugging Face.

## When to use

Use when fine-tuning an LLaMA-3-8B agent for the KSPDG differential-game challenge from navball-bot trajectory data (10/25/50 file variants) and the exact Table 1 hyperparameter set is required for replication.

## When NOT to use

- The task requires the prompt-engineering / few-shot GPT strategy instead of fine-tuning — the paper compares these two strategies separately
- KSPDG itself is needed as the environment: the README states KSPDG provides evaluation environments, NOT training environments for RL algorithms
- Hardware cannot support Flash Attention 2 or non-quantized LLaMA-3-8B training; the authors note outcomes are highly hardware-dependent

## Inputs

- LLaMA-3-8B base model identifier from Hugging Face
- Navball-bot-collected trajectory files (10/25/50 file variants) for LLaMA fine-tuning
- llama3_finetune_config.yaml target configuration file

## Outputs

- llama3_finetune_config.yaml encoding the full Table 1 hyperparameter set
- DoRA/LoRA-adapted LLaMA-3-8B checkpoint hosted to Hugging Face
- Weights & Biases run log with training metrics

## How to apply

Load the Table 1 fine-tuning specification for the LLaMA-3-8B KSPDG agent: AdamW optimizer with learning rate 1e-4 and a cosine scheduler, 3 epochs, batch size 2, and gradient accumulation 2. Define the parameter-efficient block as LoRA with r=16, alpha=8, dropout=0.05, with DoRA enabled and only the last 2 model layers set as trainable, applying no quantization. Set Flash Attention 2 as the attention implementation against the Hugging Face LLaMA-3-8B base model identifier. Point the training data paths at the navball-bot-collected trajectory files (the 10, 25, and 50 file variants) and enable Weights & Biases experiment tracking, since the paper tracks all experiments on wandb. Serialize all fields into a llama3_finetune_config.yaml so the run is exactly reproducible.

## Related tools

- **LLaMA** (Base LLaMA-3-8B model fine-tuned as the spacecraft-control agent with DoRA/LoRA)
- **Hugging Face** (Hosts the base LLaMA model, the fine-tuned checkpoints, and the training datasets)
- **Weights & Biases** (Experiment tracking of the fine-tuning runs, with public project logs, metrics, and visualizations)

## Evaluation signals

- The llama3_finetune_config.yaml parses successfully and every hyperparameter field exactly matches the Table 1 reported values (AdamW, lr 1e-4, cosine scheduler, LoRA r=16/alpha=8/dropout=0.05, DoRA enabled, Flash Attention 2, 3 epochs, last-2-layers trainable, batch size 2, gradient accumulation 2, no quantization)
- The wandb project shows a tracked run with the expected 3-epoch schedule and lr 1e-4 cosine schedule
- Only the last 2 layers are trainable; all other parameter tensors remain frozen (LoRA adapters excluded)
- The fine-tuned model moderately outperforms the navball bot baseline, as reported for the LLaMA fine-tuned models on the navball-bot datasets

## Limitations

- The paper's methods section does not report fine-tuning hyperparameters in the main text (learning rate, epochs, LoRA rank) — they must be recovered from Table 1 / the workflow
- Outcomes are highly hardware-dependent; the authors note that enhancing the model's optimization or upgrading technical resources may lead to improved results
- Details of specific experimental runs and final model versions are deferred to the appendix rather than the main results section

## Evidence

- [methods] Table 1 fine-tuning specification for the LLaMA-3-8B KSPDG agent: AdamW optimizer, learning rate 1e-4 with cosine scheduler, 3 epochs, batch size 2, and gradient accumulation 2.: [evidence span withheld — anchor status: not_found]
- [results] Fine-tuning of LLaMA models in experimental and final phases with varying hyperparameters and dataset sizes: [evidence span withheld — anchor status: not_found]
- [methods] Fine-tuning hyperparameters (learning rate, epochs, LoRA rank) not reported: [evidence span withheld — anchor status: not_found]
- [readme] KSPDG provides evaluations environments, _NOT_ training environments for reinforcement learning algorithms.: [evidence span withheld — anchor status: not_found]
