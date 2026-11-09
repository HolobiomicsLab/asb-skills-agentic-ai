---
name: causal-pair-extraction-evaluation
description: Use when evaluating extraction-task model outputs in results/experiment2/
  against a consolidated gold cause-effect pair dataset and you need similarity-based,
  probability-calibrated pair scores rather than exact matches.
when_to_use_negative:
- Scoring the causal-detection task (Experiment 1), which is a binary detection problem
  scored with exact-match metrics rather than pair matching
- When exact-match scoring of extracted pairs is desired — analysis/ provides separate
  exact-match files and this skill addresses similarity-based scoring
- When the evaluation target is not cause-effect spans (e.g., error-bucket or problem-case
  analysis outputs)
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: 'Implements the scoring pipeline: pandas loading of predicted/gold pairs,
    SentenceLM cosine similarity matrix, Hungarian assignment via scipy.optimize.linear_sum_assignment,
    and band calibration'
  repo: https://github.com/sydneyanuyah/CausalDiscovery
- name: Qwen2.5-Coder-32B-Instruct
  role: Best extraction model whose predicted pairs in results/experiment2/ are scored
    by this pipeline (47.12% C_extract)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1109/bigdata66926.2025.11401449
    title: Benchmarking LLMs for Pairwise Causal Discovery in Biomedical and Multi-Domain
      Contexts
  source_content_hash: sha256:78ce484aacec36c4ffddc1acbe158aafe01a4ba677a261f09c16a256446c42c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sydneyanuyah/CausalDiscovery
  tools:
  - Python
  - Qwen2.5-Coder-32B-Instruct
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# causal-pair-extraction-evaluation

## Summary

Scores model-extracted cause-effect pairs from the causal-extraction experiments (2a-2h) against gold pairs using SentenceLM cosine similarity, iterative Hungarian 1:1 matching, and a 10-band probabilistic score map (Table 6) calibrated on 4,000 human-labeled samples with seed 4000.

## When to use

Use when evaluating extraction-task model outputs in results/experiment2/ against a consolidated gold cause-effect pair dataset and you need similarity-based, probability-calibrated pair scores rather than exact matches.

## When NOT to use

- Scoring the causal-detection task (Experiment 1), which is a binary detection problem scored with exact-match metrics rather than pair matching
- When exact-match scoring of extracted pairs is desired — analysis/ provides separate exact-match files and this skill addresses similarity-based scoring
- When the evaluation target is not cause-effect spans (e.g., error-bucket or problem-case analysis outputs)

## Inputs

- Predicted cause-effect pairs from evaluated model outputs in results/experiment2/ (grouped by model and prompting strategy)
- Gold cause-effect pairs from the consolidated causal-extraction dataset (8,925 rows) in experiments/experiment2/, including task slices 2a-2h

## Outputs

- calibrated_score_map.csv (10 banded rows mapping cosine bands to correctness probabilities)
- Matched-pair similarity scores written to the analysis/ similarity-scoring outputs

## How to apply

Load predicted cause-effect pairs from model outputs in results/experiment2/ and gold pairs from the consolidated causal-extraction dataset (8,925 rows) in experiments/experiment2/ using pandas. Encode the cause-effect spans with a SentenceLM sentence-transformer model and compute the full cosine similarity matrix between predicted and gold pairs. Run iterative 1:1 optimal assignment with the Hungarian algorithm (scipy.optimize.linear_sum_assignment) on the cosine matrix so each predicted pair aligns with at most one gold pair. Bin the matched cosine scores into 10 equal-width bands spanning 0 to 1, then assign each band an empirical correctness probability anchored so the >= 0.9 band maps to 0.99, reproducing Table 6; the per-band means were calibrated on 4,000 human-labeled samples (400 per band, seed=4000). Write the calibrated band-to-probability map to calibrated_score_map.csv and supporting matched-pair scores to the analysis/ similarity-scoring outputs.

## Related tools

- **Python** (Implements the scoring pipeline: pandas loading of predicted/gold pairs, SentenceLM cosine similarity matrix, Hungarian assignment via scipy.optimize.linear_sum_assignment, and band calibration) — https://github.com/sydneyanuyah/CausalDiscovery
- **Qwen2.5-Coder-32B-Instruct** (Best extraction model whose predicted pairs in results/experiment2/ are scored by this pipeline (47.12% C_extract))

## Examples

```
from scipy.optimize import linear_sum_assignment; rows, cols = linear_sum_assignment(-cosine_matrix)
```

## Evaluation signals

- calibrated_score_map.csv exists with exactly 10 banded rows
- The highest band (>= 0.9) carries the calibrated probability 0.99, reproducing Table 6
- Hungarian assignment yields a 1:1 mapping — each predicted pair matched to at most one gold pair
- Matched cosine similarity scores fall within 10 equal-width bands spanning 0 to 1
- Band probabilities agree with per-band means calibrated on 4,000 human-labeled samples (400 per band, seed=4000)

## Limitations

- Calibration of the 10-band score map depends on the fixed seed 4000 and the 400-samples-per-band human-labeled sample set; different sampling may shift band probabilities
- Similarity scoring is specific to extraction task slices 2a-2h; applying it outside those slices is unsupported by the article
- The repository's analysis/ outputs also include problem-case and error-bucket files, indicating matched similarity scores alone do not capture all extraction failure modes

## Evidence

- [methods] Hungarian matching and 10-band score map: [evidence span withheld — anchor status: not_found]
- [methods] Calibration sample size and seed: [evidence span withheld — anchor status: not_found]
- [readme] Similarity-scoring analysis outputs: "`analysis/` contains exact-match, similarity-scoring, problem-case, and error-bucket files."
- [readme] Consolidated extraction dataset: "The consolidated dataset file contains 8,925 rows."
