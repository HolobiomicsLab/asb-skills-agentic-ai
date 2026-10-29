---
name: causal-extraction-error-categorization
description: Use when you have LLM extraction outputs for cause-effect pairs (e.g.,
  results/experiment2/ outputs for Qwen2.5-Coder-32B-Instruct) and need to bucketize
  failures to identify actionable error sources.
when_to_use_negative:
- The task is causal detection (binary yes/no decision, Experiment 1) rather than
  cause-effect phrase extraction (Experiment 2)
- Model outputs have not yet been evaluated against gold pairs — run the exact-match/similarity-scoring
  evaluation first
- The goal is aggregate extraction accuracy (C_extract) rather than fine-grained failure-mode
  analysis
edam_operation: http://edamontology.org/operation_2945
tools:
- name: Python
  role: pandas-based bucket assignment and value_counts(normalize=True) computation
    of error-bucket percentages
  repo: https://github.com/sydneyanuyah/CausalDiscovery
- name: Qwen2.5-Coder-32B-Instruct
  role: source model whose causal-extraction outputs (2a-2h) are bucketized; best
    extraction model at 47.12% C_extract
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1109/bigdata66926.2025.11401449
    title: Benchmarking LLMs for Pairwise Causal Discovery in Biomedical and Multi-Domain
      Contexts
  source_content_hash: sha256:d85dda8efbdd15a91568aa1a32f5e874a7895608e52017db52abd4202d9aca93
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sydneyanuyah/CausalDiscovery
  tools:
  - Python
  - Qwen2.5-Coder-32B-Instruct
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# causal-extraction-error-categorization

## Summary

Categorizes LLM cause-effect extraction failures into the paper's Table 7 error buckets (Correct, Missing, Incorrect, Hallucinated data, Switch CE, False Neg) by comparing predicted pairs against gold annotations for Qwen2.5-Coder-32B-Instruct on the causal extraction task slices 2a-2h.

## When to use

Use when you have LLM extraction outputs for cause-effect pairs (e.g., results/experiment2/ outputs for Qwen2.5-Coder-32B-Instruct) and need to bucketize failures to identify actionable error sources.

## When NOT to use

- The task is causal detection (binary yes/no decision, Experiment 1) rather than cause-effect phrase extraction (Experiment 2)
- Model outputs have not yet been evaluated against gold pairs — run the exact-match/similarity-scoring evaluation first
- The goal is aggregate extraction accuracy (C_extract) rather than fine-grained failure-mode analysis

## Inputs

- Evaluated extraction outputs for Qwen2.5-Coder-32B-Instruct from results/experiment2/, grouped by model and prompting strategy
- Gold cause-effect annotations from the consolidated Experiment 2 dataset (experiments/experiment2/, 8,925 rows, task slices 2a-2h)

## Outputs

- error_bucket_percentages.csv containing per-bucket percentages (Incorrect 6.07%, Hallucinated data 4.47%, Switch CE 4.06%, Correct 45.55%, Missing 35.70%, False Neg 3.22%)

## How to apply

Load the evaluated extraction outputs from results/experiment2/, grouped by model and prompting strategy, together with the gold cause-effect annotations from the consolidated Experiment 2 dataset (8,925 rows spanning task slices 2a-2h). Compare each predicted cause-effect pair against the gold pair using exact-match and similarity-scoring logic consistent with the analysis/ scripts. Assign each example to one bucket — Incorrect, Hallucinated data, Switch CE (cause/effect swapped), False Negative, Missing, or Correct — based on the match outcome. Compute per-bucket percentages with pandas (value_counts normalized to the total) and write the table to error_bucket_percentages.csv. Validation requires reproducing the Table 7 distribution: Correct 45.55%, Missing 35.70%, Incorrect 6.07%, Hallucinated data 4.47%, Switch CE 4.06%, False Neg 3.22%.

## Related tools

- **Python** (pandas-based bucket assignment and value_counts(normalize=True) computation of error-bucket percentages) — https://github.com/sydneyanuyah/CausalDiscovery
- **Qwen2.5-Coder-32B-Instruct** (source model whose causal-extraction outputs (2a-2h) are bucketized; best extraction model at 47.12% C_extract)

## Examples

```
python -c "import pandas as pd; df=pd.read_csv('evaluated_outputs.csv'); df['bucket'].value_counts(normalize=True).to_csv('error_bucket_percentages.csv')"
```

## Evaluation signals

- Reproduced bucket percentages match Table 7 exactly: Correct 45.55%, Missing 35.70%, Incorrect 6.07%, Hallucinated data 4.47%, Switch CE 4.06%, False Neg 3.22%
- Sum of all bucket percentages equals 100% of the evaluated examples
- Switch CE bucket contains only examples where the predicted cause and effect phrases are swapped relative to the gold pair
- Output CSV (error_bucket_percentages.csv) contains one row per bucket with a percentage column
- Actionable error sources are dominated by Incorrect, Hallucinated data, and Switch CE, with false positives and other categories rare

## Limitations

- Bucketization depends on exact-match and similarity-scoring logic; borderline paraphrase matches can shift examples between Incorrect and Correct buckets
- Distribution is specific to Qwen2.5-Coder-32B-Instruct and the 8,925-row consolidated Experiment 2 dataset; other models and prompting strategies will have different bucket profiles
- Performance plummets for implicit relationships, links spanning multiple sentences, and texts with multiple causal pairs, so bucket rates may vary sharply by dataset family

## Evidence

- [methods] analysis/ contains exact-match, similarity-scoring, problem-case, and error-bucket files.: "`analysis/` contains exact-match, similarity-scoring, problem-case, and error-bucket files."
- [readme] results/experiment2/ contains evaluated outputs grouped by model and prompting strategy.: "`results/experiment2/` contains evaluated outputs and timing files grouped by model and prompting strategy."
- [readme] the causal-extraction data, task slices 2a-2h, and the consolidated dataset of 8,925 rows: "`experiments/experiment2/` contains the causal-extraction data, task slices 2a-2h, prompt definitions, inference scripts, and shell launch files."
- [abstract] the best for extraction, Qwen2.5-Coder-32B-Instruct, reached just 47.12% (C_extract): "the best for extraction, Qwen2.5-Coder-32B-Instruct, reached just 47.12\% ($C_{extract}$)"
