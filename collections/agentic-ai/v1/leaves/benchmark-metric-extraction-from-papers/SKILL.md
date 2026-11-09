---
name: benchmark-metric-extraction-from-papers
description: 'Use when a paper reports a benchmark comparison table (e.g. GPT-3.5
  fine-tuning ablation rows in the KSPDG evaluation: Baseline GPT, Simple fine-tuning,
  + Hyperparam tuning, + System prompt, + Two train gameplays) and the analysis goal
  requires transcribing those metrics into machine-readable form.'
when_to_use_negative:
- The paper reports training/validation losses (e.g. the LRM ablation with training
  loss 2.3e-2 and validation loss 2.8e-2) rather than evaluation benchmark metrics
  — these are separate tables requiring different extraction.
- The goal is to run the KSPDG evaluation environments yourself to obtain new metrics,
  rather than extract already-published numbers (spacegym-kspdg provides evaluation
  environments, not training ones).
- The metrics of interest come from raw wandb logs or LLaMA fine-tuning dataset files
  instead of a printed comparison table in the manuscript.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: pandas
  role: Assembles the extracted table rows into a CSV with the method/best_distance_m/avg_distance_m/failure_rate
    schema
- name: Weights & Biases
  role: Publicly accessible experiment-tracking logs used to cross-check the metric
    values transcribed from the paper
- name: spacegym-kspdg
  role: Defines the KSPDG benchmark environments whose published evaluation metrics
    (distances, failure rates) are being extracted
  repo: https://github.com/mit-ll/spacegym-kspdg
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:32db006006e5bbf75a6a2c7bcca519efa74808fda00486221e1fcc597c31a2ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mit-ll/spacegym-kspdg
  tools:
  - GPT-3.5
  - Weights & Biases
  - pandas
  - spacegym-kspdg
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# benchmark-metric-extraction-from-papers

## Summary

Extract quantitative benchmark results (e.g. Best Distance, Average Distance, Failure Rate) from a paper's results tables and assemble them into a validated structured file such as gpt_finetuning_results.csv. Enables direct comparison of ablation configurations reported across rows of a published evaluation table.

## When to use

Use when a paper reports a benchmark comparison table (e.g. GPT-3.5 fine-tuning ablation rows in the KSPDG evaluation: Baseline GPT, Simple fine-tuning, + Hyperparam tuning, + System prompt, + Two train gameplays) and the analysis goal requires transcribing those metrics into machine-readable form.

## When NOT to use

- The paper reports training/validation losses (e.g. the LRM ablation with training loss 2.3e-2 and validation loss 2.8e-2) rather than evaluation benchmark metrics — these are separate tables requiring different extraction.
- The goal is to run the KSPDG evaluation environments yourself to obtain new metrics, rather than extract already-published numbers (spacegym-kspdg provides evaluation environments, not training ones).
- The metrics of interest come from raw wandb logs or LLaMA fine-tuning dataset files instead of a printed comparison table in the manuscript.

## Inputs

- Published paper text/PDF containing the benchmark results table (Table 5 of the KSPDG GPT-3.5 ablation)
- Row labels and metric columns: method name, Best Distance (m), Average Distance (m), Failure Rate
- Optional: Weights & Biases project logs for cross-validation of reported values

## Outputs

- gpt_finetuning_results.csv with columns method, best_distance_m, avg_distance_m, failure_rate
- One row per configuration, e.g. 'Simple fine-tuning, 263.55, 265.89, 0.0%' and '+ Two train gameplays, 132.09, 159.78, 0.2%'

## How to apply

Parse the paper's results section and locate the fine-tuning comparison table rows (here: Baseline GPT, Simple fine-tuning, + Hyperparam tuning, + System prompt, + Two train gameplays). For each row extract the Best Distance (m), Average Distance (m), and Failure Rate columns exactly as printed — e.g. Simple fine-tuning 263.55/265.89/0.0% and + Two train gameplays 132.09/159.78/0.2%. Assemble the rows with pandas into a CSV with columns method, best_distance_m, avg_distance_m, failure_rate. Validate each cell against the published Table 5 values (best distances 263.55, 188.90, 197.41, 132.09 for the four fine-tuning stages) before saving; correct any transcription drift before the file is reused. Optionally cross-check against the publicly accessible Weights & Biases project logs, since all experiments in this study were tracked there.

## Related tools

- **pandas** (Assembles the extracted table rows into a CSV with the method/best_distance_m/avg_distance_m/failure_rate schema)
- **Weights & Biases** (Publicly accessible experiment-tracking logs used to cross-check the metric values transcribed from the paper)
- **spacegym-kspdg** (Defines the KSPDG benchmark environments whose published evaluation metrics (distances, failure rates) are being extracted) — https://github.com/mit-ll/spacegym-kspdg

## Examples

```
import pandas as pd; rows=[["Simple fine-tuning",263.55,265.89,"0.0%"],["+ Two train gameplays",132.09,159.78,"0.2%"]]; pd.DataFrame(rows,columns=["method","best_distance_m","avg_distance_m","failure_rate"]).to_csv("gpt_finetuning_results.csv",index=False)
```

## Evaluation signals

- Every extracted cell matches the published Table 5 value exactly (best distances 263.55, 188.90, 197.41, 132.09 for the four fine-tuning stages).
- CSV schema check: exactly the columns method, best_distance_m, avg_distance_m, failure_rate with one row per configuration.
- Failure rates fall in the small reported range (0.0%–0.2%) and the lowest-distance configuration (+ Two train gameplays, 132.09/159.78) carries the highest failure rate (0.2%) — lower distance must not be silently equated with lower failure.
- Metric ordering is monotonic in a meaningful way: the final sequential configuration (132.09/159.78) improves on simple fine-tuning (263.55/265.89) with intermediates at 188.90/202.08 and 197.41/214.87.
- Cross-check at least one row against the publicly accessible Weights & Biases project logs.

## Limitations

- Fine-tuning hyperparameters (learning rate, epochs, LoRA rank) are not fully reported in the paper, so the extracted metrics cannot be tied to exact training configurations.
- The metrics are KSPDG-scenario-specific (pursuit-evasion distances in meters and failure rates); they do not generalize to other benchmark domains without re-extraction.
- Results in this study are hardware-dependent for locally trained LLaMA models; extracted numbers reflect the authors' setup only.
- Failure rate must be read as a separate column: the best-distance configuration (0.2%) is not the lowest-failure configuration (0.0%).

## Evidence

- [methods] Simple fine-tuning 263.55 265.89 0.0%: "Simple fine-tuning 263.55 265.89 0.0%"
- [methods] + Two train gameplays 132.09 159.78 0.2%: "+ Two train gameplays 132.09 159.78 0.2%"
- [results] All experiments tracked in Weights & Biases: "All experiments carried out during this investigation have been tracked using Weights & Biases (wandb)."
- [readme] KSPDG as benchmark for autonomous control: [evidence span withheld — anchor status: not_found]
