---
name: closest-approach-distance-tabulation
description: Use when reconstructing the E3 pursuit-evasion benchmark table for LLaMA
  fine-tuned agents vs navball/Lambert-MPC/iLQGames/PPO baselines, or when verifying
  reported /average closest-approach distances and failure rates in CSV form.
when_to_use_negative:
- When the goal is to train or fine-tune the LLaMA agents themselves — KSPDG explicitly
  provides evaluation environments, NOT training environments for RL algorithms
- When raw telemetry episodes or wandb experiment logs are available directly — this
  skill reconstructs a published table from reported numbers, not from raw simulation
  output
- When the comparison scenario is not E3 closest-approach pursuit-evasion (e.g., Lady-Bandit-Guard
  LBG1 or sun-blocking SB1 scenarios), since the metric rows differ
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Space Gym
  role: Provides the benchmark agents (Naive, PPO, iLQGames, Lambert-MPC) whose closest-approach
    rows populate the comparison table
  repo: https://github.com/mit-ll/spacegym-kspdg
- name: KSPDG
  role: Defines the pursuit-evaluation scenario E3 in which best/average closest-approach
    distance and failure rate are measured
  repo: https://github.com/mit-ll/spacegym-kspdg
- name: Weights & Biases (wandb)
  role: Publicly accessible experiment tracking used to cross-check the training runs
    behind each fine-tuning variant row
- name: Hugging Face
  role: Hosts the fine-tuned LLaMA model checkpoints (10 files, 25 files, 50 files,
    10 files sliding window, quantized 35 files) referenced by each table row
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:d65d4fc6ffd7d88bf51d9c0d3b672b194de9b843cd4e1910c9576efdaed42b73
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mit-ll/spacegym-kspdg
  tools:
  - Hugging Face
  - KSPDG
  - Space Gym
  - Weights & Biases (wandb)
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# closest-approach-distance-tabulation

## Summary

Reconstructs and tabulates closest-approach performance metrics (best distance, average distance, failure rate) from the KSPDG scenario E3 methods-section results table, covering LLaMA fine-tuning variants (10/25/50 files, sliding window) against navball, PPO, iLQGames, and Lambert-MPC baselines. It exists so that LLM-agent spacecraft control results can be reassembled into a machine-readable benchmark CSV for comparison.

## When to use

Use when reconstructing the E3 pursuit-evasion benchmark table for LLaMA fine-tuned agents vs navball/Lambert-MPC/iLQGames/PPO baselines, or when verifying reported best/average closest-approach distances and failure rates in CSV form.

## When NOT to use

- When the goal is to train or fine-tune the LLaMA agents themselves — KSPDG explicitly provides evaluation environments, NOT training environments for RL algorithms
- When raw telemetry episodes or wandb experiment logs are available directly — this skill reconstructs a published table from reported numbers, not from raw simulation output
- When the comparison scenario is not E3 closest-approach pursuit-evasion (e.g., Lady-Bandit-Guard LBG1 or sun-blocking SB1 scenarios), since the metric rows differ

## Inputs

- Methods-section results table for KSPDG scenario E3 (closest-approach distances in meters and failure rates per method)
- List of LLaMA fine-tuning variants: 10 files, 25 files, 50 files, 10 files with sliding window win=3
- Baseline agent rows: LLaMA (no fine-tuning), navball bot, Space Gym agents (Naive, PPO, iLQGames, Lambert-MPC)

## Outputs

- CSV table with columns Method, Best Distance (m), Average Distance (m), Failure Rate and 9 rows preserving '>' threshold notation
- Verified cell-by-cell mapping of each reconstructed value to the reported source metric (e.g., fine-tune 50 files best distance = 11.86 m)

## How to apply

Read the methods-section results table for the KSPDG scenario E3 (pursuit-evasion closest approach) and extract, for each LLaMA fine-tuning variant, the best and average closest-approach distances in meters and the failure rate: fine-tune 10 files (30.52/51.53 m, 0.00%), 25 files (13.54/29.44 m, 0.00%), 50 files (11.86/29.76 m, 0.00%), and 10 files with sliding window win=3 (23.08/40.03 m, 0.00%). Add the comparison rows: baseline LLaMA (52.69/140.68 m, 9.09%), navball bot that generated the training data (34.34/36.43 m), and Space Gym benchmark agents Naive (225.0/225.0 m), PPO (>1643/2346 m), iLQGames (>53.31/60.99 m), and Lambert-MPC (>18.24/47.94 m). Assemble all 9 rows into a single CSV with columns Method, Best Distance (m), Average Distance (m), Failure Rate, preserving the '>' threshold notation for PPO, iLQGames, and Lambert-MPC where the paper reports only lower bounds. The decision point this table supports: all dataset files were collected with the navball bot, so the fine-tuned LLaMA (best 11.86 m at 50 files) is compared directly against its own data-generating bot (34.34 m) to quantify moderate improvement.

## Related tools

- **Space Gym** (Provides the benchmark agents (Naive, PPO, iLQGames, Lambert-MPC) whose closest-approach rows populate the comparison table) — https://github.com/mit-ll/spacegym-kspdg
- **KSPDG** (Defines the pursuit-evaluation scenario E3 in which best/average closest-approach distance and failure rate are measured) — https://github.com/mit-ll/spacegym-kspdg
- **Weights & Biases (wandb)** (Publicly accessible experiment tracking used to cross-check the training runs behind each fine-tuning variant row)
- **Hugging Face** (Hosts the fine-tuned LLaMA model checkpoints (10 files, 25 files, 50 files, 10 files sliding window, quantized 35 files) referenced by each table row)

## Examples

```
python -c "import pandas as pd; df = pd.read_csv('e3_closest_approach.csv', keep_default_na=False); print(df[df['Method'].str.contains('50 files')]); assert df.shape[0] == 9"
```

## Evaluation signals

- All 9 rows are present in the reconstructed CSV (4 fine-tuned variants + baseline LLaMA + navball bot + 4 Space Gym benchmark agents)
- Spot-check invariants: fine-tune 50 files best distance = 11.86 m, fine-tune 10 files = 30.52/51.53 m at 0.00% failure, navball bot = 34.34/36.43 m
- '>' threshold notation is preserved for PPO (>1643/2346 m), iLQGames (>53.31/60.99 m), and Lambert-MPC (>18.24/47.94 m) rather than being silently converted to numbers
- Failure-rate column shows 0.00% for all four fine-tuned variants and 9.09% for the baseline LLaMA
- The ordering is consistent with the paper's claim: fine-tuned LLaMA (11.86 m best at 50 files) moderately outperforms the navball bot (34.34 m) that generated its training data

## Limitations

- Values for PPO, iLQGames, and Lambert-MPC are lower-bound thresholds (reported with '>' notation), not exact distances
- Failure rate for the navball bot row is reported as '-' and cannot be tabulated numerically
- Outcomes are highly hardware-dependent: fine-tuning hyperparameters (learning rate, epochs, LoRA rank) and local training/inference hardware are not specified in the paper
- KSPDG scenarios are non-blocking and evaluation-only; the tabulated numbers reflect a specific scripted adversary and cannot be trained against directly

## Evidence

- [methods] fine-tune 50 files 11.86 29.76 0.00%: "fine-tune 50 files 11.86 29.76 0.00%"
- [methods] Navball (bot) 34.34 36.43 -: "Navball (bot) 34.34 36.43 -"
- [methods] Lambert-MPC >18.24 47.94 -: "Lambert-MPC >18.24 47.94 -"
- [methods] All dataset files were collected using the navball bot: "It is noteworthy that all dataset files were collected using the navball bot"
- [readme] KSPDG provides evaluations environments, NOT training environments: [evidence span withheld — anchor status: not_found]
