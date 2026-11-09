---
name: rouge-bertscore-generation-evaluation
description: Use when scoring natural-language generations (e.g., ATOMIC tail predictions
  from T5 variants) against multiple reference tails per head and comparing fine-tuning
  configurations with automatic text-generation metrics.
when_to_use_negative:
- The target task is not natural-language generation (e.g., classification or QA multiple-choice
  scoring), where BLEU/ROUGE-style overlap metrics are not the paper's evaluation
  focus
- Only a single reference exists per head with no multiple-tail scoring convention,
  so the max-over-tails instance scoring of the paper does not apply
- Human-adequacy or semantic-correctness judgments are needed; the paper's automatic
  metrics (ROUGE, BERTScore, BLEURT) only capture correlation, not correctness guarantees
tools:
- name: T5-large
  role: Checkpoint whose Mask-Filling generations (Mixed objective + Masked Prompting)
    are evaluated and expected to emerge as best under this scoring protocol
  repo: https://github.com/puraminy/MTO/
- name: T5-base
  role: Checkpoint fine-tuned at batch size 8 with AdaFactor whose per-configuration
    ROUGE/BERTScore rows populate the results tables
  repo: https://github.com/puraminy/MTO/
- name: Ada Factor optimizer
  role: Optimizer used at a consistent learning rate of 0.0001 during the fine-tuning
    runs whose outputs this evaluation scores
  repo: https://github.com/puraminy/MTO/
provenance:
  source_task_ids:
  - task_001
  - task_008
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:a20c21f98fc0fc92a5890a924b0c08f95f7bf50f59752dc70e5e6ade96891e5c
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
  - Ada Factor optimizer
  - T5
  - T5-base
  - T5-large
  - T5-lm
  - T5-v1
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# rouge-bertscore-generation-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluation protocol for tail-generation (KB completion) outputs from T5-based models: score each generated tail against gold tails with ROUGE, BERTScore, and BLEURT, keeping the highest per-instance score over a head's tails, then rely on ROUGE as the representative metric given the high correlation among the three.

## When to use

Use when scoring natural-language generations (e.g., ATOMIC tail predictions from T5 variants) against multiple reference tails per head and comparing fine-tuning configurations with automatic text-generation metrics.

## When NOT to use

- The target task is not natural-language generation (e.g., classification or QA multiple-choice scoring), where BLEU/ROUGE-style overlap metrics are not the paper's evaluation focus
- Only a single reference exists per head with no multiple-tail scoring convention, so the max-over-tails instance scoring of the paper does not apply
- Human-adequacy or semantic-correctness judgments are needed; the paper's automatic metrics (ROUGE, BERTScore, BLEURT) only capture correlation, not correctness guarantees

## Inputs

- Generated tail predictions from T5 checkpoints (T5-v1, T5-lm, T5-base, T5-large) for each objective/template configuration
- Gold ATOMIC2020 tails keyed by head (100 unique heads per relation, >=3 target tails per head, ~300 instances per relation)
- Per-instance metric scores (ROUGE, BERTScore, BLEURT) underlying Tables B3/B4

## Outputs

- Per-instance maximum-over-tails ROUGE/BERTScore/BLEURT scores averaged per relation and configuration
- Results table mirroring Tables B3 and B4 with best per-model objective/template assignments
- Correlation matrix/heatmap (e.g., figure_b1_metric_correlations.png) of pairwise Pearson/Spearman coefficients among BLEURT, ROUGE, and BERTScore

## How to apply

Generate tails for evaluation heads (the paper uses 100 unique heads per relation, each with at least three target tails, ~300 instances per relation) and score each instance against its head's tails using ROUGE, BERTScore, and BLEURT, assigning each instance the highest score over its head's tails and averaging over all instances. Align scores per evaluation instance before computing cross-metric statistics. Then compute pairwise Pearson/Spearman correlations among BLEURT, ROUGE, and BERTScore (e.g., with scipy.stats); in the paper's experiments these metrics are highly correlated with similar trends, so ROUGE is reported as the representative metric (see Figure B1). Aggregate scores into results tables mirroring Tables B3 and B4 and record the best per-model objective/template assignment (e.g., T5-large + Mixed + Masked Prompting for Mask-Filling). Keep the few-shot protocol intact: n=30 samples per relation over 3 epochs, AdaFactor optimizer at learning rate 0.0001, batch size 8 for T5-base and 4 for T5-large.

## Related tools

- **T5-large** (Checkpoint whose Mask-Filling generations (Mixed objective + Masked Prompting) are evaluated and expected to emerge as best under this scoring protocol) — https://github.com/puraminy/MTO/
- **T5-base** (Checkpoint fine-tuned at batch size 8 with AdaFactor whose per-configuration ROUGE/BERTScore rows populate the results tables) — https://github.com/puraminy/MTO/
- **Ada Factor optimizer** (Optimizer used at a consistent learning rate of 0.0001 during the fine-tuning runs whose outputs this evaluation scores) — https://github.com/puraminy/MTO/

## Examples

```
from scipy.stats import spearmanr; rho, p = spearmanr(rouge_scores, bertscore_scores)  # per-instance scores, max over >=3 tails per head
```

## Evaluation signals

- Reproduced mean ROUGE/BERTScore values for each model/objective/template combination match the values reported in appendix Tables B3 and B4
- Every evaluation instance is scored against at least three target tails for its head, and the instance receives the maximum score over those tails before averaging
- Pairwise Pearson/Spearman correlation coefficients among BLEURT, ROUGE, and BERTScore match the values reported in Section B.1 / Figure B1, showing high correlation and similar trends
- Few-shot evaluation uses n=30 samples per relation over 3 epochs, matching the Figure 4 protocol, with T5-large + Mixed + Masked Prompting emerging as best for Mask-Filling
- Evaluation covers 100 unique heads per relation with ~300 instances per relation

## Limitations

- ROUGE, BLEURT, and BERT scores are highly correlated and exhibit similar trends, so reporting all three is redundant; the authors focus on ROUGE as the representative metric
- In few-shot settings (FS n=30), experiments were repeated three times with only mean scores reported; random seeds and variance are not reported in this section
- Code availability: repository code is under refinement, and the paper promises future public release on GitHub; evaluation scripts may not yet be reproducible from the repo

## Evidence
