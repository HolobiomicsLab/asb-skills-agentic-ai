---
name: rl-sample-efficiency-analysis
description: Use when comparing RL methods for sparse-reward benchmarks on sample
  efficiency, using episode-level step logs (episode_metrics_*.csv) to compute total
  environment steps and relative reduction percentages.
when_to_use_negative:
- The analysis goal is execution accuracy, success rate, or reward AUC rather than
  environment-step/sample-complexity comparison
- No per-episode step counts are available (e.g., only pre-aggregated or pre-generated
  summary files without episode_metrics_*.csv logs)
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3810
tools:
- name: ULPS
  role: Guided method whose episode-level environment interactions are quantified
    (7,286 total steps in the 4x4 setting)
  repo: github:github.com__USD-AI-ResearchLab__uncertainty-aware-llm-rl
- name: Proximal Policy Optimization (PPO)
  role: Unguided baseline policy whose episode-level environment interactions (49,147
    total steps) provide the comparison denominator
  repo: github:github.com__USD-AI-ResearchLab__uncertainty-aware-llm-rl
- name: MiniGrid
  role: MiniGrid-UnlockPickup benchmark environment (4x4 and 8x8 grids) on which the
    step counts are collected
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1109/cai68641.2026.11536354
    title: Uncertainty-Aware LLM-Guided Policy Shaping for Sparse-Reward Reinforcement
      Learning
  source_content_hash: sha256:e442a8930ab72d9179a0bb3f234f8e36148c9b3b3f86b0c6bc4afc809d1b914d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MiniGrid
  - Proximal Policy Optimization (PPO)
  - ULPS
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3810
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# rl-sample-efficiency-analysis

## Summary

Quantifies sample complexity in reinforcement learning by aggregating per-episode step counts into total environment interactions and comparing methods (e.g., ULPS vs. unguided PPO) to compute relative reduction in interactions on MiniGrid-UnlockPickup.

## When to use

Use when comparing RL methods for sparse-reward benchmarks on sample efficiency, using episode-level step logs (episode_metrics_*.csv) to compute total environment steps and relative reduction percentages.

## When NOT to use

- The analysis goal is execution accuracy, success rate, or reward AUC rather than environment-step/sample-complexity comparison
- No per-episode step counts are available (e.g., only pre-aggregated or pre-generated summary files without episode_metrics_*.csv logs)

## Inputs

- Episode-level step-count logs (results/episode_metrics_*.csv) for ULPS 4x4 and 8x8 MiniGrid-UnlockPickup runs
- Episode-level step-count logs (results/episode_metrics_*.csv) for the unguided PPO baseline run

## Outputs

- Total environment steps per configuration (e.g., ULPS 7,286 vs unguided RL 49,147)
- Relative reduction in environment interactions (~86%)
- Average steps per episode per grid size (4x4: 7.24; 8x8: 15.37)
- Sample-efficiency summary table saved as CSV (e.g., sample_efficiency_summary.csv) in results/

## How to apply

Load per-episode step counts from the episode-level logs (episode_metrics_*.csv) under results/ for each configuration (ULPS runs on 4x4 and 8x8 MiniGrid-UnlockPickup and the unguided PPO baseline) using pandas. Aggregate episode steps per configuration to obtain total environment interactions; in the reference experiment this yields ULPS 7,286 steps versus 49,147 for unguided RL. Compute the relative reduction as (49,147 - 7,286) / 49,147, reproducing the reported ~86% reduction. Additionally compute average steps per episode per grid size (4x4: 7.24, 8x8: 15.37) and check the ratio (15.37/7.24 ≈ 2.12) for near-linear scaling against the quadrupled grid area. Assemble totals, reduction percentage, and per-grid average steps into a sample-efficiency summary table and save it as CSV (e.g., sample_efficiency_summary.csv) alongside consolidated summaries in results/.

## Related tools

- **ULPS** (Guided method whose episode-level environment interactions are quantified (7,286 total steps in the 4x4 setting)) — github:github.com__USD-AI-ResearchLab__uncertainty-aware-llm-rl
- **Proximal Policy Optimization (PPO)** (Unguided baseline policy whose episode-level environment interactions (49,147 total steps) provide the comparison denominator) — github:github.com__USD-AI-ResearchLab__uncertainty-aware-llm-rl
- **MiniGrid** (MiniGrid-UnlockPickup benchmark environment (4x4 and 8x8 grids) on which the step counts are collected)

## Examples

```
python MainCode/run_4x4_experiment.py
```

## Evaluation signals

- Computed total steps reproduce the reported values: ULPS 7,286 vs unguided RL 49,147 in the 4x4 MiniGrid-UnlockPickup setting
- Relative reduction (49,147 - 7,286) / 49,147 reproduces the reported ~86%
- Average-steps ratio 15.37/7.24 ≈ 2.12 is consistent with approximately linear scaling when grid size doubles (area quadruples)
- Summary CSV (e.g., sample_efficiency_summary.csv) contains totals, reduction percentage, and per-grid average steps and is saved alongside consolidated summaries in results/

## Limitations

- Not all result files are regenerated automatically by a single script; some results (CSV files and plots) were pre-generated during experimentation and are included under results/
- Some plotting and summarization scripts assume the same output folder structure used during the experiments, so path adjustments may be required locally

## Evidence

- [results] ULPS requires only 7,286 total environment steps versus 49,147 for unguided RL, an 86% reduction in environment interactions.: [evidence span withheld — anchor status: not_found]
