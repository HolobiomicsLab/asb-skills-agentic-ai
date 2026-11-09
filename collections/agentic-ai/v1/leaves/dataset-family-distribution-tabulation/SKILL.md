---
name: dataset-family-distribution-tabulation
description: Use when reproducing or auditing how benchmark instances are distributed
  across source dataset families (e.g., CausalNet, MedCaus, COPA) and experiment task-slice
  tags (2a-2h), or when validating that summed instance counts match reported table
  totals.
when_to_use_negative:
- The goal is model performance analysis (e.g., exact-match or similarity scoring
  of generations) rather than instance-count auditing — use the analysis/ scripts
  instead
- The input is model output from results/experiment2/ grouped by model and prompting
  strategy, not the source dataset CSVs with dataset-family columns
- The dataset CSVs lack the source dataset-family column needed for grouping, making
  per-family tallies undefined
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: Used with pandas to load data/exp1.csv and data/Experiment2.csv, group by
    source dataset family and task-slice tag (2a-2h), tally row counts, and combine
    counts into a single table tagged by experiment
  repo: https://github.com/sydneyanuyah/CausalDiscovery
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1109/bigdata66926.2025.11401449
    title: Benchmarking LLMs for Pairwise Causal Discovery in Biomedical and Multi-Domain
      Contexts
  source_content_hash: sha256:c64d73fa6759f7335f99c687c925074c4a6ff68ddbac503b346573d58be73e34
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sydneyanuyah/CausalDiscovery
  tools:
  - Python
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# Dataset-family distribution tabulation

## Summary

A pandas-based skill for tallying annotated benchmark instances per source dataset family and per task-slice tag, replicating the sample-count distribution behind Table XVIII of the pairwise causal discovery benchmark (3,008 detection instances in data/exp1.csv and 8,925 extraction instances in data/Experiment2.csv).

## When to use

Use when reproducing or auditing how benchmark instances are distributed across source dataset families (e.g., CausalNet, MedCaus, COPA) and experiment task-slice tags (2a-2h), or when validating that summed instance counts match reported table totals.

## When NOT to use

- The goal is model performance analysis (e.g., exact-match or similarity scoring of generations) rather than instance-count auditing — use the analysis/ scripts instead
- The input is model output from results/experiment2/ grouped by model and prompting strategy, not the source dataset CSVs with dataset-family columns
- The dataset CSVs lack the source dataset-family column needed for grouping, making per-family tallies undefined

## Inputs

- data/exp1.csv — Experiment 1 causal-detection dataset CSV (3,008 rows) from experiments/experiment1/
- data/Experiment2.csv — Experiment 2 consolidated causal-extraction dataset CSV (8,925 rows) from experiments/experiment2/, containing task slices 2a-2h

## Outputs

- A combined pandas table of row counts per source dataset family and per task-slice tag, annotated with experiment type (detection vs. extraction)
- Validation report confirming summed detection count = 3,008 and summed extraction count = 8,925, matching Table XVIII instance totals across the 12 datasets

## How to apply

Load the Experiment 1 causal-detection CSV (data/exp1.csv, 3,008 rows) and the Experiment 2 consolidated causal-extraction CSV (data/Experiment2.csv, 8,925 rows) into pandas DataFrames. Group the Experiment 1 dataframe by source dataset family (CausalNet, CausalProbe, SemEval2010Task8, MedCaus, CauseNet, FinCausal, CausalBench, CRASS, Coling22, ECI-B, PubMed, COPA) and tally row counts per family. Group the Experiment 2 dataframe by both source dataset family and task-slice tag (2a-2h) and tally row counts per combination. Combine the per-dataset and per-slice counts into a single table tagged by experiment (detection vs. extraction). The rationale is that the paper's Table XVIII reports instance totals across these 12 datasets, so per-family tallies must reconcile to the published totals.

## Related tools

- **Python** (Used with pandas to load data/exp1.csv and data/Experiment2.csv, group by source dataset family and task-slice tag (2a-2h), tally row counts, and combine counts into a single table tagged by experiment) — https://github.com/sydneyanuyah/CausalDiscovery

## Examples

```
import pandas as pd; exp1 = pd.read_csv('data/exp1.csv'); exp2 = pd.read_csv('data/Experiment2.csv'); print(exp1.groupby('dataset_family').size(), exp2.groupby(['dataset_family','task_slice']).size())
```

## Evaluation signals

- Summed Experiment 1 (detection) groupby counts equal exactly 3,008 rows
- Summed Experiment 2 (extraction) groupby counts equal exactly 8,925 rows
- Every row in the combined table maps to one of the 12 dataset families (CausalNet, CausalProbe, SemEval2010Task8, MedCaus, CauseNet, FinCausal, CausalBench, CRASS, Coling22, ECI-B, PubMed, COPA) and, for Experiment 2, to one of the 8 task-slice tags 2a-2h
- Per-family and per-slice totals reconcile with the instance totals reported in Table XVIII across the 12 datasets
- The combined table distinguishes detection vs. extraction experiment tags so the two skills' counts are not mixed

## Limitations

- Validation depends on the consolidated CSVs matching the published row counts (3,008 and 8,925); any upstream filtering or subset export will break reconciliation with Table XVIII
- Task-slice tags (2a-2h) exist only in the Experiment 2 extraction data; detection instances in data/exp1.csv cannot be tabulated by slice
- The discussion/limitations section of the paper was not extractable, so additional caveats about dataset balancing may be undocumented

## Evidence

- [other] Load the Experiment 1 causal-detection dataset CSV (data/exp1.csv, 3,008 rows) and the Experiment 2 consolidated causal-extraction dataset CSV (data/Experiment2.csv, 8,925 rows) into pandas DataFrames.: [evidence span withheld — anchor status: not_found]
- [other] verify the summed detection count equals 3,008 and the summed extraction count equals 8,925: [evidence span withheld — anchor status: not_found]
- [readme] The dataset file contains 3,008 rows.: "`experiments/experiment1/` contains the causal-detection data, prompt definitions, inference script, and model outputs. The dataset file contains 3,008 rows."
- [readme] The consolidated dataset file contains 8,925 rows.: "`experiments/experiment2/` contains the causal-extraction data, task slices 2a-2h, prompt definitions, inference scripts, and shell launch files. The consolidated dataset file contains 8,925 rows."
- [readme] The experiment files contain examples from the following dataset families: "The experiment files contain examples from the following dataset families:"
