---
name: optimal-bipartite-hungarian-matching
description: Use when scoring model-extracted cause-effect spans against gold pairs
  (tasks 2a-2h) and each predicted pair may match multiple gold pairs, so an optimal
  1:1 assignment is needed instead of greedy nearest-neighbor matching.
when_to_use_negative:
- Task is causal detection (Experiment 1), which uses label classification and exact-match-style
  scoring rather than span-level pair matching
- Only one predicted pair exists per text instance, so a greedy similarity comparison
  suffices and bipartite assignment adds no value
- You need exact-match scoring of extraction spans — the analysis/ directory already
  provides separate exact-match files for that purpose
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: Implements the similarity matrix, Hungarian assignment via scipy.optimize.linear_sum_assignment,
    and 10-band calibration of the score map
- name: Qwen2.5-Coder-32B-Instruct
  role: Best extraction model (47.12% C_extract) whose predicted cause-effect pairs
    serve as one side of the matching bipartition
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1109/bigdata66926.2025.11401449
    title: Benchmarking LLMs for Pairwise Causal Discovery in Biomedical and Multi-Domain
      Contexts
  source_content_hash: sha256:56fbf90fd8dce38a2f1ec742d8a36571431552765b8d9db1637a3f4db0def1e2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - Qwen2.5-Coder-32B-Instruct
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# Optimal bipartite Hungarian matching

## Summary

Reconstructs the paper's extraction-scoring alignment step: iterative Hungarian 1:1 assignment (scipy.optimize.linear_sum_assignment) over a SentenceLM cosine-similarity matrix to pair each predicted cause-effect pair with at most one gold pair before band-based probabilistic scoring.

## When to use

Use when scoring model-extracted cause-effect spans against gold pairs (tasks 2a-2h) and each predicted pair may match multiple gold pairs, so an optimal 1:1 assignment is needed instead of greedy nearest-neighbor matching.

## When NOT to use

- Task is causal detection (Experiment 1), which uses label classification and exact-match-style scoring rather than span-level pair matching
- Only one predicted pair exists per text instance, so a greedy similarity comparison suffices and bipartite assignment adds no value
- You need exact-match scoring of extraction spans — the analysis/ directory already provides separate exact-match files for that purpose

## Inputs

- Predicted cause-effect pairs from evaluated model outputs in results/experiment2/ (CSV, grouped by model and prompting strategy)
- Gold cause-effect pairs from the consolidated causal-extraction dataset (8,925 rows) in experiments/experiment2/
- Task slices 2a-2h from the causal-extraction data

## Outputs

- calibrated_score_map.csv with exactly 10 banded rows mapping cosine-similarity bands to correctness probabilities (highest band >= 0.9 -> 0.99)
- Matched-pair similarity scores written to analysis/ similarity-scoring outputs

## How to apply

Load the predicted cause-effect pairs from evaluated model outputs in results/experiment2/ and the gold pairs from the consolidated causal-extraction dataset (8,925 rows) in experiments/experiment2/ with pandas. Encode the cause-effect spans with a SentenceLM sentence-transformer model and compute the full cosine-similarity matrix between predicted and gold pairs. Run iterative 1:1 optimal assignment with scipy.optimize.linear_sum_assignment on the cosine matrix so each predicted pair aligns with at most one gold pair. Bin the matched cosine scores into 10 equal-width bands spanning 0 to 1, then calibrate a probabilistic score map (Table 6) from per-band means estimated on 4,000 human-labeled samples (400 per band, seed=4000), anchored so the band >= 0.9 maps to 0.99.

## Related tools

- **Python** (Implements the similarity matrix, Hungarian assignment via scipy.optimize.linear_sum_assignment, and 10-band calibration of the score map)
- **Qwen2.5-Coder-32B-Instruct** (Best extraction model (47.12% C_extract) whose predicted cause-effect pairs serve as one side of the matching bipartition)

## Examples

```
from scipy.optimize import linear_sum_assignment; row_ind, col_ind = linear_sum_assignment(-cosine_matrix)
```

## Evaluation signals

- calibrated_score_map.csv exists with exactly 10 banded rows
- The highest band (>= 0.9) carries the calibrated probability 0.99, reproducing Table 6
- Matched assignment is 1:1 — no predicted pair is aligned to more than one gold pair after linear_sum_assignment
- Cosine bands span 0 to 1 in 10 equal-width intervals with per-band means calibrated on 400 samples per band (4,000 total, seed=4000)
- Matched-pair scores appear in analysis/ similarity-scoring outputs

## Limitations

- Band probabilities are empirical estimates from 4,000 human-labeled samples (400 per band), so mid-band scores carry calibration uncertainty
- The score map is calibrated for this benchmark's SentenceLM embeddings and 0-1 cosine binning; transfer to other sentence encoders is unvalidated
- 1:1 matching caps each predicted pair at one gold pair, which may penalize models that split or merge causal spans

## Evidence

- [methods] Run iterative 1:1 optimal assignment with the Hungarian algorithm (scipy.optimize.linear_sum_assignment) on the cosine matrix to align each predicted pair with at most one gold pair.: [evidence span withheld — anchor status: not_found]
- [methods] Calibrate a probabilistic score map by assigning each band an empirical correctness probability, anchored so that the band >= 0.9 maps to 0.99, reproducing Table 6.: [evidence span withheld — anchor status: not_found]
- [readme] experiments/experiment2/ contains the causal-extraction data, task slices 2a-2h, prompt definitions, inference scripts, and shell launch files. The consolidated dataset file contains 8,925 rows.: "experiments/experiment2/ contains the causal-extraction data, task slices 2a-2h, prompt definitions, inference scripts, and shell launch files. The consolidated dataset file contains 8,925 rows."
- [readme] The scripts use seed `4000`.: "The scripts use seed `4000`."
