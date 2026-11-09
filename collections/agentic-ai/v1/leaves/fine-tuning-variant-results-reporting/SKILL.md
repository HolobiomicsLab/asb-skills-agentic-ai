---
name: fine-tuning-variant-results-reporting
description: Use when reproducing or validating a methods-section results table of
  fine-tuned model variants versus baseline agents, where each row reports /average
  closest-approach distance (m) and failure rate in the KSPDG scenario E3.
when_to_use_negative:
- Input is raw training telemetry or wandb logs rather than an already-reported results
  table — this skill tabulates reported metrics, it does not run evaluations
- 'The task requires reproducing GPT training-loss/validation-loss figures (e.g. LRM
  0.2: 2.3e-2 train, 2.8e-2 validation) — those belong to a separate fine-tuning-hyperparameter
  reporting skill'
- Evaluation must be re-run from scratch in KSPDG — the paper's environment is evaluation-only
  and not designed for large-scale RL training
edam_operation: http://edamontology.org/operation_2409
tools:
- name: Weights & Biases
  role: Experiment tracking source whose public project logs provide detailed training-process
    metrics to cross-check the tabulated variant results
- name: Hugging Face
  role: Hosts the fine-tuned LLaMA model variants (10 files, 25/50 files, 10 files
    sliding window, 35 files quantized) referenced by the table rows
- name: spacegym-kspdg
  role: Defines the KSPDG differential-game evaluation environments (pursuit-evasion,
    Lady-Bandit-Guard, sun-blocking) in which the reported closest-approach metrics
    were measured
  repo: https://github.com/mit-ll/spacegym-kspdg/tree main
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:d47ea9ec0fed6b9bd65880129e9c164d2023092280f07548fd6da2ec1a93916b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Hugging Face
  - Weights & Biases
  - spacegym-kspdg
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# fine-tuning-variant-results-reporting

## Summary

Reconstructs and validates the benchmark results table comparing LLaMA agents fine-tuned on navball-bot-collected logs (10/25/50 files, sliding window) against the navball bot and Space Gym baselines (Naive, PPO, iLQGames, Lambert-MPC) in the KSPDG pursuit-evasion scenario, reporting closest-approach distances and failure rates.

## When to use

Use when reproducing or validating a methods-section results table of fine-tuned model variants versus baseline agents, where each row reports best/average closest-approach distance (m) and failure rate in the KSPDG scenario E3.

## When NOT to use

- Input is raw training telemetry or wandb logs rather than an already-reported results table — this skill tabulates reported metrics, it does not run evaluations
- The task requires reproducing GPT training-loss/validation-loss figures (e.g. LRM 0.2: 2.3e-2 train, 2.8e-2 validation) — those belong to a separate fine-tuning-hyperparameter reporting skill
- Evaluation must be re-run from scratch in KSPDG — the paper's environment is evaluation-only and not designed for large-scale RL training

## Inputs

- Methods-section results table for LLaMA fine-tuning variants in scenario E3 (best/average closest-approach distance in meters, failure rate %)
- Comparison rows for baseline agents: navball bot, baseline LLaMA, Space Gym agents (Naive, PPO, iLQGames, Lambert-MPC)
- Optional GPT fine-tuning ablation rows (LRM variants, system prompt, human gameplay additions)
- Weights & Biases public project logs and Hugging Face model repositories for cross-checking run provenance

## Outputs

- Single CSV table with columns Method, Best Distance (m), Average Distance (m), Failure Rate containing all 9+ rows
- Row-per-variant validation notes confirming each cell matches the reported metric (e.g. fine-tune 50 files best distance = 11.86 m; navball bot = 34.34/36.43 m)

## How to apply

Read the methods-section results table for the LLaMA experiments and extract best and average closest-approach distances plus failure rate for each fine-tuning variant: fine-tune 10 files (30.52/51.53 m, 0.00%), 25 files (13.54/29.44 m, 0.00%), 50 files (11.86/29.76 m, 0.00%), and 10 files with sliding window win=3 (23.08/40.03 m, 0.00%). Then extract comparison rows for the baseline LLaMA (52.69/140.68 m, 9.09%), the navball bot training-data generator (34.34/36.43 m), and the Space Gym benchmark agents Naive (225.0/225.0 m), PPO (>1643/2346 m), iLQGames (>53.31/60.99 m), and Lambert-MPC (>18.24/47.94 m). Assemble all rows into a single CSV with columns Method, Best Distance (m), Average Distance (m), Failure Rate, preserving the '>' threshold notation used for PPO, iLQGames, and Lambert-MPC where the paper only reports a bound. Optionally include the GPT prompt-engineering ablation rows (simple fine-tuning 263.55/265.89 0.0%; +hyperparam tuning 188.90/202.08 0.1%; +system prompt 197.41/214.87 0.0%; +two train gameplays 132.09/159.78 0.2%).

## Related tools

- **Weights & Biases** (Experiment tracking source whose public project logs provide detailed training-process metrics to cross-check the tabulated variant results)
- **Hugging Face** (Hosts the fine-tuned LLaMA model variants (10 files, 25/50 files, 10 files sliding window, 35 files quantized) referenced by the table rows)
- **spacegym-kspdg** (Defines the KSPDG differential-game evaluation environments (pursuit-evasion, Lady-Bandit-Guard, sun-blocking) in which the reported closest-approach metrics were measured) — https://github.com/mit-ll/spacegym-kspdg/tree main

## Examples

```
git clone git@github.com:mit-ll/spacegym-kspdg.git
```

## Evaluation signals

- All 9 rows present: 4 LLaMA fine-tuning variants, baseline LLaMA, navball bot, and Space Gym agents Naive, PPO, iLQGames, Lambert-MPC
- Spot-check cells: fine-tune 50 files best distance = 11.86 m; navball bot = 34.34/36.43 m; fine-tune 10 files win=3 = 23.08/40.03 m, 0.00%
- '>' threshold notation preserved for PPO (>1643/2346 m), iLQGames (>53.31/60.99 m), and Lambert-MPC (>18.24/47.94 m) where the paper reports bounds
- Failure rates for LLaMA fine-tuned variants are 0.00% while baseline LLaMA is 9.09% — a missing 9.09% indicates the wrong row was copied
- Column schema is exactly Method, Best Distance (m), Average Distance (m), Failure Rate with no extra or missing columns

## Limitations

- Lambert-MPC best distance is only reported as a threshold (>18.24 m), so exact values cannot be reconstructed for that row
- Fine-tuning hyperparameters (learning rate, epochs, LoRA rank) are not reported, limiting reproducibility of the variants being tabulated
- Hardware specifications for local LLaMA training/inference are not specified, and outcomes are described as highly hardware-dependent
- KSPDG provides evaluation environments, not training environments, so the table's numbers cannot be regenerated by RL training within the library itself

## Evidence

- [methods] fine-tune 10 files 30.52 51.53 0.00%: "fine-tune 10 files 30.52 51.53 0.00%"
- [methods] fine-tune 50 files best/average and navball bot rows: "Navball (bot) 34.34 36.43 -"
- [methods] Lambert-MPC baseline threshold notation: "Lambert-MPC >18.24 47.94 -"
- [methods] datasets collected with the navball bot: "It is noteworthy that all dataset files were collected using the navball bot"
- [results] experiment tracking provenance: "All experiments carried out during this investigation have been tracked using Weights & Biases (wandb)."
- [readme] evaluation-not-training design principle: [evidence span withheld — anchor status: not_found]
