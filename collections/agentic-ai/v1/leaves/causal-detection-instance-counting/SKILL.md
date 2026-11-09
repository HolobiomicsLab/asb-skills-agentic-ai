---
name: causal-detection-instance-counting
description: Use when you need to verify or recompute how many annotated cause–effect
  instances the benchmark curated per dataset family (CausalNet, CausalProbe, SemEval2010Task8,
  MedCaus, CauseNet, FinCausal, CausalBench, CRASS, Coling22, ECI-B, PubMed, COPA)
  and per task-slice tag 2a–2h.
when_to_use_negative:
- You need model performance or generation counts (580,125 generations across 13 models
  × 5 prompting styles) rather than curated instance counts — this skill only tallies
  input rows.
- The CSVs have been re-split or filtered (e.g., per-prompt-strategy subsets in results/experiment2/),
  so raw row counts would no longer match the 3,008 / 8,925 benchmark totals.
- You are counting evaluation outputs in analysis/ (exact-match, similarity-scoring,
  error-bucket files) — those are model-output artifacts, not curated instances.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: pandas-based grouping and tallying of rows by source dataset family and task-slice
    tag across both CSVs
  repo: https://github.com/sydneyanuyah/CausalDiscovery
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1109/bigdata66926.2025.11401449
    title: Benchmarking LLMs for Pairwise Causal Discovery in Biomedical and Multi-Domain
      Contexts
  source_content_hash: sha256:e5bd24008e1ce3994712252d7aeb183bde241f50c2a51c939d99ea0ba0837b82
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sydneyanuyah/CausalDiscovery
  tools:
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# causal-detection-instance-counting

## Summary

Reproduce the per-source-dataset and per-task-slice instance-count distribution of the pairwise causal discovery benchmark by tallying rows in the causal-detection CSV (data/exp1.csv, 3,008 rows) and the consolidated causal-extraction CSV (data/Experiment2.csv, 8,925 rows) with pandas.

## When to use

Use when you need to verify or recompute how many annotated cause–effect instances the benchmark curated per dataset family (CausalNet, CausalProbe, SemEval2010Task8, MedCaus, CauseNet, FinCausal, CausalBench, CRASS, Coling22, ECI-B, PubMed, COPA) and per task-slice tag 2a–2h.

## When NOT to use

- You need model performance or generation counts (580,125 generations across 13 models × 5 prompting styles) rather than curated instance counts — this skill only tallies input rows.
- The CSVs have been re-split or filtered (e.g., per-prompt-strategy subsets in results/experiment2/), so raw row counts would no longer match the 3,008 / 8,925 benchmark totals.
- You are counting evaluation outputs in analysis/ (exact-match, similarity-scoring, error-bucket files) — those are model-output artifacts, not curated instances.

## Inputs

- experiments/experiment1/data/exp1.csv — causal-detection dataset, 3,008 rows
- experiments/experiment2/data/Experiment2.csv — consolidated causal-extraction dataset, 8,925 rows, with task-slice tags 2a–2h and a source-dataset-family column

## Outputs

- Per-dataset-family row-count table for the detection experiment (summing to 3,008)
- Per-dataset-family × per-task-slice (2a–2h) row-count table for the extraction experiment (summing to 8,925)
- Combined count table tagged by experiment type (detection vs. extraction) covering the 12 dataset families

## How to apply

Load data/exp1.csv (3,008 rows) and data/Experiment2.csv (8,925 rows) into pandas DataFrames. Group the detection dataframe by source dataset family and tally row counts; group the extraction dataframe jointly by source dataset family and task-slice tag (2a–2h) and tally counts per combination. Combine the per-dataset and per-slice tallies into one table tagged by experiment (causal detection vs. causal extraction) so the 12 dataset families are represented across both skills. Validate that summed detection counts equal 3,008 and summed extraction counts equal 8,925, matching the paper's Table XVIII instance totals; the extraction instances (8,925) underpin 13 models × 5 prompting styles = 580,125 generations in the study.

## Related tools

- **Python** (pandas-based grouping and tallying of rows by source dataset family and task-slice tag across both CSVs) — https://github.com/sydneyanuyah/CausalDiscovery

## Examples

```
import pandas as pd; e1 = pd.read_csv('data/exp1.csv'); e2 = pd.read_csv('data/Experiment2.csv'); assert len(e1) == 3008 and len(e2) == 8925; print(e2.groupby(['source_dataset','task_slice']).size())
```

## Evaluation signals

- Summed detection-families counts equal exactly 3,008 rows.
- Summed extraction (tags 2a–2h) counts equal exactly 8,925 rows.
- The combined table tags each count by experiment (detection vs. extraction) and covers all 12 dataset families (CausalNet, CausalProbe, SemEval2010Task8, MedCaus, CauseNet, FinCausal, CausalBench, CRASS, Coling22, ECI-B, PubMed, COPA).
- Extraction slice tallies resolve to exactly 8 task-slice groups (2a–2h).
- Reported instance totals are consistent with the derived generation count 13 models × 5 prompting styles = 580,125 generations.

## Limitations

- Counts depend on the consolidated CSVs as distributed; no per-slice mapping beyond tags 2a–2h is documented, so re-slicing beyond those tags cannot be validated.
- The README confirms only the two row totals (3,008 and 8,925); per-family breakdowns must be reconstructed from the CSV columns themselves.
- Discussion/limitations text is absent from the extracted article, so no additional caveats or known distributional edge cases are documented.

## Evidence

- [readme] The dataset file contains 3,008 rows.: "experiments/experiment1/ contains the causal-detection data, prompt definitions, inference script, and model outputs. The dataset file contains 3,008 rows."
- [readme] The consolidated dataset file contains 8,925 rows.: "experiments/experiment2/ contains the causal-extraction data, task slices 2a-2h, prompt definitions, inference scripts, and shell launch files. The consolidated dataset file contains 8,925 rows."
- [abstract] 12 diverse datasets across two skills: [evidence span withheld — anchor status: not_found]
- [methods] Experiment 2 inference script consumes data/Experiment2.csv: "python exp2.py \
  --model MODEL_NAME_OR_PATH \
  --data data/Experiment2.csv \
  --exp_folder OUTPUT_FOLDER"
