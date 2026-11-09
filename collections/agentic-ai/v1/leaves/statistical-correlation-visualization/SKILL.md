---
name: statistical-correlation-visualization
description: Use when multiple automated metrics (ROUGE, BLEURT, BERTScore) score
  the same generation outputs and you must justify collapsing to a single representative
  metric via a correlation matrix/heatmap.
when_to_use_negative:
- Scores exist only as aggregate per-run means rather than per-instance values — the
  analysis requires per-instance alignment with the max-score-per-head convention
- Only a single metric was computed for the outputs, so there is no second metric
  to correlate against
- The question is which fine-tuning method performs best (MTO strategy comparison),
  not whether the evaluation metrics agree
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: T5
  role: Backbone of the MTO experiments; its generated ATOMIC20 20 tails are the outputs
    scored per-instance by BLEURT, ROUGE, and BERTScore — the scores this skill correlates
  repo: https://github.com/puraminy/MTO/
- name: T5-v1
  role: Denoising-objective pre-trained T5 variant whose predictions are among the
    scored instances feeding the correlation matrix
  repo: https://github.com/puraminy/MTO/
- name: T5-lm
  role: Prefix-LM-adapted T5 variant (100k extra LM steps) whose predictions are among
    the scored instances feeding the correlation matrix
  repo: https://github.com/puraminy/MTO/
- name: T5-base
  role: Base-capacity checkpoint whose per-instance metric scores feed the correlation
    matrix across task categories
  repo: https://github.com/puraminy/MTO/
- name: T5-large
  role: Large-capacity checkpoint whose per-instance metric scores feed the correlation
    matrix across task categories
  repo: https://github.com/puraminy/MTO/
- name: scipy.stats
  role: Computes the pairwise Pearson and Spearman correlation coefficients among
    BLEURT, ROUGE, and BERTScore
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:ec32b245f18df703869cd34359b2cb14ffc74627172007d028c4d2abbd68697f
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
  - scipy.stats
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# statistical-correlation-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes pairwise Pearson/Spearman correlations across per-instance BLEURT, ROUGE, and BERTScore scores from the T5-based MTO experiments on ATOMIC20 20 and renders the correlation matrix/heatmap (Figure B1), justifying ROUGE as the single representative metric for downstream reporting.

## When to use

Use when multiple automated metrics (ROUGE, BLEURT, BERTScore) score the same generation outputs and you must justify collapsing to a single representative metric via a correlation matrix/heatmap.

## When NOT to use

- Scores exist only as aggregate per-run means rather than per-instance values — the analysis requires per-instance alignment with the max-score-per-head convention
- Only a single metric was computed for the outputs, so there is no second metric to correlate against
- The question is which fine-tuning method performs best (MTO strategy comparison), not whether the evaluation metrics agree

## Inputs

- Per-instance BLEURT, ROUGE, and BERTScore prediction scores underlying appendix Tables B3 and B4 (T5-based MTO runs on ATOMIC20 20)
- ATOMIC20 20 evaluation instances: distinct heads with their target tails (100 unique heads, minimum three target outputs, ~300 instances per relation)
- Generated tail outputs from the fine-tuned T5, T5-v1, T5-lm, T5-base, and T5-large checkpoints

## Outputs

- Pairwise Pearson/Spearman correlation coefficient table among BLEURT, ROUGE, and BERTScore, per task category
- Correlation matrix heatmap rendered as figure_b1_metric_correlations.png
- Tabulated coefficients replicating the values reported in Section B.1

## How to apply

Load the per-instance BLEURT, ROUGE, and BERTScore scores underlying appendix Tables B3 and B4, produced by the T5-based MTO experiments on ATOMIC20 20 relations. Align scores per evaluation instance using the paper's convention: for each distinct head, compare generated outputs against the corresponding tails (100 unique heads, minimum three target outputs, ~300 instances per relation) and keep the highest per-instance score. Compute pairwise Pearson and Spearman correlations among BLEURT, ROUGE, and BERTScore with scipy.stats, separately for the Map-Phrasal category and the remaining task categories. Render the correlation matrix/heatmap as figure_b1_metric_correlations.png and tabulate the coefficients. High pairwise correlation means the metrics rank fine-tuning methods consistently, so ROUGE alone can be reported; validate the reproduced coefficients against Section B.1 and Figure B1 (output_matches_reference).

## Related tools

- **T5** (Backbone of the MTO experiments; its generated ATOMIC20 20 tails are the outputs scored per-instance by BLEURT, ROUGE, and BERTScore — the scores this skill correlates) — https://github.com/puraminy/MTO/
- **T5-v1** (Denoising-objective pre-trained T5 variant whose predictions are among the scored instances feeding the correlation matrix) — https://github.com/puraminy/MTO/
- **T5-lm** (Prefix-LM-adapted T5 variant (100k extra LM steps) whose predictions are among the scored instances feeding the correlation matrix) — https://github.com/puraminy/MTO/
- **T5-base** (Base-capacity checkpoint whose per-instance metric scores feed the correlation matrix across task categories) — https://github.com/puraminy/MTO/
- **T5-large** (Large-capacity checkpoint whose per-instance metric scores feed the correlation matrix across task categories) — https://github.com/puraminy/MTO/
- **scipy.stats** (Computes the pairwise Pearson and Spearman correlation coefficients among BLEURT, ROUGE, and BERTScore)

## Examples

```
from scipy.stats import pearsonr, spearmanr; pearsonr(rouge_scores, bleurt_scores); spearmanr(rouge_scores, bert_scores)  # per-instance scores from Tables B3/B4
```

## Evaluation signals

- Reproduced pairwise Pearson/Spearman coefficients match the values reported in Section B.1 and Figure B1 (output_matches_reference)
- Per-instance table shows the expected structure: ~300 instances per relation, 100 unique heads, and the maximum score kept across each head's target tails
- Correlation matrix is symmetric with a unit diagonal; heatmap renders as figure_b1_metric_correlations.png
- Pearson and Spearman coefficients agree in sign and are near +1 across metric pairs, supporting the 'highly correlated' reduction to ROUGE
- The correlation pattern holds for both the Map-Phrasal category and the remaining task categories

## Limitations

- The MTO repository code is 'under refinement and improvement' and was only promised for future public release; Tables B3/B4 score dumps may need to be requested from the authors (pouramini at gmail)
- Correlations are established for T5-family encoder-decoder models on ATOMIC20 20 KB completion; the paper notes results are specific to this architecture
- The max-score-per-head convention means correlations reflect best-match scoring per head; behavior under average-score aggregation is not evaluated

## Evidence

- [supplementary] ROUGE, BLEURT, and BERT scores are highly correlated and exhibit similar trends, so the authors focus on ROUGE scores as the representative metric (see Figure B1 in the Appendix).: [evidence span withheld — anchor status: not_found]
