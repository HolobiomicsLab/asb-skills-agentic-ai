---
name: expected-calibration-error-computation
description: Use when comparing probabilistic action/prediction quality across models
  and you must quantify confidence calibration from logged per-episode outcomes, producing
  ECE, Brier Score, and AUC-ROC rows for a calibration comparison table.
when_to_use_negative:
- When the goal is raw execution accuracy or reward AUC rather than probabilistic
  calibration quality — use the episode/reward summaries instead.
- When predictions are already summarized without access to per-episode outcome logs
  or per-action probabilities, since ECE binning and Brier Score require individual
  probabilistic predictions.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3474
tools:
- name: BERT (bert-base-uncased)
  role: Fine-tuned action predictor whose probabilistic predictions are evaluated
    for calibration (ECE, Brier Score, Discrimination Analysis).
- name: ULPS
  role: Main implementation (Calibrated LLM + PPO) and experiment runners that generate
    the episode-level CSV logs feeding the calibration metrics.
- name: Proximal Policy Optimization (PPO)
  role: Learned policy blended with the LLM guidance whose resulting predictions are
    calibrated and evaluated against baselines.
- name: MiniGrid
  role: MiniGrid-UnlockPickup benchmark environment generating the episodic outcomes
    used for calibration diagnostics.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1109/cai68641.2026.11536354
    title: Uncertainty-Aware LLM-Guided Policy Shaping for Sparse-Reward Reinforcement
      Learning
  source_content_hash: sha256:2bf533eb1390458737c9b6698514feefa4d857c1e1301f73a82a61c168646227
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BERT (bert-base-uncased)
  - MiniGrid
  - Proximal Policy Optimization (PPO)
  - ULPS
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# expected-calibration-error-computation

## Summary

Computation of probabilistic calibration diagnostics (10-bin Expected Calibration Error, Brier Score, and Discrimination Analysis AUC-ROC) for a fine-tuned BERT action predictor and reference baselines in the ULPS MiniGrid-UnlockPickup pipeline, used to demonstrate that the calibrated LLM-guided policy is better calibrated than prior and baseline models (Table I).

## When to use

Use when comparing probabilistic action/prediction quality across models and you must quantify confidence calibration from logged per-episode outcomes, producing ECE, Brier Score, and AUC-ROC rows for a calibration comparison table.

## When NOT to use

- When the goal is raw execution accuracy or reward AUC rather than probabilistic calibration quality — use the episode/reward summaries instead.
- When predictions are already summarized without access to per-episode outcome logs or per-action probabilities, since ECE binning and Brier Score require individual probabilistic predictions.

## Inputs

- episode-level outcome logs (episode_metrics_*.csv)
- entropy logs from Monte Carlo dropout (entropy_values_*.csv)
- fine-tuned bert-base-uncased action predictor probabilities
- reference-configuration predictions (Shoaeinaeini & Harrison, Unguided RL, Uncalibrated LLM) for AUC-ROC comparison

## Outputs

- calibration metrics files (calibration_metrics_*.csv) with accuracy, 10-bin ECE, and Brier Score
- consolidated calibration comparison table (experiment_auc_summary.csv) with accuracy, ECE, Brier Score, and AUC-ROC for all four model rows

## How to apply

Run the ULPS experiment runners (run_4x4_experiment.py / run_8x8_experiment.py) to emit episode-level logs (episode_metrics_*.csv) and entropy logs (entropy_values_*.csv); while doing so, estimate epistemic uncertainty via Monte Carlo dropout over the fine-tuned bert-base-uncased action predictor. From these logs compute accuracy, a 10-bin Expected Calibration Error, and Brier Score, logging the diagnostics to calibration_metrics_*.csv under results/. Then compute Discrimination Analysis AUC-ROC for the proposed ULPS model and the reference configurations (Shoaeinaeini & Harrison, Unguided RL, Uncalibrated LLM), and consolidate all metric rows into experiment_auc_summary.csv. Interpret against the paper's reported reference points: ULPS achieves a Brier Score of 0.06 (vs. 0.20 for prior work), Discrimination Analysis of 1.0 (vs. 0.8), and ECE of 0.20, indicating better-aligned probabilistic predictions.

## Related tools

- **BERT (bert-base-uncased)** (Fine-tuned action predictor whose probabilistic predictions are evaluated for calibration (ECE, Brier Score, Discrimination Analysis).)
- **ULPS** (Main implementation (Calibrated LLM + PPO) and experiment runners that generate the episode-level CSV logs feeding the calibration metrics.)
- **Proximal Policy Optimization (PPO)** (Learned policy blended with the LLM guidance whose resulting predictions are calibrated and evaluated against baselines.)
- **MiniGrid** (MiniGrid-UnlockPickup benchmark environment generating the episodic outcomes used for calibration diagnostics.)

## Examples

```
python MainCode/run_4x4_experiment.py  # generates episode_metrics_*.csv, entropy_values_*.csv, calibration_metrics_*.csv under results/
```

## Evaluation signals

- experiment_auc_summary.csv exists and contains accuracy, ECE, Brier Score, and AUC-ROC fields populated for all four model rows.
- The ULPS row reflects the reported improvement in execution accuracy after fine-tuning relative to the reference rows.
- Reported values fall near the paper's Table I reference points: Brier Score 0.06 (vs. 0.20), Discrimination Analysis 1.0 (vs. 0.8), ECE 0.20.
- calibration_metrics_*.csv files exist under results/ for each run (4x4 and 8x8) with the 10-bin ECE diagnostics populated.
- ECE computation consistently uses 10 bins, matching the paper's calibration protocol.

## Limitations

- Not all result files are regenerated automatically by a single script — consolidation into experiment_auc_summary.csv may require manual or partial scripting.
- Some results (CSV files and plots) were pre-generated during experimentation and included under results/ rather than reproducibly recomputed.
- Some plotting and summarization scripts assume the same output folder structure used during the experiments; minor path adjustments may be needed.

## Evidence

- [methods] workflow step 4 logging calibration diagnostics: [evidence span withheld — anchor status: not_found]
- [methods] consolidation validation: [evidence span withheld — anchor status: not_found]
- [readme] output file structure from README: [evidence span withheld — anchor status: not_found]
