---
name: bert-action-predictor-fine-tuning
description: Use when you need to convert an oracle's symbolic solution trajectories
  into a trained LLM action predictor (e.g., BERT on bert-base-uncased) whose probabilistic
  action outputs feed a PPO shaping pipeline on sparse-reward gridworld tasks.
when_to_use_negative:
- The task is not a discrete-action gridworld (e.g., MiniGrid-UnlockPickup); the oracle
  supervision and action-prediction head assume this benchmark's symbolic trajectory
  format.
- A pretrained LLM/guidance policy is already available and calibrated — re-fine-tuning
  from oracle trajectories is unnecessary.
- You need fully scripted end-to-end regeneration of every results file; the README
  notes 'Not all result files are regenerated automatically by a single script' and
  some CSVs/plots are pre-generated artifacts.
edam_operation: http://edamontology.org/operation_3454
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: BERT (bert-base-uncased)
  role: Language model fine-tuned on A*-oracle symbolic trajectories to act as the
    ULPS action predictor
- name: MiniGrid
  role: Benchmark environment (MiniGrid-UnlockPickup) on which oracle trajectories
    are synthesized and the fine-tuned predictor is evaluated
- name: Proximal Policy Optimization (PPO)
  role: Learned RL policy that receives entropy-based guidance from the fine-tuned
    BERT predictor during training
- name: ULPS
  role: Overall framework (Calibrated LLM + PPO) in whose MainCode pipeline the fine-tuning
    and experiment runs execute
  repo: USD-AI-ResearchLab/uncertainty-aware-llm-rl
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1109/cai68641.2026.11536354
    title: Uncertainty-Aware LLM-Guided Policy Shaping for Sparse-Reward Reinforcement
      Learning
  source_content_hash: sha256:38449409f136fafcc51654576d0cbb7b312c2cd54fc54fa05afae8b41574d6af
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
  tools:
  - BERT (bert-base-uncased)
  - MiniGrid
  - Proximal Policy Optimization (PPO)
  - ULPS
  edam_operation: http://edamontology.org/operation_3454
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# bert-action-predictor-fine-tuning

## Summary

Fine-tunes a bert-base-uncased language model as an action predictor for MiniGrid-UnlockPickup, using optimal symbolic trajectories synthesized by an A*-based oracle as supervision, to serve as the calibrated LLM guidance policy in the ULPS framework.

## When to use

Use when you need to convert an oracle's symbolic solution trajectories into a trained LLM action predictor (e.g., BERT on bert-base-uncased) whose probabilistic action outputs feed a PPO shaping pipeline on sparse-reward gridworld tasks.

## When NOT to use

- The task is not a discrete-action gridworld (e.g., MiniGrid-UnlockPickup); the oracle supervision and action-prediction head assume this benchmark's symbolic trajectory format.
- A pretrained LLM/guidance policy is already available and calibrated — re-fine-tuning from oracle trajectories is unnecessary.
- You need fully scripted end-to-end regeneration of every results file; the README notes 'Not all result files are regenerated automatically by a single script' and some CSVs/plots are pre-generated artifacts.

## Inputs

- A*-oracle symbolic demonstration trajectories for MiniGrid-UnlockPickup
- bert-base-uncased pretrained model checkpoint
- MiniGrid-UnlockPickup environment (4x4 and 8x8 grid configurations)

## Outputs

- Fine-tuned BERT action predictor (calibrated LLM policy for ULPS)
- episode_metrics_*.csv per-episode logs under results/
- entropy_values_*.csv entropy logs
- calibration_metrics_*.csv (accuracy, 10-bin ECE, Brier Score)
- experiment_auc_summary.csv consolidated summary for Table I comparison

## How to apply

First run the A*-based oracle on MiniGrid-UnlockPickup (4x4 and 8x8 configurations) to synthesize optimal symbolic demonstration trajectories, which become the fine-tuning supervision. Train the bert-base-uncased action predictor on these trajectories inside the MainCode pipeline, then load the fine-tuned predictor into run_4x4_experiment.py / run_8x8_experiment.py where its action probabilities are blended with the PPO policy via an entropy-based mechanism conditioned on Monte Carlo dropout uncertainty. Log per-episode outcomes and calibration diagnostics (accuracy, 10-bin Expected Calibration Error, Brier Score) to episode_metrics_*.csv, entropy_values_*.csv, and calibration_metrics_*.csv under results/. Judge success by calibration quality: the paper reports Brier Score 0.06 (vs. 0.20 uncalibrated), Discrimination Analysis 1.0 (vs. 0.8), ECE 0.20, and >9% execution-accuracy improvement after fine-tuning relative to reference configurations.

## Related tools

- **BERT (bert-base-uncased)** (Language model fine-tuned on A*-oracle symbolic trajectories to act as the ULPS action predictor)
- **MiniGrid** (Benchmark environment (MiniGrid-UnlockPickup) on which oracle trajectories are synthesized and the fine-tuned predictor is evaluated)
- **Proximal Policy Optimization (PPO)** (Learned RL policy that receives entropy-based guidance from the fine-tuned BERT predictor during training)
- **ULPS** (Overall framework (Calibrated LLM + PPO) in whose MainCode pipeline the fine-tuning and experiment runs execute) — USD-AI-ResearchLab/uncertainty-aware-llm-rl

## Examples

```
python MainCode/run_4x4_experiment.py
```

## Evaluation signals

- experiment_auc_summary.csv exists with accuracy, ECE, Brier Score, and AUC-ROC populated for all four model rows (ULPS, Shoaeinaeini & Harrison, Unguided RL, Uncalibrated LLM).
- ULPS row shows Brier Score of 0.06 (vs. 0.20), Discrimination Analysis of 1.0 (vs. 0.8), and ECE of 0.20.
- Execution accuracy after fine-tuning improves by more than 9% relative to reference configurations.
- calibration_metrics_*.csv uses a 10-bin Expected Calibration Error binning scheme.
- Per-episode logs (episode_metrics_*.csv) and entropy logs (entropy_values_*.csv) are generated by the run scripts for each grid size.

## Limitations

- Not all result files are regenerated automatically by a single script; some CSV files and plots were pre-generated during experimentation and are included under results/.
- Some plotting and summarization scripts assume the same output folder structure used during the experiments.
- Only a subset of development plots was selected for the final paper, so the artifact does not reproduce every intermediate figure.
- No changelog found for the repository; some parts may require minor path adjustments depending on the local environment.

## Evidence

- [abstract] oracle trajectories as supervision: "ULPS employs an A*-based oracle to synthesize optimal symbolic trajectories, which are used to fine-tune a BERT-based language model"
- [results] calibration outcome: [evidence span withheld — anchor status: not_found]
