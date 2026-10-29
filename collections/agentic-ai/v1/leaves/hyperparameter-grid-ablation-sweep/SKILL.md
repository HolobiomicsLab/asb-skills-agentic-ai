---
name: hyperparameter-grid-ablation-sweep
description: Use when a trained ULPS agent must be re-trained across a crossed grid
  of uncertainty-estimation hyperparameters (e.g. dropout rate x MC forward passes)
  and compared via reward AUC and steps-to-goal to pick a cheaper near-optimal configuration.
when_to_use_negative:
- Input consists of pre-generated result files under results/ that already contain
  the consolidated ablation table — reuse those instead of re-running the 9-configuration
  sweep
- The task targets a different axis of ablation (e.g. LLM guidance blending or baseline
  comparison against Unguided PPO, Linear Decay, Q-Learning, DQN) rather than the
  dropout/forward-pass grid
- The environment is not the MiniGrid UnlockPickup 4x4 setup (e.g. the 8x8 variant),
  since the 9-configuration grid and AUC ranges documented here were established for
  4x4
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: ULPS
  role: Calibrated LLM + PPO agent whose MC dropout uncertainty estimation hyperparameters
    are swept in the ablation grid
  repo: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- name: Proximal Policy Optimization (PPO)
  role: The learned RL policy trained in each ablation configuration before reward
    AUC and steps-to-goal are computed
  repo: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- name: MiniGrid
  role: UnlockPickup benchmark environment (4x4) in which each ablation configuration
    is trained and evaluated
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1109/cai68641.2026.11536354
    title: Uncertainty-Aware LLM-Guided Policy Shaping for Sparse-Reward Reinforcement
      Learning
  source_content_hash: sha256:bdc61d0a624c39f9a41d12585de6b7608b1464eff3adbc27a954af60446d31b7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
  tools:
  - MiniGrid
  - Proximal Policy Optimization (PPO)
  - ULPS
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# hyperparameter-grid-ablation-sweep

## Summary

Runs a full-factorial ablation sweep over Monte Carlo dropout rate and number of stochastic forward passes for the ULPS calibrated LLM-guided PPO agent in MiniGrid-UnlockPickup 4x4, consolidating average steps-to-goal and reward AUC per configuration into a summary table.

## When to use

Use when a trained ULPS agent must be re-trained across a crossed grid of uncertainty-estimation hyperparameters (e.g. dropout rate x MC forward passes) and compared via reward AUC and steps-to-goal to pick a cheaper near-optimal configuration.

## When NOT to use

- Input consists of pre-generated result files under results/ that already contain the consolidated ablation table — reuse those instead of re-running the 9-configuration sweep
- The task targets a different axis of ablation (e.g. LLM guidance blending or baseline comparison against Unguided PPO, Linear Decay, Q-Learning, DQN) rather than the dropout/forward-pass grid
- The environment is not the MiniGrid UnlockPickup 4x4 setup (e.g. the 8x8 variant), since the 9-configuration grid and AUC ranges documented here were established for 4x4

## Inputs

- ULPS 4x4 experiment runner (MainCode/run_4x4_experiment.py) trained on MiniGrid-UnlockPickup 4x4
- Ablation study scripts from the 'Ablation Study/' folder
- Episode-level logs per configuration: episode_metrics_*.csv
- Entropy logs per configuration: entropy_values_*.csv

## Outputs

- experiment_auc_summary.csv with 9 (dropout rate, forward-passes) rows of average steps to goal and reward AUC
- Per-configuration average steps-to-goal and reward AUC values

## How to apply

Launch the ULPS 4x4 experiment via MainCode/run_4x4_experiment.py to obtain baseline episode-level CSV logs, then use the ablation scripts in 'Ablation Study/' to re-run training while sweeping dropout rate (0.05, 0.1, 0.2) crossed with MC forward-pass count (4, 8, 12), yielding 9 configurations. For each run, parse episode_metrics_*.csv and entropy_values_*.csv to compute average steps to goal, and integrate the per-episode reward curves over training to compute reward AUC. Consolidate the 9 (dropout, forward-passes) rows into experiment_auc_summary.csv, then reason over the trade-off: e.g. dropout 0.1 with 8 passes reached nearly the best reward AUC (2055.08) at lower compute cost than the top configuration (2055.88 at dropout 0.2, 12 passes), while dropout 0.05 with 4 passes degraded severely (599.46 AUC, 50 steps). The rationale is that MC dropout uncertainty estimation quality depends on both the dropout rate and the number of stochastic passes, so both axes must be crossed rather than varied one at a time.

## Related tools

- **ULPS** (Calibrated LLM + PPO agent whose MC dropout uncertainty estimation hyperparameters are swept in the ablation grid) — https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- **Proximal Policy Optimization (PPO)** (The learned RL policy trained in each ablation configuration before reward AUC and steps-to-goal are computed) — https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- **MiniGrid** (UnlockPickup benchmark environment (4x4) in which each ablation configuration is trained and evaluated)

## Examples

```
python MainCode/run_4x4_experiment.py; python "Ablation Study/"/ablation_run.py --dropout 0.1 --mc_passes 8  # consolidate into experiment_auc_summary.csv
```

## Evaluation signals

- experiment_auc_summary.csv contains exactly 9 configuration rows (3 dropout rates x 3 forward-pass counts)
- All 9 rows have non-null average-steps-to-goal and reward-AUC values
- Reported metrics are consistent with the pre-generated CSV logs under results/
- Per-configuration average steps to goal is reproducible from the corresponding episode_metrics_*.csv
- Reward AUC values fall within the expected range seen in prior runs (roughly ~599 for the worst configuration up to ~2055 for the best)

## Limitations

- Not all result files are regenerated automatically by a single script; some plotting and summarization scripts assume the same output folder structure used during the experiments
- Some results (CSV files and plots) were pre-generated during experimentation and are included under results/ rather than regenerated from the sweep
- The codebase reflects the research structure and may require minor path adjustments depending on the local environment

## Evidence

- [methods] Sweep the dropout-rate axis (0.05, 0.1, 0.2) using the ablation scripts in the 'Ablation Study/' folder, re-running training with a new dropout rate per configuration.: [evidence span withheld — anchor status: not_found]
- [methods] Consolidate the 9 (dropout, forward-passes) rows of average steps to goal and reward AUC into experiment_auc_summary.csv.: [evidence span withheld — anchor status: not_found]
- [results] using a dropout rate of 0.1 with 8 forward passes achieves nearly the best reward AUC (2055.08) while being computationally cheaper than the top configuration (2055.88 at dropout 0.2, 12 passes): [evidence span withheld — anchor status: not_found]
