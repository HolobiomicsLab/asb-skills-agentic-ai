---
name: auc-roc-discrimination-analysis
description: Use when comparing probabilistic action predictors (e.g., an LLM policy
  vs. uncalibrated LLM, unguided RL, or prior work) and you need discrimination plus
  calibration metrics (ECE, Brier Score, AUC-ROC) consolidated into a comparison table.
when_to_use_negative:
- When only a single model's point-estimate accuracy is needed with no probabilistic
  calibration or ranking comparison
- When the environment/task is not MiniGrid-UnlockPickup-style sparse-reward RL with
  binary per-episode success outcomes, since the discrimination metric is defined
  over binary episode outcomes
- When you cannot produce predicted action probabilities (e.g., a deterministic policy
  without confidence scores), as ECE/Brier/AUC-ROC all require calibrated probability
  outputs
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3474
tools:
- name: BERT (bert-base-uncased)
  role: Fine-tuned LLM action predictor whose probabilistic predictions are scored
    with AUC-ROC and calibration metrics
- name: Proximal Policy Optimization (PPO)
  role: Learned RL policy blended with the LLM policy; baseline (Unguided PPO) included
    in the discrimination comparison
- name: ULPS
  role: Main implementation (Calibrated LLM + PPO) and experiment runners that emit
    the logs feeding the AUC-ROC analysis
- name: MiniGrid
  role: UnlockPickup benchmark environment generating the episode outcomes used for
    discrimination analysis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1109/cai68641.2026.11536354
    title: Uncertainty-Aware LLM-Guided Policy Shaping for Sparse-Reward Reinforcement
      Learning
  source_content_hash: sha256:ffbfea63aa719eace89bc1659f6ae19858f7ac48cc29f328f2fca1b84f687662
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BERT (bert-base-uncased)
  - MiniGrid
  - Proximal Policy Optimization (PPO)
  - ULPS
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# auc-roc-discrimination-analysis

## Summary

Computes Discrimination Analysis (AUC-ROC) alongside accuracy, 10-bin Expected Calibration Error, and Brier Score to evaluate whether a fine-tuned BERT action predictor produces better-calibrated probabilistic predictions than prior/baseline models in the ULPS MiniGrid-UnlockPickup setting.

## When to use

Use when comparing probabilistic action predictors (e.g., an LLM policy vs. uncalibrated LLM, unguided RL, or prior work) and you need discrimination plus calibration metrics (ECE, Brier Score, AUC-ROC) consolidated into a comparison table.

## When NOT to use

- When only a single model's point-estimate accuracy is needed with no probabilistic calibration or ranking comparison
- When the environment/task is not MiniGrid-UnlockPickup-style sparse-reward RL with binary per-episode success outcomes, since the discrimination metric is defined over binary episode outcomes
- When you cannot produce predicted action probabilities (e.g., a deterministic policy without confidence scores), as ECE/Brier/AUC-ROC all require calibrated probability outputs

## Inputs

- episode_metrics_*.csv (per-episode outcomes from ULPS experiment runners)
- entropy_values_*.csv (MC-dropout entropy logs used for uncertainty-conditioned blending)
- calibration_metrics_*.csv (accuracy, 10-bin ECE, Brier Score per configuration)
- Predicted action probabilities and binary episode success outcomes for the ULPS model and the Shoaeinaeinaeini & Harrison / Unguided RL / Uncalibrated LLM baselines

## Outputs

- experiment_auc_summary.csv with accuracy, ECE, Brier Score, and AUC-ROC fields populated for all four model rows
- Discrimination Analysis (AUC-ROC) values for the proposed and reference configurations matching Table I

## How to apply

Run the ULPS experiment runners (MainCode/run_4x4_experiment.py, run_8x8_experiment.py) to produce episode_metrics_*.csv, entropy_values_*.csv, and calibration_metrics_*.csv under results/. From the logged per-episode outcomes and predicted action probabilities, compute accuracy, a 10-bin Expected Calibration Error, Brier Score, and Discrimination Analysis as AUC-ROC. Compute the same metrics for the reference configurations (Shoaeinaeini & Harrison, Unguided RL, Uncalibrated LLM) so all four model rows are directly comparable. Consolidate the metric rows into experiment_auc_summary.csv, which mirrors the structure of the paper's Table I comparison. Judge correctness by checking that the ULPS row shows Brier Score ~0.06 vs. 0.20, Discrimination Analysis ~1.0 vs. 0.8, and ECE ~0.20, reflecting the reported >9% execution-accuracy improvement after fine-tuning.

## Related tools

- **BERT (bert-base-uncased)** (Fine-tuned LLM action predictor whose probabilistic predictions are scored with AUC-ROC and calibration metrics)
- **Proximal Policy Optimization (PPO)** (Learned RL policy blended with the LLM policy; baseline (Unguided PPO) included in the discrimination comparison)
- **ULPS** (Main implementation (Calibrated LLM + PPO) and experiment runners that emit the logs feeding the AUC-ROC analysis)
- **MiniGrid** (UnlockPickup benchmark environment generating the episode outcomes used for discrimination analysis)

## Examples

```
python MainCode/run_4x4_experiment.py  # then consolidate metrics into results/experiment_auc_summary.csv
```

## Evaluation signals

- experiment_auc_summary.csv exists with accuracy, ECE, Brier Score, and AUC-ROC fields populated for all four model rows (ULPS, Shoaeinaeini & Harrison, Unguided RL, Uncalibrated LLM)
- ULPS row shows Brier Score ~0.06 vs. 0.20 for the prior model, Discrimination Analysis ~1.0 vs. 0.8, and ECE ~0.20
- ULPS execution accuracy reflects the reported >9% improvement after fine-tuning relative to reference rows
- ECE is computed with 10 bins and Brier Score over the same episode set used for AUC-ROC, so metrics are directly comparable across configurations
- Underlying episode_metrics_*.csv and calibration_metrics_*.csv files exist in results/ and are consistent with the consolidated summary

## Limitations

- Not all result files are regenerated automatically by a single script; some consolidation steps must be run manually
- Some results (CSV files and plots) were pre-generated during experimentation and shipped under results/ rather than recomputed
- Plotting and summarization scripts assume the same output folder structure used during the experiments, and may need minor path adjustments locally
- No changelog found in the repository, so provenance of pre-generated metric files is hard to audit

## Evidence

- [abstract] calibration metrics definition: "whose influence is conditioned on epistemic uncertainty estimated via Monte Carlo (MC) dropout"
- [results] ULPS calibration improvement finding: [evidence span withheld — anchor status: not_found]
- [readme] script outputs logs: "These scripts generate episode-level CSV logs and summaries."
- [readme] result file inventory: "consolidated experiment summaries (`experiment_auc_summary.csv`)"
- [readme] reproducibility caveat: "Not all result files are regenerated automatically by a single script."
