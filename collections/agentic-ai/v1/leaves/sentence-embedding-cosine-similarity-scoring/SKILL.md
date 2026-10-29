---
name: sentence-embedding-cosine-similarity-scoring
description: Use when comparing predicted cause-effect pairs against gold pairs from
  causal extraction outputs (e.g., results/experiment2/ vs. the 8,925-row consolidated
  dataset) and exact-match scoring is too strict.
when_to_use_negative:
- The task is causal detection (Experiment 1), which uses exact-match scoring on labels
  rather than span-level similarity matching
- Only one predicted pair and one gold pair exist per text, making Hungarian assignment
  unnecessary (plain cosine similarity suffices)
tools:
- name: Python
  role: 'Implements the similarity-scoring pipeline: pandas loading, SentenceLM encoding,
    scipy.optimize.linear_sum_assignment matching, banding, and calibration'
  repo: https://github.com/sydneyanuyah/CausalDiscovery
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1109/bigdata66926.2025.11401449
    title: Benchmarking LLMs for Pairwise Causal Discovery in Biomedical and Multi-Domain
      Contexts
  source_content_hash: sha256:a4d6a06d15e67acd6b574fc828207e781f9ad3c9707f77d55b11908cfedfa9fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sydneyanuyah/CausalDiscovery
  tools:
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# sentence-embedding-cosine-similarity-scoring

## Summary

Scores model-extracted cause-effect spans against gold pairs in extraction tasks 2a-2h by computing SentenceLM cosine similarity, applying iterative Hungarian 1:1 matching, binning scores into 10 bands, and calibrating a probabilistic score map (Table 6) on 4,000 human-labeled samples.

## When to use

Use when comparing predicted cause-effect pairs against gold pairs from causal extraction outputs (e.g., results/experiment2/ vs. the 8,925-row consolidated dataset) and exact-match scoring is too strict.

## When NOT to use

- The task is causal detection (Experiment 1), which uses exact-match scoring on labels rather than span-level similarity matching
- Only one predicted pair and one gold pair exist per text, making Hungarian assignment unnecessary (plain cosine similarity suffices)

## Inputs

- Predicted cause-effect pairs from evaluated model outputs in results/experiment2/ (grouped by model and prompting strategy)
- Gold cause-effect pairs from the consolidated causal-extraction dataset (8,925 rows) in experiments/experiment2/, task slices 2a-2h
- 4,000 human-labeled similarity samples (400 per band, seed=4000) for calibration

## Outputs

- calibrated_score_map.csv (10 banded rows mapping cosine bands to correctness probabilities, top band >= 0.9 mapped to 0.99)
- Matched-pair similarity scores in the analysis/ similarity-scoring output files

## How to apply

Load predicted pairs from evaluated model outputs in results/experiment2/ and gold pairs from the consolidated causal-extraction dataset (8,925 rows) in experiments/experiment2/ with pandas. Encode cause-effect spans with a SentenceLM sentence-transformer and compute the full cosine similarity matrix between predicted and gold pairs. Run iterative 1:1 optimal assignment with scipy.optimize.linear_sum_assignment (Hungarian algorithm) so each predicted pair aligns with at most one gold pair. Bin matched cosine scores into 10 equal-width bands spanning 0 to 1, then calibrate each band's empirical correctness probability from 4,000 human-labeled samples (400 per band, seed=4000), anchoring the >=0.9 band at 0.99 to reproduce Table 6. Write the band-to-probability map to calibrated_score_map.csv and matched-pair scores to the analysis/ similarity-scoring outputs.

## Related tools

- **Python** (Implements the similarity-scoring pipeline: pandas loading, SentenceLM encoding, scipy.optimize.linear_sum_assignment matching, banding, and calibration) — https://github.com/sydneyanuyah/CausalDiscovery

## Examples

```
from scipy.optimize import linear_sum_assignment; import numpy as np; rows, cols = linear_sum_assignment(-cosine_matrix); scores = cosine_matrix[rows, cols]
```

## Evaluation signals

- calibrated_score_map.csv exists and contains exactly 10 banded rows
- The highest band (>= 0.9) carries the calibrated probability 0.99
- Matched-pair scores are present in the analysis/ similarity-scoring output files
- Each predicted pair is matched to at most one gold pair (1:1 assignment from linear_sum_assignment)
- All cosine similarity scores fall within the 0-to-1 banding range

## Limitations

- The probabilistic score map is calibrated on 4,000 human-labeled samples (400 per band); probabilities for rare bands may be noisy
- Similarity scoring complements but does not replace exact-match scoring; the repo keeps both as separate analysis/ outputs
- The paper reports extraction performance dropped for implicit relationships, multi-sentence links, and texts with multiple causal pairs, which can degrade matched-pair quality

## Evidence

- [methods] analysis/ contains exact-match, similarity-scoring, problem-case, and error-bucket files.: "`analysis/` contains exact-match, similarity-scoring, problem-case, and error-bucket files."
- [methods] The scripts use seed 4000.: "The scripts use seed `4000`."
- [readme] results/experiment2/ contains evaluated outputs grouped by model and prompting strategy.: "`results/experiment2/` contains evaluated outputs and timing files grouped by model and prompting strategy."
- [readme] experiments/experiment2/ contains causal-extraction data, task slices 2a-2h, and the consolidated dataset file contains 8,925 rows.: "`experiments/experiment2/` contains the causal-extraction data, task slices 2a-2h, prompt definitions, inference scripts, and shell launch files. The consolidated dataset file contains 8,925 rows."
