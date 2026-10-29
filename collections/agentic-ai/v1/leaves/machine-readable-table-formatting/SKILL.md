---
name: machine-readable-table-formatting
description: Use when a results section reports a comparison table with fixed column
  metrics (e.g. Distance (m), Average Distance (m), Failure Rate) across method rows,
  and the values must be extracted cell-exactly into a structured file.
when_to_use_negative:
- The source is an unstructured narrative result statement with no tabular row/column
  structure to map to a schema
- The table contains derived statistics (means, confidence intervals) rather than
  per-configuration raw reported values, where cell-exact transcription would misrepresent
  uncertainty
- Values are only available in a figure/plot rather than as printed numeric text
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3365
tools:
- name: pandas
  role: Assemble extracted table rows into a validated CSV dataframe
- name: Weights & Biases
  role: Cross-check experiment tracking logs used to source/verify reported table
    values
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:f0f201609593c92c0dced701789b0e4325389c3a3f96f0baa9069c7a49b0f027
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GPT-3.5
  - Weights & Biases
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3365
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# machine-readable-table-formatting

## Summary

Convert a paper's reported results table (here, the sequential GPT-3.5 fine-tuning ablation rows from the KSPDG spacecraft-control study) into a validated machine-readable CSV with explicit method/metric columns, enabling downstream programmatic analysis.

## When to use

Use when a results section reports a comparison table with fixed column metrics (e.g. Best Distance (m), Average Distance (m), Failure Rate) across method rows, and the values must be extracted cell-exactly into a structured file.

## When NOT to use

- The source is an unstructured narrative result statement with no tabular row/column structure to map to a schema
- The table contains derived statistics (means, confidence intervals) rather than per-configuration raw reported values, where cell-exact transcription would misrepresent uncertainty
- Values are only available in a figure/plot rather than as printed numeric text

## Inputs

- Paper results/methods section containing the fine-tuning comparison table (Table 5) rows as raw text
- Column headers: method, Best Distance (m), Average Distance (m), Failure Rate

## Outputs

- gpt_finetuning_results.csv with columns method, best_distance_m, avg_distance_m, failure_rate and five rows (baseline + four fine-tuning stages)

## How to apply

Parse the results section to locate the comparison table rows — for this paper: Baseline GPT, Simple fine-tuning, + Hyperparam tuning, + System prompt, + Two train gameplays. Extract each row's Best Distance (m), Average Distance (m), and Failure Rate columns, e.g. Simple fine-tuning yields 263.55 / 265.89 / 0.0% and + Two train gameplays yields 132.09 / 159.78 / 0.2%. Assemble the rows with pandas into a CSV with columns method, best_distance_m, avg_distance_m, failure_rate (normalizing percentages to numeric form). Validate each cell against the reported Table 5 values exactly — best distances 263.55, 188.90, 197.41, 132.09 for the four fine-tuning stages — before saving gpt_finetuning_results.csv. The rationale is that KSPDG performance is scored by lower best/average distance plus low failure rate, so transcribing values verbatim (including the counterintuitive result that the final configuration 132.09/159.78 improves on the simple fine-tuning baseline 263.55/265.89) preserves the paper's sequential ablation ordering for reuse.

## Related tools

- **pandas** (Assemble extracted table rows into a validated CSV dataframe)
- **Weights & Biases** (Cross-check experiment tracking logs used to source/verify reported table values)

## Examples

```
import pandas as pd; df = pd.read_csv('gpt_finetuning_results.csv'); assert df['best_distance_m'].tolist() == [263.55, 188.90, 197.41, 132.09]
```

## Evaluation signals

- Each CSV cell matches the reported Table 5 value exactly: best distances 263.55, 188.90, 197.41, 132.09 and avg distances 265.89, 202.08, 214.87, 159.78 across the four fine-tuning stages
- Failure-rate column contains all four stage values (0.0%, 0.1%, 0.0%, 0.2%) normalized to numeric form
- Row count equals the number of table rows (baseline GPT plus four fine-tuning configurations) with no dropped or reordered stages
- Monotonic ablation ordering is preserved: simple fine-tuning 263.55 -> +LRM 0.2 tuning 188.90 -> +system prompt 197.41 -> +two gameplays 132.09
- CSV parses cleanly with pandas dtypes: method as string, distance columns as float, failure_rate as float

## Limitations

- The paper does not report fine-tuning hyperparameters (learning rate, epochs, LoRA rank), so the table alone cannot explain why intermediate configurations fluctuate non-monotonically (188.90 -> 197.41)
- Failure-rate differences are very small (0.0% to 0.2%), so the CSV comparison should weight best/avg distance as the primary metric
- Lower-distance scores are better in KSPDG pursuit-evaluation, which is counterintuitive if naively treating larger numbers as 'improved' — direction of improvement must be documented with the table

## Evidence

- [methods] Sequential fine-tuning ablation rows: "Simple fine-tuning 263.55 265.89 0.0%"
- [methods] Final configuration row: "+ Two train gameplays 132.09 159.78 0.2%"
- [methods] Intermediate hyperparameter tuning row: "+ Hyperparam tuning 188.90 202.08 0.1%"
- [results] Experiment tracking context: "All experiments carried out during this investigation have been tracked using Weights & Biases (wandb)."
- [other] Table extraction workflow: [evidence span withheld — anchor status: not_found]
