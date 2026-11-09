---
name: spacecraft-agent-benchmark-comparison
description: Use when reproducing or extending the KSPDG scenario E3 pursuit-evasion
  benchmark table that compares LLaMA fine-tuned spacecraft agents (10/25/50 training
  files, sliding window) against the navball bot and Space Gym baselines on closest-approach
  distance and failure rate.
when_to_use_negative:
- 'You want to train agents on KSPDG: the KSPDG library explicitly provides evaluation
  environments, NOT training environments, so this table is an evaluation benchmark
  only'
- 'You are running Kerbal Space Program 2 (KSP2): the KSPDG setup only works with
  KSP1 plus the Making History expansion'
- Your metrics of interest are different (e.g., delta-v or capture cost) — the table
  is restricted to closest-approach distance and failure rate as reported in scenario
  E3
tools:
- name: KSPDG
  role: Differential-game (pursuit-evasion) scenario suite in Kerbal Space Program
    that defines scenario E3 and the evaluation environment in which the benchmark
    agents are scored
  repo: https://github.com/mit-ll/spacegym-kspdg
- name: Space Gym
  role: Benchmark library that ships the KSPDG environments and the baseline agents
    (Naive, PPO, iLQGames, Lambert-MPC) compared in the table
  repo: https://github.com/mit-ll/spacegym-kspdg
- name: kspdg (ARCLab-MIT)
  role: MIT-ARCLab participation repository hosting the agent implementations (including
    the navball bot) used to generate training data and evaluation runs
  repo: https://github.com/ARCLab-MIT/kspdg
- name: Hugging Face
  role: Hosts the fine-tuned LLaMA model variants (10, 25, 50 files; sliding window)
    referenced by the table rows
- name: Weights & Biases (wandb)
  role: Public experiment-tracking logs used to trace training runs behind each fine-tuned
    LLaMA row
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:433c9d26123ea627ed4d5656f6425a18c3158756e98931b069e33cb49e787491
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mit-ll/spacegym-kspdg
  tools:
  - Hugging Face
  - KSPDG
  - Space Gym
  - Weights & Biases (wandb)
  - kspdg (ARCLab-MIT)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# spacecraft-agent-benchmark-comparison

## Summary

Reconstructs the LLaMA fine-tuning benchmark table from the KSPDG scenario E3 results of 'Large language models as autonomous spacecraft operators in Kerbal Space Program' (DOI 10.1016/j.asr.2025.06.034), comparing LLaMA agents fine-tuned on 10/25/50 navball-bot log files (plus a sliding-window win=3 variant) against the navball bot, baseline LLaMA, and Space Gym benchmark agents (Naive, PPO, iLQGames, Lambert-MPC) using best/average closest-approach distances and failure rate.

## When to use

Use when reproducing or extending the KSPDG scenario E3 pursuit-evasion benchmark table that compares LLaMA fine-tuned spacecraft agents (10/25/50 training files, sliding window) against the navball bot and Space Gym baselines on closest-approach distance and failure rate.

## When NOT to use

- You want to train agents on KSPDG: the KSPDG library explicitly provides evaluation environments, NOT training environments, so this table is an evaluation benchmark only
- You are running Kerbal Space Program 2 (KSP2): the KSPDG setup only works with KSP1 plus the Making History expansion
- Your metrics of interest are different (e.g., delta-v or capture cost) — the table is restricted to closest-approach distance and failure rate as reported in scenario E3

## Inputs

- Methods-section results table for scenario E3 (closest-approach distances and failure rates per agent)
- Filter-step table rows with numeric metrics, e.g. 'fine-tune 50 files 11.86 29.76 0.00%'
- List of benchmark agent definitions from the Space Gym / KSPDG suites (Naive, PPO, iLQGames, Lambert-MPC, navball bot)

## Outputs

- 9-row CSV table with columns: Method, Best Distance (m), Average Distance (m), Failure Rate
- Validation checklist confirming each cell matches the reported source values and all 9 rows are present

## How to apply

Read the methods-section results table for the LLaMA experiments in scenario E3 and extract best and average closest-approach distances (m) plus failure rate for each LLaMA fine-tuning variant: 10 files (30.52/51.53 m, 0.00%), 25 files (13.54/29.44 m, 0.00%), 50 files (11.86/29.76 m, 0.00%), and 10 files with sliding window win=3 (23.08/40.03 m, 0.00%). Then extract the comparison rows: baseline LLaMA (52.69/140.68 m, 9.09%), the navball bot that generated the training data (34.34/36.43 m), and Space Gym benchmark agents Naive (225.0/225.0 m), PPO (>1643/2346 m), iLQGames (>53.31/60.99 m), and Lambert-MPC (>18.24/47.94 m). Assemble all rows into a single CSV with columns Method, Best Distance (m), Average Distance (m), Failure Rate, preserving the '>' threshold notation where a solver exceeded the evaluation horizon rather than terminating. Validate by checking each reconstructed cell against the source table (e.g., fine-tune 50 files best distance = 11.86 m) and confirming all 9 rows are present. The rationale is that the LLaMA agents were fine-tuned exclusively on navball-bot-collected logs, so the table isolates whether fine-tuning on bot demonstrations improves upon the generator bot itself and the optimal-control/RL baselines.

## Related tools

- **KSPDG** (Differential-game (pursuit-evasion) scenario suite in Kerbal Space Program that defines scenario E3 and the evaluation environment in which the benchmark agents are scored) — https://github.com/mit-ll/spacegym-kspdg
- **Space Gym** (Benchmark library that ships the KSPDG environments and the baseline agents (Naive, PPO, iLQGames, Lambert-MPC) compared in the table) — https://github.com/mit-ll/spacegym-kspdg
- **kspdg (ARCLab-MIT)** (MIT-ARCLab participation repository hosting the agent implementations (including the navball bot) used to generate training data and evaluation runs) — https://github.com/ARCLab-MIT/kspdg
- **Hugging Face** (Hosts the fine-tuned LLaMA model variants (10, 25, 50 files; sliding window) referenced by the table rows)
- **Weights & Biases (wandb)** (Public experiment-tracking logs used to trace training runs behind each fine-tuned LLaMA row)

## Examples

```
git clone git@github.com:mit-ll/spacegym-kspdg.git
```

## Evaluation signals

- All 9 rows are present: 4 LLaMA fine-tune variants, baseline LLaMA, navball bot, Naive, PPO, iLQGames, Lambert-MPC
- fine-tune 50 files row shows Best Distance = 11.86 m and Average = 29.76 m, the best of the LLaMA variants
- Navball (bot) row equals 34.34 / 36.43 m with no failure rate ('-'), since all LLaMA training files were collected with this bot
- All four LLaMA fine-tuned variants report 0.00% failure rate, while the baseline LLaMA reports 9.09%
- '>' threshold notation is preserved for PPO (>1643/2346), iLQGames (>53.31/60.99), and Lambert-MPC (>18.24/47.94), where solvers did not terminate within the evaluation horizon

## Limitations

- Failure rate and closest-approach distances for PPO, iLQGames, and Lambert-MPC are lower-bounded thresholds ('>') because those agents did not terminate within the episode, so exact values cannot be compared
- Fine-tuning hyperparameters (learning rate, epochs, LoRA rank) and hardware for local LLaMA training/inference are not reported, limiting reproducibility
- KSP is not designed for massively parallel headless execution and game time does not pause during agent decisions, penalizing slow policies — results are hardware- and latency-dependent
- The setup has been verified mainly on macOS and partially on Ubuntu 18.04; other operating systems may deviate, and none of it works on KSP2

## Evidence

- [methods] fine-tune 50 files 11.86 29.76 0.00%: "fine-tune 50 files 11.86 29.76 0.00%"
- [methods] Navball (bot) 34.34 36.43 -: "Navball (bot) 34.34 36.43 -"
- [methods] Lambert-MPC >18.24 47.94 -: "Lambert-MPC >18.24 47.94 -"
- [methods] all dataset files were collected using the navball bot: "It is noteworthy that all dataset files were collected using the navball bot"
- [readme] KSPDG provides evaluations environments, NOT training environments: [evidence span withheld — anchor status: not_found]
