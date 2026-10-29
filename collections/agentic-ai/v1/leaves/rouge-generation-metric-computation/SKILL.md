---
name: rouge-generation-metric-computation
description: Use when evaluating sequence-to-sequence generation of commonsense KB
  tails (or similar multi-reference text generation) where each head has 3+ gold continuations
  and models fine-tuned with different templates/objectives must be compared.
when_to_use_negative:
- Evaluating classification-style tasks where output is a single discrete choice (e.g.
  CommonsenseQA / Open BookQA answer accuracy) rather than free-form generated tails
  scored against multiple gold references.
- Comparing non-generative retrieval or embedding-based KB completion methods whose
  outputs are not tail strings consumable by ROUGE.
- Settings without multiple gold tails per head (single-reference scoring), where
  the highest-score-against-gold-tails attribution scheme does not apply.
tools:
- name: Match Task to Objective (MTO) framework
  role: Supplies the template strategies (Mapping, Prompting, Masked Mapping, Masked
    Prompting) whose generated outputs the ROUGE protocol scores and compares
  repo: https://github.com/puraminy/MTO/
- name: T5-v1
  role: Encoder-decoder checkpoint pre-trained on C4 with denoising objective whose
    generated tails are ROUGE-scored
  repo: https://github.com/puraminy/MTO/
- name: T5-lm
  role: T5-v1 derivation further trained on the prefix LM objective; generated tails
    are scored to test objective alignment effects
  repo: https://github.com/puraminy/MTO/
- name: T5-base
  role: Standard T5-base checkpoint fine-tuned (batch size 8) and evaluated via ROUGE
    on 100 heads per relation
  repo: https://github.com/puraminy/MTO/
- name: T5-large
  role: Higher-capacity counterpart (batch size 4) whose generated tails are ROUGE-scored
    for model-capacity comparison
  repo: https://github.com/puraminy/MTO/
- name: Ada Factor optimizer
  role: Optimizer used for fine-tuning (learning rate 0.0001) before ROUGE evaluation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:d2b843419574b38d3d8f9b3bd08651c8140d74d6f81aa36a3534f288f18f5eb6
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
yes_rate: 0.8461538461538461
n_claims: 13
---

# rouge-generation-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes ROUGE scores for free-form generated tail texts in commonsense KB completion (ATOMIC2020 Mask-Filling / Map-Phrasal relations) using MTO-framework T5 checkpoints, attributing each test instance the highest ROUGE against its gold tails. This protocol makes few-shot gains of MTO-aligned templates over the conventional Mapping baseline quantitatively comparable.

## When to use

Use when evaluating sequence-to-sequence generation of commonsense KB tails (or similar multi-reference text generation) where each head has 3+ gold continuations and models fine-tuned with different templates/objectives must be compared.

## When NOT to use

- Evaluating classification-style tasks where output is a single discrete choice (e.g. CommonsenseQA / Open BookQA answer accuracy) rather than free-form generated tails scored against multiple gold references.
- Comparing non-generative retrieval or embedding-based KB completion methods whose outputs are not tail strings consumable by ROUGE.
- Settings without multiple gold tails per head (single-reference scoring), where the highest-score-against-gold-tails attribution scheme does not apply.

## Inputs

- Generated tail strings from T5/T5-v1/T5-lm/T5-base/T5-large checkpoints per template strategy
- ATOMIC2020 head-tail gold pairs (100 unique heads per relation, each with >= 3 gold tails, ~300 instances)
- Few-shot training splits of n = 30 samples per relation/task
- Strategy labels (Mapping baseline vs. Prompting / Masked Mapping / Masked Prompting and QA template variants)

## Outputs

- Per-instance highest-ROUGE scores averaged over instances for each relation and strategy
- Relative-gain table of each MTO-aligned strategy over the Mapping baseline (e.g. mto_fewshot_rouge_results.csv)
- Mean ROUGE over three few-shot repeats (FS n = 30) per model/template combination

## How to apply

For each of the selected ATOMIC2020 relations (Mask-Filling and Map-Phrasal), generate tails for 100 unique heads per relation, where each head has at least 3 gold tails (~300 instances total). Evaluate each generated tail with ROUGE against all gold tails of the same head and attribute the instance the highest score obtained across its gold tails, then average over instances. Apply this after fine-tuning T5-v1, T5-lm, and T5-base/T5-large checkpoints with the AdaFactor optimizer (learning rate 0.0001, mini-batch size 8 for T5-base / 4 for T5-large, 3 epochs) under each template strategy (Mapping, Prompting, Masked Mapping, Masked Prompting). In few-shot settings (n = 30 samples per relation/task), repeat the experiment three times and report the mean score. Finally compute the relative gain of each MTO-aligned strategy over the Mapping baseline (direct fine-tuning with no adaptation, marked '-') and persist per-strategy scores, e.g. to mto_fewshot_rouge_results.csv.

## Related tools

- **Match Task to Objective (MTO) framework** (Supplies the template strategies (Mapping, Prompting, Masked Mapping, Masked Prompting) whose generated outputs the ROUGE protocol scores and compares) — https://github.com/puraminy/MTO/
- **T5-v1** (Encoder-decoder checkpoint pre-trained on C4 with denoising objective whose generated tails are ROUGE-scored) — https://github.com/puraminy/MTO/
- **T5-lm** (T5-v1 derivation further trained on the prefix LM objective; generated tails are scored to test objective alignment effects) — https://github.com/puraminy/MTO/
- **T5-base** (Standard T5-base checkpoint fine-tuned (batch size 8) and evaluated via ROUGE on 100 heads per relation) — https://github.com/puraminy/MTO/
- **T5-large** (Higher-capacity counterpart (batch size 4) whose generated tails are ROUGE-scored for model-capacity comparison) — https://github.com/puraminy/MTO/
- **Ada Factor optimizer** (Optimizer used for fine-tuning (learning rate 0.0001) before ROUGE evaluation)

## Evaluation signals

- Each test instance must receive the highest ROUGE score across its >= 3 gold tails, and scores must be averaged over ~300 instances per relation (100 unique heads).
- Few-shot (FS n = 30) results must be the mean of three repeated runs; check that three repeats were executed and the mean reported.
- Validation check: the best MTO-aligned objective-template strategy must achieve a relative performance gain of over 120% compared to the conventional Mapping baseline (no adaptation, marked '-') in the few-shot setting.
- All per-strategy ROUGE scores must be written out (e.g. to mto_fewshot_rouge_results.csv) with one row per strategy/model combination.
- Fine-tuning parameters that feed the evaluation must match the protocol: AdaFactor, learning rate 0.0001, mini-batch size 8 for T5-base / 4 for T5-large, 3 epochs.

## Limitations

- Results are specific to the encoder-decoder architecture (T5 family); alternative architectures were not explored, so ROUGE-based conclusions may not transfer.
- Random seeds / variance for few-shot experiments are not reported in the results section (only the mean of three repeats), so score differences lack uncertainty estimates.
- Code availability was promised for future public release and the repository README states the code is 'under refinement and improvement', so the scoring pipeline may need to be re-implemented from the paper's description.

## Evidence

- [methods] ROUGE protocol with highest score against gold tails: [evidence span withheld — anchor status: not_found]
