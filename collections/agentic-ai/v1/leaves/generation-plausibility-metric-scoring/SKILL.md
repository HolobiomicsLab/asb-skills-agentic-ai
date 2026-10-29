---
name: generation-plausibility-metric-scoring
description: Use when evaluating generated natural-language completions (e.g., T5
  outputs on ATOMIC20_20 relations) with automatic metrics and you need to justify
  reporting a single metric by demonstrating inter-metric correlation.
when_to_use_negative:
- The task is classification or multiple-choice QA scoring (e.g., CommonsenseQA /
  Open BookQA accuracy), not open-ended generation plausibility scoring
- You need a metric sensitive to differences the three metrics agree on — if BLEURT,
  ROUGE, and BERTScore are expected to diverge, correlation-based reduction to a single
  metric is inappropriate
edam_topics:
- http://edamontology.org/topic_0003
- http://edamontology.org/topic_0203
tools:
- name: T5
  role: Encoder-decoder model whose generated completions on ATOMIC20_20 are scored
    and correlated across metrics
  repo: https://github.com/puraminy/MTO/
- name: T5-base
  role: Widely adopted downstream variant whose per-instance metric scores feed the
    correlation analysis
  repo: https://github.com/puraminy/MTO/
- name: T5-large
  role: Higher-capacity counterpart variant included in the metric-scored MTO experiments
  repo: https://github.com/puraminy/MTO/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:50b0d06751c12d113bd83c0ff6eb4a5026a6d63a03a71dd0fcd109d8c1947669
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: cc-by-nc-sa
    url: https://github.com/puraminy/MTO
  repo_url: https://github.com/puraminy/MTO
  tools:
  - T5
  - T5-base
  - T5-large
  - T5-lm
  - T5-v1
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0003
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# generation-plausibility-metric-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores T5-generated KB-completion outputs with BLEURT, ROUGE, and BERTScore, applies the paper's max-per-instance scoring convention over multiple target tails, and verifies metric consistency via pairwise Pearson/Spearman correlations so one representative metric (ROUGE) can be reported.

## When to use

Use when evaluating generated natural-language completions (e.g., T5 outputs on ATOMIC20_20 relations) with automatic metrics and you need to justify reporting a single metric by demonstrating inter-metric correlation.

## When NOT to use

- The task is classification or multiple-choice QA scoring (e.g., CommonsenseQA / Open BookQA accuracy), not open-ended generation plausibility scoring
- You need a metric sensitive to differences the three metrics agree on — if BLEURT, ROUGE, and BERTScore are expected to diverge, correlation-based reduction to a single metric is inappropriate

## Inputs

- Per-instance prediction scores (BLEURT, ROUGE, BERTScore) from T5/T5-v1/T5-lm/T5-base/T5-large MTO experiments on ATOMIC20_20 relations
- Evaluation instances structured as head with >=3 target tails (~300 instances per relation, 100 unique heads)

## Outputs

- figure_b1_metric_correlations.png (correlation matrix/heatmap of BLEURT, ROUGE, BERTScore)
- Table of pairwise Pearson/Spearman correlation coefficients across Map-Phrasal and remaining task categories

## How to apply

Load the per-instance prediction scores (BLEURT, ROUGE, BERTScore) underlying appendix Tables B3 and B4 from the T5-based MTO experiments on the ATOMIC20_20 relations. Align scores per evaluation instance using the paper's scoring convention: for each distinct head, compare generated outputs against the corresponding tails (100 unique heads, minimum three target outputs, ~300 instances per relation) and keep the highest per-instance score. Then compute pairwise Pearson and Spearman correlations among BLEURT, ROUGE, and BERTScore with scipy.stats, separately across the Map-Phrasal and remaining task categories. The rationale is the paper's finding that the three metrics are highly correlated with similar trends, so ROUGE alone is a valid representative; render the correlation matrix/heatmap as figure_b1_metric_correlations.png and tabulate the coefficients.

## Related tools

- **T5** (Encoder-decoder model whose generated completions on ATOMIC20_20 are scored and correlated across metrics) — https://github.com/puraminy/MTO/
- **T5-base** (Widely adopted downstream variant whose per-instance metric scores feed the correlation analysis) — https://github.com/puraminy/MTO/
- **T5-large** (Higher-capacity counterpart variant included in the metric-scored MTO experiments) — https://github.com/puraminy/MTO/

## Examples

```
from scipy.stats import pearsonr, spearmanr; rho, p = pearsonr(bleurt_scores, rouge_scores); rs, ps = spearmanr(rouge_scores, bertscore_scores)
```

## Evaluation signals

- Reproduced pairwise correlation coefficients match the values reported in Section B.1 and Figure B1 (output_matches_reference)
- Each per-instance score is the maximum over at least three target tails for the same head (min three target outputs, ~300 instances per relation)
- All three metrics show similar trends across Map-Phasal and remaining task categories, consistent with the paper's high-correlation claim
- Heatmap renders a valid 3x3 symmetric matrix with unit diagonal for BLEURT, ROUGE, BERTScore
- Few-shot score variants reflect the paper's convention of averaging over three repeated runs (n = 30 samples) where applicable

## Limitations

- The MTO repository code is under refinement; materials may need to be requested from the authors by email (pouramini at gmail)
- Findings are specific to the T5 encoder-decoder architecture and KB completion / QA tasks; classification and other task types were not investigated
- Few-shot experiments (n = 30) report only the mean of three repeats without variance/seeds in the reported section

## Evidence

- [supplementary] In the paper's experiments, ROUGE, BLEURT, and BERT scores are highly correlated and exhibit similar trends, so the authors focus on ROUGE scores as the representative metric: [evidence span withheld — anchor status: not_found]
- [supplementary] for each distinct head, compare generated outputs against the corresponding tails (100 unique heads, minimum three target outputs, ~300 instances per relation) and keep the highest per-instance score: [evidence span withheld — anchor status: not_found]
