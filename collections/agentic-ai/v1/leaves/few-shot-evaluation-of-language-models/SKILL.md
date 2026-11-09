---
name: few-shot-evaluation-of-language-models
description: Use when comparing fine-tuning strategies for encoder-decoder PLMs under
  few-shot data scarcity (e.g., 30 training samples per task) and you need relative
  performance gains over a supervised baseline.
when_to_use_negative:
- Full-dataset (non-few-shot) fine-tuning scenarios where the >120% few-shot gain
  claim does not apply and baseline gaps are much smaller
- Non-generative tasks such as classification, where the article states the impact
  of the proposed objective was not investigated
- Non encoder-decoder architectures, since the article notes results are specific
  to the encoder-decoder architecture
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3360
tools:
- name: T5
  role: Base encoder-decoder PLM fine-tuned under each task-template strategy
  repo: https://github.com/puraminy/MTO/
- name: T5-v1
  role: Checkpoint pre-trained with a denoising objective on C4, used for denoising-aligned
    (masked) templates
  repo: https://github.com/puraminy/MTO/
- name: T5-lm
  role: T5-v1 variant given 100k extra prefix-LM training steps, used for LM-aligned
    (prompting) templates
  repo: https://github.com/puraminy/MTO/
- name: T5-base
  role: Widely adopted checkpoint fine-tuned with mini-batch size 8 in the few-shot
    comparison
  repo: https://github.com/puraminy/MTO/
- name: Ada Factor optimizer
  role: Optimizer with a consistent learning rate of 0.0001 throughout fine-tuning
  repo: https://github.com/puraminy/MTO/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:4dcf8ac17c527caa66351f39216fb98ff9d498185b0e437cff220d6ab7c3fb83
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
  - T5-lm
  - T5-v1
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# few-shot-evaluation-of-language-models

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce and evaluate few-shot (n=30 per relation/task) fine-tuning of T5 encoder-decoder models on commonsense KB completion, comparing objective-aligned strategies (Masked Mapping, Masked Prompting) against conventional supervised Mapping to quantify relative ROUGE gains.

## When to use

Use when comparing fine-tuning strategies for encoder-decoder PLMs under few-shot data scarcity (e.g., 30 training samples per task) and you need relative performance gains over a supervised baseline.

## When NOT to use

- Full-dataset (non-few-shot) fine-tuning scenarios where the >120% few-shot gain claim does not apply and baseline gaps are much smaller
- Non-generative tasks such as classification, where the article states the impact of the proposed objective was not investigated
- Non encoder-decoder architectures, since the article notes results are specific to the encoder-decoder architecture

## Inputs

- ATOMIC2020 head-tail pairs for Mask-Filling and Map-Phrasal relations
- Few-shot training splits (n=30 samples per relation/task)
- Top 8,000 relevance-ranked OMCS sentences for the unsupervised adaptation stage
- Pre-trained checkpoints: T5-v1, T5-lm, T5-base, T5-large
- Evaluation set: 100 unique heads per relation, each with >=3 gold tails (~300 instances)

## Outputs

- Per-strategy ROUGE scores for each few-shot fine-tuning variant
- Relative-gain comparison of Prompting/Masked Mapping/Masked Prompting vs. the Mapping baseline
- mto_fewshot_rouge_results.csv with per-strategy scores

## How to apply

Draw few-shot training splits of n=30 samples per relation/task from ATOMIC2020 Mask-Filling and Map-Phrasal relations and format head-tail pairs with Table 3 templates (Mapping, Prompting, Masked Mapping, Masked Prompting), placing the 'X' mask token before the target fragment. Before fine-tuning, run an adaptation stage: unsupervised pretraining on the top 8,000 relevance-ranked OMCS sentences split by the Sentence Splitter, applying the denoising (masked-token) objective for Mask-Filling tasks and the LM objective for Map-Phrasal tasks. Fine-tune T5-v1, T5-lm, and T5-base/T5-large checkpoints with the AdaFactor optimizer at learning rate 0.0001, mini-batch size 8 (T5-base) or 4 (T5-large), for 3 epochs. Evaluate generated tails with ROUGE on 100 unique heads per relation (each with at least 3 gold tails, ~300 instances), attributing each instance its highest score against its gold tails and averaging over instances. Compute each aligned strategy's relative gain over the Mapping baseline (direct fine-tuning with no adaptation, marked '-' in Figure 4) and confirm the best MTO-aligned strategy exceeds 120% relative gain.

## Related tools

- **T5** (Base encoder-decoder PLM fine-tuned under each task-template strategy) — https://github.com/puraminy/MTO/
- **T5-v1** (Checkpoint pre-trained with a denoising objective on C4, used for denoising-aligned (masked) templates) — https://github.com/puraminy/MTO/
- **T5-lm** (T5-v1 variant given 100k extra prefix-LM training steps, used for LM-aligned (prompting) templates) — https://github.com/puraminy/MTO/
- **T5-base** (Widely adopted checkpoint fine-tuned with mini-batch size 8 in the few-shot comparison) — https://github.com/puraminy/MTO/
- **Ada Factor optimizer** (Optimizer with a consistent learning rate of 0.0001 throughout fine-tuning) — https://github.com/puraminy/MTO/

## Examples

```
python mto_finetune.py --task masked-prompting --fewshot 30 --model t5-base --lr 0.0001 --batch-size 8 --epochs 3 --adaptation-omcs 8000
```

## Evaluation signals

- The best MTO-aligned objective-template strategy achieves a relative ROUGE gain of over 120% versus the Mapping baseline in the few-shot setting
- Few-shot experiments repeated three times with mean scores reported; per-strategy scores must appear in mto_fewshot_rouge_results.csv
- Evaluation covers 100 unique heads per relation with >=3 gold tails each (~300 instances); per-instance ROUGE uses the maximum over gold tails
- Mini-batch size respects model size: 8 for T5-base, 4 for T5-large, with 3 training epochs at learning rate 0.0001
- The Mapping baseline row shows '-' for adaptation (no adaptation stage), distinguishing it from aligned strategies

## Limitations

- Few-shot QA experiments report only the mean of three repeats; random seeds and variance are not reported in this section
- Results are specific to the encoder-decoder architecture; alternative architectures were not explored
- Scaling with model parameters and varying amounts of adaptation-phase training data not investigated
- Code was promised for future public release; the README states the code is 'under refinement and improvement' and directs users to contact the author for project materials

## Evidence
