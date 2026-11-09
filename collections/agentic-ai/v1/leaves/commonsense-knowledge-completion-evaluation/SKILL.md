---
name: commonsense-knowledge-completion-evaluation
description: Use when scoring generated tails of Masked-Filling/Map-Phrasal ATOMIC2020
  tasks from T5 models, or when checking whether ROUGE/BLEURT/BERTScore rankings agree
  (Figure B1).
when_to_use_negative:
- Evaluating question-answering tasks (CommonsenseQA, Open BookQA), which use accuracy/choice-based
  Answer Prompting and Choice Prompting rather than multi-reference tail generation
  scoring
- Tasks with exactly one gold reference per instance — the max-over-gold-tails convention
  assumes at least three tails per head
- Comparing decoder-only or encoder-only architectures — the paper's results are specific
  to the encoder-decoder (T5) setting
tools:
- name: T5
  role: Encoder-decoder checkpoints (T5-v1, T5-lm, T5-base, T5-large) whose generated
    tails are scored under this evaluation protocol
  repo: https://github.com/puraminy/MTO/
- name: Match Task to Objective (MTO) framework
  role: Provides the fine-tuning templates whose few-shot outputs are evaluated via
    the max-over-gold-tails ROUGE convention
  repo: https://github.com/puraminy/MTO/
- name: Ada Factor optimizer
  role: Optimizer (learning rate 0.0001, mini-batch 8 for T5-base / 4 for T5-large,
    3 epochs) used to produce the checkpoints being evaluated
provenance:
  source_task_ids:
  - task_008
  - task_003
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:fdd037add1b955b88430c0b1094a63844963dd2d3a8e75370dbf9a7e195c5554
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
  - Match Task to Objective (MTO) framework
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
yes_rate: 1.0
n_claims: 16
---

# commonsense-knowledge-completion-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluation protocol for commonsense KB completion generation tasks: score generated tails (from MTO-aligned fine-tuned T5 checkpoints on ATOMIC2020 relations) with ROUGE, BLEURT, and BERTScore, attributing each head the maximum score against its multiple gold tails.

## When to use

Use when scoring generated tails of Masked-Filling/Map-Phrasal ATOMIC2020 tasks from T5 models, or when checking whether ROUGE/BLEURT/BERTScore rankings agree (Figure B1).

## When NOT to use

- Evaluating question-answering tasks (CommonsenseQA, Open BookQA), which use accuracy/choice-based Answer Prompting and Choice Prompting rather than multi-reference tail generation scoring
- Tasks with exactly one gold reference per instance — the max-over-gold-tails convention assumes at least three tails per head
- Comparing decoder-only or encoder-only architectures — the paper's results are specific to the encoder-decoder (T5) setting

## Inputs

- Per-instance generated tail predictions from T5/T5-v1/T5-lm/T5-base/T5-large checkpoints fine-tuned with Mapping, Prompting, Masked Mapping, or Masked Prompting templates
- Gold ATOMIC2020 head-tail relation sets (100 unique heads per relation, minimum three target tails per head, ~300 instances per relation)
- Few-shot split specification (n = 30 samples per relation/task) and per-instance ROUGE/BLEURT/BERTScore values underlying Tables B3 and B4

## Outputs

- Per-relation averaged ROUGE scores (mto_fewshot_rouge_results.csv) with relative gains of each aligned strategy over the Mapping baseline
- Pairwise Pearson/Spearman correlation matrix among BLEURT, ROUGE, and BERTScore (figure_b1_metric_correlations.png)
- Validation report confirming reproduced correlations match Section B.1 / Figure B1 values

## How to apply

Evaluate seq2seq outputs against multiple references using the paper's convention: for each relation, take 100 unique heads each with at least three gold tails (~300 instances per relation) and attribute each instance the highest score against its gold tails, then average over instances. Load the per-instance ROUGE, BLEURT, and BERTScore predictions underlying Tables B3/B4 from the T5-based MTO experiments on the 20 ATOMIC20_20 relations. Because the paper shows the three metrics are highly correlated with similar trends (Figure B1, Section B.1), use ROUGE as the representative metric for headline comparisons such as the >120% few-shot gain of MTO-aligned strategies over the Mapping baseline. Optionally verify metric agreement by computing pairwise Pearson/Spearman correlations (scipy.stats) across Map-Phrasal and remaining task categories.

## Related tools

- **T5** (Encoder-decoder checkpoints (T5-v1, T5-lm, T5-base, T5-large) whose generated tails are scored under this evaluation protocol) — https://github.com/puraminy/MTO/
- **Match Task to Objective (MTO) framework** (Provides the fine-tuning templates whose few-shot outputs are evaluated via the max-over-gold-tails ROUGE convention) — https://github.com/puraminy/MTO/
- **Ada Factor optimizer** (Optimizer (learning rate 0.0001, mini-batch 8 for T5-base / 4 for T5-large, 3 epochs) used to produce the checkpoints being evaluated)

## Examples

```
from scipy.stats import pearsonr, spearmanr; r_p, _ = pearsonr(rouge_scores, bleurt_scores); r_s, _ = spearmanr(rouge_scores, bert_scores)  # pairwise metric correlations per Section B.1
```

## Evaluation signals

- Each relation's evaluation set contains exactly 100 unique heads, each with at least 3 gold tails (~300 instances per relation)
- Per-instance scores use the maximum against the head's gold tails before averaging over instances
- Reproduced Pearson/Spearman coefficients among BLEURT, ROUGE, and BERTScore match the values in Section B.1 / Figure B1 (output_matches_reference)
- Best MTO-aligned strategy yields a relative gain over 120% versus the Mapping baseline (direct fine-tuning with no adaptation, marked '-') in the n=30 few-shot setting
- Few-shot results are the mean of three experiment repeats

## Limitations

- The three metrics are highly correlated, so metric choice is mostly redundant — but the correlation was only verified on the T5-based MTO experiments over the 20 ATOMIC20_20 relations
- Experiments were repeated three times with means reported; random seeds/variance for the few-shot runs are not reported in the paper
- Results are specific to the encoder-decoder architecture; scaling with model parameters and adaptation-data amounts was not investigated
- MTO code was under refinement at README-writing time; availability was promised via direct contact (pouramini at gmail) rather than a fully published release

## Evidence

- [results] In the paper's experiments, ROUGE, BLEURT, and BERT scores are highly correlated and exhibit similar trends, so the authors focus on ROUGE scores as the representative metric (see Figure B1 in the Appendix).: [evidence span withheld — anchor status: not_found]
- [results] using n = 30 samples on selected tasks from the ATOMIC20 20 dataset: "using n = 30 samples on selected tasks from the ATOMIC20 20 dataset"
