---
name: atomic2020-knowledge-completion-data-formatting
description: Use when preparing few-shot KB-completion training data from ATOMIC2020
  (head, relation, tail) tuples for T5 fine-tuning with Mapping, Prompting, Masked
  Mapping, or Masked Prompting templates.
when_to_use_negative:
- The downstream model or task is outside the encoder-decoder generation/QA scope
  — the article states results are specific to the encoder-decoder architecture and
  alternative architectures were not explored
- You need full-dataset supervised KB completion rather than few-shot (n=30, 3 epochs)
  template-formatted data
- The source data lacks (head, relation, tail) tuple structure or natural-language
  relation templates, so neither mapping nor prompting templates can be instantiated
edam_operation: http://edamontology.org/operation_0306
tools:
- name: T5
  role: Encoder-decoder backbone whose checkpoints (T5-v1, T5-lm, T5-base, T5-large)
    are fine-tuned on the template-formatted few-shot sets
  repo: https://github.com/puraminy/MTO/
- name: Ada Factor optimizer
  role: Optimizer used for fine-tuning on the formatted few-shot data, with a consistent
    learning rate of 0.0001
  repo: https://github.com/puraminy/MTO/
- name: Sentence Splitter
  role: Model trained on knowledge-graph tuples and applied to OMCS sentences to produce
    adaptation data for the Mixed objective
  repo: https://github.com/puraminy/MTO/
- name: Match Task to Objective (MTO) framework
  role: Framework providing the objective/template alignment strategy that governs
    which template each model is formatted and fine-tuned with
  repo: https://github.com/puraminy/MTO/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:53617d53bc1491052af8e7318bcc5cbc6ae5be98515b370faf6d320d04560ea8
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
  - Sentence Splitter
  - T5
  - T5-base
  - T5-large
  - T5-lm
  - T5-v1
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# atomic2020-knowledge-completion-data-formatting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Formats ATOMIC2020/ATOMIC (head, relation, tail) tuples into few-shot (n=30) fine-tuning sets using the MTO paper's Mapping, Prompting, Masked Mapping, and Masked Prompting templates, aligned with each T5 variant's pre-training objective for commonsense KB completion.

## When to use

Use when preparing few-shot KB-completion training data from ATOMIC2020 (head, relation, tail) tuples for T5 fine-tuning with Mapping, Prompting, Masked Mapping, or Masked Prompting templates.

## When NOT to use

- The downstream model or task is outside the encoder-decoder generation/QA scope — the article states results are specific to the encoder-decoder architecture and alternative architectures were not explored
- You need full-dataset supervised KB completion rather than few-shot (n=30, 3 epochs) template-formatted data
- The source data lacks (head, relation, tail) tuple structure or natural-language relation templates, so neither mapping nor prompting templates can be instantiated

## Inputs

- ATOMIC2020 (head, relation, tail) tuples for selected Mask-Filling and Map-Phrasal relations
- Table A1 relation templates and Table 3 fine-tuning template definitions (Mapping, Prompting, Masked Mapping, Masked Prompting)
- Top 8,000 scored OMCS sentences (for Mixed-objective adaptation data)
- T5 checkpoints: T5-v1, T5-lm, T5-base, T5-large

## Outputs

- Few-shot training sets of n=30 samples per relation, rendered per fine-tuning template
- Adaptation corpus of split/labelled natural sentences (denoising- vs LM-suitable) for the Mixed objective
- Fine-tuned T5 checkpoints per objective/template combination
- Results table mirroring Tables B3/B4 with per-combination ROUGE and BERTScore

## How to apply

Load ATOMIC2020 (head, relation, tail) tuples for the selected Mask-Filling and Map-Phrasal relations and build few-shot training sets of exactly n=30 samples per relation using the Table 3 fine-tuning templates (Mapping, Prompting, Masked Mapping, Masked Prompting) instantiated from the Table A1 relation templates; masked variants replace the target element with a unique mask token ("X"). Choose the template so it aligns with the model's pre-training/adaptation objective (e.g., denoising-trained T5-v1 with masked-token templates; T5-lm with LM-style prompts). If the Mixed adaptation objective is used, train the Sentence Splitter on knowledge-graph tuples, apply it to the top 8,000 scored OMCS sentences, and label each produced sample as denoising- or LM-suitable. Fine-tune each T5 variant (T5-v1, T5-lm, T5-base, T5-large) with the AdaFactor optimizer at learning rate 0.0001 and batch size 8 for T5-base / 4 for T5-large for 3 epochs.

## Related tools

- **T5** (Encoder-decoder backbone whose checkpoints (T5-v1, T5-lm, T5-base, T5-large) are fine-tuned on the template-formatted few-shot sets) — https://github.com/puraminy/MTO/
- **Ada Factor optimizer** (Optimizer used for fine-tuning on the formatted few-shot data, with a consistent learning rate of 0.0001) — https://github.com/puraminy/MTO/
- **Sentence Splitter** (Model trained on knowledge-graph tuples and applied to OMCS sentences to produce adaptation data for the Mixed objective) — https://github.com/puraminy/MTO/
- **Match Task to Objective (MTO) framework** (Framework providing the objective/template alignment strategy that governs which template each model is formatted and fine-tuned with) — https://github.com/puraminy/MTO/

## Examples

```
few_shot = {rel: [fmt_template(t, 'Masked Mapping') for t in atomic2020_tuples if t.relation == rel][:30] for rel in mask_filling_relations}
```

## Evaluation signals

- Each relation's formatted few-shot set contains exactly n=30 samples over 3 fine-tuning epochs, as in Figure 4
- Masked Mapping / Masked Prompting instances contain the unique mask token ("X") as placeholder for the target element
- Evaluation covers 100 unique heads each with at least three target tails (~300 instances per relation), with ROUGE/BERTScore assigned per instance as the max over its head's tails
- Reproduced mean ROUGE/BERTScore values for each model/objective/template combination match Tables B3 and B4
- The T5-large + Mixed + Masked Prompting configuration emerges as best for Mask-Filling in the aggregated results

## Limitations

- Repository code is under refinement and improvement; the discussion promises future public release, so the code is not yet available and practitioners may need to contact the authors
- Random seeds and variance for the few-shot experiments are not reported in the results section (only the mean of three repeats)
- Findings are specific to the encoder-decoder architecture; scaling with model parameters and adaptation-data volume, and impact on classification tasks, were not investigated

## Evidence

- [methods] masked-token templates as Table 3 variants: "this template utilizes a unique mask token (referred to as ”X”) as a placeholder for the target element"
- [results] few-shot sample size on ATOMIC20 20: "using n = 30 samples on selected tasks from the ATOMIC20 20 dataset"
- [methods] AdaFactor optimizer and learning rate: "we employ the Ada Factor optimizer [34] with a consistent learning rate of 0.0001 throughout the fine-tuning process"
- [methods] Sentence Splitter training on KG tuples: "we train a model known as the Sentence Splitter using tuples extracted from a knowledge graph"
- [abstract] template-objective alignment rationale: "In the fine-tuning stage, we design novel templates that align with the objectives of the pre-training and adaptation stages."
- [readme] code availability caveat: "The code is under refinement and improvement"
