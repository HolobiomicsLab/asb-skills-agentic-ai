---
name: cross-model-calibration-comparison
description: Use when comparing probabilistic action-prediction quality across a fine-tuned/uncertainty-aware
  model and baselines (prior work, unguided RL, uncalibrated LLM) and you must fill
  a calibration metrics table from logged episode and calibration CSVs.
when_to_use_negative:
- You need to regenerate raw episode-level logs from scratch — per the README, not
  all result files are regenerated automatically by a single script and some CSVs
  were pre-generated during experimentation.
- You have only a single model's predictions and no reference configurations — there
  is no cross-model Table I comparison to consolidate.
edam_operation: http://edamontology.org/operation_3659
tools:
- name: BERT (bert-base-uncased)
  role: Fine-tuned action predictor whose probabilistic outputs are scored for calibration
    (accuracy, 10-bin ECE, Brier Score, AUC-ROC).
- name: Proximal Policy Optimization (PPO)
  role: Learned policy blended with the LLM guidance; its unguided variant is one
    of the reference configurations in the comparison.
- name: MiniGrid
  role: Environment (MiniGrid-UnlockPickup) whose 4x4 and 8x8 experiment runs generate
    the episode and calibration CSV logs.
- name: ULPS
  role: MainCode pipeline (Calibrated LLM + PPO) whose run scripts emit the calibration_metrics_*.csv
    files consolidated into experiment_auc_summary.csv.
  repo: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1109/cai68641.2026.11536354
    title: Uncertainty-Aware LLM-Guided Policy Shaping for Sparse-Reward Reinforcement
      Learning
  source_content_hash: sha256:3a29ea77ebe83f363d14ec69813d15b76d107f3001852a08ad72f6ea8767e9f3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
  tools:
  - BERT (bert-base-uncased)
  - MiniGrid
  - Proximal Policy Optimization (PPO)
  - ULPS
  edam_operation: http://edamontology.org/operation_3659
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# cross-model-calibration-comparison

## Summary

Consolidates per-model calibration diagnostics (accuracy, 10-bin Expected Calibration Error, Brier Score, and Discrimination Analysis AUC-ROC) for the ULPS fine-tuned BERT action predictor and its reference configurations into a single experiment_auc_summary.csv matching the paper's Table I comparison.

## When to use

Use when comparing probabilistic action-prediction quality across a fine-tuned/uncertainty-aware model and baselines (prior work, unguided RL, uncalibrated LLM) and you must fill a calibration metrics table from logged episode and calibration CSVs.

## When NOT to use

- You need to regenerate raw episode-level logs from scratch — per the README, not all result files are regenerated automatically by a single script and some CSVs were pre-generated during experimentation.
- You have only a single model's predictions and no reference configurations — there is no cross-model Table I comparison to consolidate.

## Inputs

- results/episode_metrics_*.csv (per-episode outcomes)
- results/entropy_values_*.csv (per-episode entropy from MC dropout blending)
- results/calibration_metrics_*.csv (accuracy, 10-bin ECE, Brier Score)
- Reference-model configurations: Shoaeinaeini & Harrison, Unguided RL, Uncalibrated LLM

## Outputs

- results/experiment_auc_summary.csv with accuracy, ECE, Brier Score, and AUC-ROC fields populated for all four model rows
- Calibration comparison rows matching the structure of the paper's Table I

## How to apply

Run the ULPS pipeline (A*-based oracle trajectories fine-tune bert-base-uncased, MC dropout estimates epistemic uncertainty, entropy-based blending with PPO via run_4x4_experiment.py / run_8x8_experiment.py), which logs per-episode outcomes, entropy values, and calibration diagnostics to episode_metrics_*.csv, entropy_values_*.csv, and calibration_metrics_*.csv under results/. Compute accuracy and 10-bin Expected Calibration Error and Brier Score from the calibration logs, then compute Discrimination Analysis AUC-ROC for the proposed model and for the Shoaeinaeini & Harrison, Unguided RL, and Uncalibrated LLM reference configurations. Consolidate all metric rows into results/experiment_auc_summary.csv using the same structure as the paper's Table I. Validate that all four model rows have accuracy, ECE, Brier Score, and AUC-ROC populated and that the ULPS row reflects the reported >9% execution-accuracy improvement after fine-tuning (reported values: Brier 0.06 vs 0.20, Discrimination Analysis 1.0 vs 0.8, ECE 0.20).

## Related tools

- **BERT (bert-base-uncased)** (Fine-tuned action predictor whose probabilistic outputs are scored for calibration (accuracy, 10-bin ECE, Brier Score, AUC-ROC).)
- **Proximal Policy Optimization (PPO)** (Learned policy blended with the LLM guidance; its unguided variant is one of the reference configurations in the comparison.)
- **MiniGrid** (Environment (MiniGrid-UnlockPickup) whose 4x4 and 8x8 experiment runs generate the episode and calibration CSV logs.)
- **ULPS** (MainCode pipeline (Calibrated LLM + PPO) whose run scripts emit the calibration_metrics_*.csv files consolidated into experiment_auc_summary.csv.) — https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl

## Examples

```
python MainCode/run_8x8_experiment.py  # then consolidate results/experiment_auc_summary.csv and verify accuracy, ECE, Brier Score, AUC-ROC are populated for all four model rows
```

## Evaluation signals

- experiment_auc_summary.csv exists with accuracy, ECE, Brier Score, and AUC-ROC fields populated for all four model rows.
- ULPS row shows improved execution accuracy after fine-tuning relative to the reference rows (paper reports >9% improvement).
- Proposed model's Brier Score (~0.06) is clearly below the prior work's (~0.20) and its Discrimination Analysis (~1.0) exceeds the reference (~0.8).
- ECE values were computed with the paper's 10-bin scheme and match the reported ECE of 0.20 for the proposed model.
- Table structure matches experiment_auc_summary.csv as used for the paper's Table I comparison (four model rows: ULPS, Shoaeinaeini & Harrison, Unguided RL, Uncalibrated LLM).

## Limitations

- Not all result files are regenerated automatically by a single script; some CSV files and plots were generated during experimentation and are only included under results/.
- Some plotting and summarization scripts assume the same output folder structure used during the experiments, so minor path adjustments may be required locally.
- Several additional plots were generated during development but only a subset was selected for the final paper, so consolidated summaries may not cover every diagnostic.
- No changelog found in the repository, so provenance of pre-generated calibration artifacts is not documented.

## Evidence

- [results] The proposed fine-tuned model achieves superior calibration: a Brier Score of 0.06 (vs. 0.20), Discrimination Analysis of 1.0 (vs. 0.8), and Expected Calibration Error of 0.20: [evidence span withheld — anchor status: not_found]
- [full_text] Log per-episode outcomes, entropy values, and calibration diagnostics (accuracy, 10-bin Expected Calibration Error, Brier Score) to the episode_metrics_*.csv, entropy_values_*.csv, and calibration_metrics_*.csv files under results/.: [evidence span withheld — anchor status: not_found]
