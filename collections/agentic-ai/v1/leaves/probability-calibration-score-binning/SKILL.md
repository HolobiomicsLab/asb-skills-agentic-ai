---
name: probability-calibration-score-binning
description: Use when scoring model-extracted cause-effect pairs against gold pairs
  and a continuous similarity score (cosine) must be converted into a calibrated per-pair
  correctness probability via binned empirical calibration.
when_to_use_negative:
- Evaluating the causal-detection task (Experiment 1), which uses exact-match scoring
  on binary causal-link labels rather than pair-level similarity calibration
- The task requires strict exact string match of cause-effect spans; the similarity-based
  probabilistic map is designed for partial/semantic credit, not exact-match metrics
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3345
tools:
- name: Python
  role: Implements the similarity-matrix computation, Hungarian matching, band binning,
    and score-map calibration pipeline
- name: scipy.optimize.linear_sum_assignment
  role: Solves the 1:1 optimal assignment (Hungarian algorithm) between predicted
    and gold cause-effect pairs on the cosine matrix
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1109/bigdata66926.2025.11401449
    title: Benchmarking LLMs for Pairwise Causal Discovery in Biomedical and Multi-Domain
      Contexts
  source_content_hash: sha256:6546fc4d044a6eae6adbf169e98601eb97ee58430355957fe26df27afc2915f8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - scipy.optimize.linear_sum_assignment
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3345
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# probability-calibration-score-binning

## Summary

Reconstructs the paper's SentenceLM-based extraction scorer: predicted and gold cause-effect pairs from Experiment 2 (task slices 2a-2h) are matched 1:1 via iterative Hungarian assignment on cosine similarities, and matched scores are binned into 10 bands that map to calibrated correctness probabilities (Table 6), with the >=0.9 band anchored to 0.99.

## When to use

Use when scoring model-extracted cause-effect pairs against gold pairs and a continuous similarity score (cosine) must be converted into a calibrated per-pair correctness probability via binned empirical calibration.

## When NOT to use

- Evaluating the causal-detection task (Experiment 1), which uses exact-match scoring on binary causal-link labels rather than pair-level similarity calibration
- The task requires strict exact string match of cause-effect spans; the similarity-based probabilistic map is designed for partial/semantic credit, not exact-match metrics

## Inputs

- Predicted cause-effect pairs from evaluated model outputs in results/experiment2/ (CSV, grouped by model and prompting strategy)
- Gold cause-effect pairs from the consolidated causal-extraction dataset in experiments/experiment2/ (8,925 rows; task slices 2a-2h)
- SentenceLM sentence-transformer model for span encoding
- 4,000 human-labeled calibration samples (400 per cosine band, seed=4000)

## Outputs

- calibrated_score_map.csv with exactly 10 banded rows mapping cosine bands to correctness probabilities
- Matched-pair similarity scores written to analysis/ similarity-scoring outputs
- Calibrated probabilistic score map reproducing Table 6

## How to apply

Load the predicted cause-effect pairs from results/experiment2/ model outputs and the gold pairs from the consolidated 8,925-row causal-extraction dataset in experiments/experiment2/ (task slices 2a-2h) using pandas. Encode cause-effect spans with a SentenceLM sentence-transformer and compute the full cosine similarity matrix between predicted and gold pairs. Run iterative 1:1 optimal assignment with scipy.optimize.linear_sum_assignment (Hungarian algorithm) so each predicted pair aligns to at most one gold pair. Bin matched cosine scores into 10 equal-width bands spanning 0 to 1, then calibrate a per-band empirical correctness probability (from 4,000 human-labeled samples, 400 per band, seed=4000), anchoring the >=0.9 band to 0.99 to reproduce Table 6. Save the band-to-probability map as calibrated_score_map.csv and write matched-pair scores to the analysis/ similarity-scoring outputs.

## Related tools

- **Python** (Implements the similarity-matrix computation, Hungarian matching, band binning, and score-map calibration pipeline)
- **scipy.optimize.linear_sum_assignment** (Solves the 1:1 optimal assignment (Hungarian algorithm) between predicted and gold cause-effect pairs on the cosine matrix)

## Examples

```
from scipy.optimize import linear_sum_assignment; rows, cols = linear_sum_assignment(-cosine_matrix); band = min(int(sim // 0.1), 9); prob = score_map[band]
```

## Evaluation signals

- calibrated_score_map.csv exists and contains exactly 10 banded rows
- The highest band (cosine >= 0.9) carries the calibrated probability 0.99
- Every matched predicted pair is assigned to at most one gold pair (valid 1:1 Hungarian assignment)
- Band probabilities are monotonically consistent with per-band empirical correctness means from the 4,000-sample calibration set (400 per band, seed=4000)
- Matched-pair scores are present in the analysis/ similarity-scoring output files

## Limitations

- Calibration is empirical: band probabilities are derived from 4,000 human-labeled samples (400 per band), so bands with sparse support may be noisy
- Extraction performance is low overall (best extraction model Qwen2.5-Coder-32B-Instruct reached 47.12% C_extract), meaning many predicted pairs will fall in low-similarity bands and remain unmatched
- Scores depend on the choice of SentenceLM sentence-transformer encoding of the spans

## Evidence

- [methods] SentenceLM cosine similarity between predicted and gold cause-effect pairs, iterative Hungarian 1:1 matching, bins cosine scores into 10 bands: [evidence span withheld — anchor status: not_found]
- [methods] per-band means calibrated on 4,000 human-labeled samples (400 per band, seed=4000): [evidence span withheld — anchor status: not_found]
- [methods] analysis/ contains exact-match, similarity-scoring, problem-case, and error-bucket files: "`analysis/` contains exact-match, similarity-scoring, problem-case, and error-bucket files."
- [readme] experiments/experiment2/ contains the causal-extraction data, task slices 2a-2h ... The consolidated dataset file contains 8,925 rows: "`experiments/experiment2/` contains the causal-extraction data, task slices 2a-2h, prompt definitions, inference scripts, and shell launch files. The consolidated dataset file contains 8,925 rows."
- [readme] The scripts use seed `4000`.: "The scripts use seed `4000`."
