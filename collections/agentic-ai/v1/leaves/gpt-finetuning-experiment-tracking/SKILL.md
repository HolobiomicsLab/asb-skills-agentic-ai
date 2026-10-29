---
name: gpt-finetuning-experiment-tracking
description: Use when comparing incremental LLM fine-tuning configurations against
  a baseline and you need logged training/validation losses plus a stepwise performance
  table for each fine-tuning stage.
when_to_use_negative:
- The task is pure reinforcement-learning training on a trainable simulator — KSPDG
  explicitly provides evaluation environments, NOT training environments for RL
- Only a single model configuration is evaluated with no baseline/ablation comparison,
  making the stepwise table redundant
- Evaluation requires GUI-free massively parallel headless execution; the README warns
  KSP was not designed for faster-than-real-time RL training runs
tools:
- name: Weights & Biases
  role: Experiment tracking backend logging training processes, metrics, and visualizations
    for every fine-tuning stage
- name: Hugging Face
  role: Hosting the trained models and fine-tuning datasets referenced in each tracked
    experiment
- name: KSPDG (spacegym-kspdg)
  role: Gymnasium/PettingZoo evaluation environments used to score each checkpoint
    on best/average distance and failure rate
  repo: https://github.com/mit-ll/spacegym-kspdg
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:d88a34b2aad17758da44676bdb3012edc669242caf2d8260e9b2924ee9d59597
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mit-ll/spacegym-kspdg
  tools:
  - GPT-3.5
  - Hugging Face
  - KSPDG (spacegym-kspdg)
  - Weights & Biases
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# gpt-finetuning-experiment-tracking

## Summary

Track sequential GPT-3.5 fine-tuning ablations (simple fine-tuning, +LRM tuning, +system prompt, +two train gameplays) with Weights & Biases and assemble their KSPDG performance rows (best distance, average distance, failure rate) into a validated CSV table.

## When to use

Use when comparing incremental LLM fine-tuning configurations against a baseline and you need logged training/validation losses plus a stepwise performance table for each fine-tuning stage.

## When NOT to use

- The task is pure reinforcement-learning training on a trainable simulator — KSPDG explicitly provides evaluation environments, NOT training environments for RL
- Only a single model configuration is evaluated with no baseline/ablation comparison, making the stepwise table redundant
- Evaluation requires GUI-free massively parallel headless execution; the README warns KSP was not designed for faster-than-real-time RL training runs

## Inputs

- Fine-tuned GPT-3.5 model checkpoints (simple, LRM=0.2, system prompt, two-gameplay variants)
- KSPDG evaluation environment episodes (e.g. pursuit-evasion, Lady-Bandit-Guard scenarios)
- Per-run Weights & Biases logs of training and validation loss
- Human gameplay and synthetic expert training data files

## Outputs

- gpt_finetuning_results.csv with columns method, best_distance_m, avg_distance_m, failure_rate
- Per-configuration wandb traces of training/validation loss (e.g. + LRM 0.2: 2.3e-2 / 2.8e-2)
- Ranked ablation comparison identifying the best configuration (two train gameplays: 132.09/159.78)

## How to apply

Run each fine-tuning stage through the OpenAI API, starting from simple fine-tuning (best/avg distance 263.55/265.89 m, 0.0% failure), then adding Learning Rate Multiplier 0.2 tuning (188.90/202.08), the system prompt (197.41/214.87), and two human gameplays (132.09/159.78, 0.2% failure). Track every run in Weights & Biases, logging training and validation loss per stage (e.g. LRM 0.2: 2.3e-2 train / 2.8e-2 validation; +2 human gameplays: 1.3e-4 train but validation worsened to 4.4e-1). Evaluate each checkpoint in the KSPDG differential-game environment and record Best Distance (m), Average Distance (m), and Failure Rate per configuration. Assemble the rows into a pandas DataFrame with columns method, best_distance_m, avg_distance_m, failure_rate and verify every cell matches the reported Table 5 values before saving gpt_finetuning_results.csv. Choose the winning configuration by lowest mean/final scores (here the two-gameplay variant at 132.09/159.78), while checking validation loss to catch overfitting stages.

## Related tools

- **Weights & Biases** (Experiment tracking backend logging training processes, metrics, and visualizations for every fine-tuning stage)
- **Hugging Face** (Hosting the trained models and fine-tuning datasets referenced in each tracked experiment)
- **KSPDG (spacegym-kspdg)** (Gymnasium/PettingZoo evaluation environments used to score each checkpoint on best/average distance and failure rate) — https://github.com/mit-ll/spacegym-kspdg

## Examples

```
import pandas as pd; rows=[{"method":"Simple fine-tuning","best_distance_m":263.55,"avg_distance_m":265.89,"failure_rate":"0.0%"},{"method":"+ Two train gameplays","best_distance_m":132.09,"avg_distance_m":159.78,"failure_rate":"0.2%"}]; pd.DataFrame(rows).to_csv("gpt_finetuning_results.csv", index=False)
```

## Evaluation signals

- Every CSV cell matches the reported Table 5 values exactly (e.g. best distance 263.55, 188.90, 197.41, 132.09 for the four fine-tuning stages)
- Schema check: file gpt_finetuning_results.csv has columns method, best_distance_m, avg_distance_m, failure_rate
- Failure rates fall within reported range (0.0% to 0.2%) across all four fine-tuning configurations
- wandb project for each run contains training and validation loss traces; stages where validation loss diverges (e.g. +2 human gameplays: 4.4e-1) are flagged as overfit
- Best configuration by mean/final KSPDG score reproduces the two-gameplay variant (132.09/159.78) beating the simple fine-tuning baseline (263.55/265.89)

## Limitations

- Fine-tuning hyperparameters beyond LRM (learning rate, epochs, LoRA rank) are not reported, limiting exact replication of the ablation stages
- Validation loss worsened to 4.4e-1 when two human gameplays were added despite the best task scores, so lowest loss does not equal best performance
- Results are highly hardware-dependent, especially for local LLaMA training/inference
- KSPDG environments are evaluation-only; agents cannot be directly trained on them, so scores come from held-out episodes only

## Evidence

- [results] All experiments carried out during this investigation have been tracked using Weights & Biases (wandb).: "All experiments carried out during this investigation have been tracked using Weights & Biases (wandb)."
- [results] The projects are publicly accessible and provide detailed logs, metrics, and visualizations of the training processes.: "The projects are publicly accessible and provide detailed logs, metrics, and visualizations of the training processes."
- [methods] Fine-tuning ablation rows: "Simple fine-tuning 263.55 265.89 0.0%"
- [results] Validation loss divergence for two-gameplay stage: "+ 2 human gameplays 1.3 × 10−4 4.4 × 10−1"
- [readme] KSPDG is evaluation-only: [evidence span withheld — anchor status: not_found]
