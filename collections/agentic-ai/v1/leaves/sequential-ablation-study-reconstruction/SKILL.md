---
name: sequential-ablation-study-reconstruction
description: 'Use when a paper reports an ablation table where configurations add
  one training/ intervention at a time (e.g. GPT-3.5 fine-tuning rows: Simple fine-tuning,
  + Hyperparam tuning, + System prompt, + Two train gameplays) and the per-step metric
  deltas must be extracted and tabulated.'
when_to_use_negative:
- The paper's table contains independent, non-cumulative model comparisons (e.g. LLaMA
  10/25/50-file datasets, Navball bot, Lambert-MPC baseline) rather than a sequential
  ablation over stages
- The evaluation metric semantics are unknown — here lower distance is better, and
  misreading 132.09 as worse than 263.55 inverts the conclusion
- Only training/validation losses are reported with no environment evaluation metrics
  to reconstruct
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: GPT-3.5
  role: Model fine-tuned sequentially through the four ablation stages being reconstructed
    from the table
- name: Weights & Biases
  role: Experiment tracking where detailed logs, metrics, and visualizations of the
    fine-tuning runs can be reviewed for verification
- name: pandas
  role: Used to assemble the extracted ablation rows into the method/best_distance_m/avg_distance_m/failure_rate
    CSV
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:36c9f07dc305d6f5a2976a2985e9f4260e445df9722d1a4238446871a5b73fcb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GPT-3.5
  - Weights & Biases
  - pandas
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# Sequential ablation study reconstruction

## Summary

Reconstructs a paper's sequential fine-tuning ablation table (e.g. GPT-3.5 simple fine-tuning → +hyperparam tuning → +system prompt → +two train gameplays) as a structured CSV of evaluation metrics (Best Distance, Average Distance, Failure Rate) so that stepwise contributions can be compared.

## When to use

Use when a paper reports an ablation table where configurations add one training/ intervention at a time (e.g. GPT-3.5 fine-tuning rows: Simple fine-tuning, + Hyperparam tuning, + System prompt, + Two train gameplays) and the per-step metric deltas must be extracted and tabulated.

## When NOT to use

- The paper's table contains independent, non-cumulative model comparisons (e.g. LLaMA 10/25/50-file datasets, Navball bot, Lambert-MPC baseline) rather than a sequential ablation over stages
- The evaluation metric semantics are unknown — here lower distance is better, and misreading 132.09 as worse than 263.55 inverts the conclusion
- Only training/validation losses are reported with no environment evaluation metrics to reconstruct

## Inputs

- Paper results/methods section containing the ablation comparison table (Table 5-style rows with Best Distance (m), Average Distance (m), Failure Rate)
- Training/validation loss table for the same fine-tuning stages (optional cross-check)
- Source card / research question specifying which configuration row is the target

## Outputs

- gpt_finetuning_results.csv with columns method, best_distance_m, avg_distance_m, failure_rate
- Row-wise deltas showing stepwise performance change relative to the simple fine-tuning baseline (263.55/265.89)

## How to apply

Parse the results/methods section and locate the ablation comparison rows (for this paper: Baseline GPT, Simple fine-tuning, + Hyperparam tuning, + System prompt, + Two train gameplays). For each row extract the three reported KSPDG evaluation columns — Best Distance (m), Average Distance (m), and Failure Rate — e.g. Simple fine-tuning 263.55/265.89/0.0% and + Two train gameplays 132.09/159.78/0.2%. Note that in KSPDG pursuit metrics lower distance is better, so 132.09/159.78 is the best score, not the worst. Assemble the rows into a CSV with columns method, best_distance_m, avg_distance_m, failure_rate using pandas, then validate each cell against the source table verbatim (263.55, 188.90, 197.41, 132.09 for the four fine-tuning stages) before saving gpt_finetuning_results.csv. Optionally cross-check against the training/validation-loss table (e.g. + 2 human gameplays had training loss 1.3e-4 but validation loss worsening to 4.4e-1) to flag configurations where best evaluation score does not align with best training loss.

## Related tools

- **GPT-3.5** (Model fine-tuned sequentially through the four ablation stages being reconstructed from the table)
- **Weights & Biases** (Experiment tracking where detailed logs, metrics, and visualizations of the fine-tuning runs can be reviewed for verification)
- **pandas** (Used to assemble the extracted ablation rows into the method/best_distance_m/avg_distance_m/failure_rate CSV)

## Examples

```
import pandas as pd; df = pd.DataFrame({'method':['Simple fine-tuning','+ Hyperparam tuning','+ System prompt','+ Two train gameplays'],'best_distance_m':[263.55,188.90,197.41,132.09],'avg_distance_m':[265.89,202.08,214.87,159.78],'failure_rate':['0.0%','0.1%','0.0%','0.2%']}); df.to_csv('gpt_finetuning_results.csv', index=False)
```

## Evaluation signals

- Each CSV cell matches the source table exactly: best distances 263.55, 188.90, 197.41, 132.09 and avg distances 265.89, 202.08, 214.87, 159.78 for the four fine-tuning stages
- Failure rates recorded as reported: 0.0% (simple), 0.1% (+hyperparam), 0.0% (+system prompt), 0.2% (+two gameplays)
- Best overall configuration identified as + Two train gameplays at 132.09/159.78, consistent with lower-is-better KSPDG distance metrics
- Schema check: CSV has exactly the columns method, best_distance_m, avg_distance_m, failure_rate
- Cross-check stage losses where available: LRM 0.2 → 2.3e-2/2.8e-2, system prompt → 1.9e-1/5.2e-3, + 2 human gameplays → 1.3e-4/4.4e-1 (train/validation)

## Limitations

- Fine-tuning hyperparameters (learning rate, epochs, LoRA rank) are not reported in the paper's methods section, limiting exact reproduction of each stage
- The final ablation stage shows best evaluation scores but validation loss worsened to 4.4e-1 with training loss 1.3e-4, indicating overfitting risk not visible from the metrics table alone
- KSPDG provides evaluation environments, not training environments, so scores depend on the non-blocking step function and scripted bot adversaries rather than a fixed RL training set

## Evidence

- [methods] Simple fine-tuning row: "Simple fine-tuning 263.55 265.89 0.0%"
- [methods] Two train gameplays row: "+ Two train gameplays 132.09 159.78 0.2%"
- [results] Validation loss cross-check: "+ 2 human gameplays 1.3 × 10−4 4.4 × 10−1"
- [readme] Evaluation-not-training environment: [evidence span withheld — anchor status: not_found]
