---
name: passage-prefix-suffix-splitting
description: 'Use when building test-time adaptation signals for RAG: you have retrieved
  passages (e.g. top-20 MiniLM-v6 reranked sentences) and need prefix-suffix pairs
  for suffix-prediction training on models like Llama-3.1-8B-Instruct.'
when_to_use_negative:
- Passages are shorter than the configured minimum length threshold — they are filtered
  out before splitting and cannot yield meaningful prefix-suffix context
- The goal is the 'wo seg' ablation baseline that deliberately adapts on whole passages
  via next-token prediction without prefix-suffix splitting
- You need supervised domain adaptation with labeled data — TTARAG's splitting exists
  specifically to create self-supervised signals without labels
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3377
tools:
- name: TTARAG
  role: Framework whose adaptation mechanism consumes the prefix-suffix pairs as self-supervised
    learning signals for test-time parameter updates
  repo: https://github.com/sunxin000/TTARAG
- name: Llama-3.1-8B-Instruct
  role: Base LLM whose parameters are adapted on the split prefix-suffix pairs during
    test-time adaptation
- name: ms-marco-MiniLM-L-6-v2
  role: Cross-encoder used to select/rerank retrieval content that is then split into
    prefix-suffix pairs
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1109/icassp55912.2026.11464478
    title: Predict the Retrieval! Test Time Adaptation for Retrieval Augmented Generation
  source_content_hash: sha256:54c59840807f286566bc043afe2eb935c17444956fe3d18663ed566265599fe6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sunxin000/TTARAG
  tools:
  - Llama-3.1-8B-Instruct
  - NVIDIA A100 GPU
  - Qwen2.5-72B-Instruct
  - TTARAG
  - ms-marco-MiniLM-L-6-v2
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3377
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# passage-prefix-suffix-splitting

## Summary

Converts retrieved passages into self-supervised prefix-suffix training pairs for TTARAG: short passages are filtered out and each remaining passage is split with a two-tier strategy (punctuation-boundary split with a midpoint fallback) so the LLM can be trained to predict suffixes from prefixes during test-time adaptation.

## When to use

Use when building test-time adaptation signals for RAG: you have retrieved passages (e.g. top-20 MiniLM-v6 reranked sentences) and need prefix-suffix pairs for suffix-prediction training on models like Llama-3.1-8B-Instruct.

## When NOT to use

- Passages are shorter than the configured minimum length threshold — they are filtered out before splitting and cannot yield meaningful prefix-suffix context
- The goal is the 'wo seg' ablation baseline that deliberately adapts on whole passages via next-token prediction without prefix-suffix splitting
- You need supervised domain adaptation with labeled data — TTARAG's splitting exists specifically to create self-supervised signals without labels

## Inputs

- Retrieved passages (e.g. top-20 sentences per query after MiniLM-v6 reranking, or top-5 BM25 passages)
- Configured minimum passage length threshold
- Query text q used as conditioning in the suffix-prediction objective
- Tokenized passage text with labels prepared for prefix masking

## Outputs

- Prefix-suffix pairs per query (up to max_adapt_pairs, e.g. 3)
- Tokenized training batch with prefix tokens masked in labels
- Adaptation loss L_adapt = -sum_i log P(suffix_i | prefix_i, q; theta) gradients for AdamW updates

## How to apply

First apply length filtering, discarding passages shorter than the configured minimum length threshold since they cannot support meaningful adaptation. Split each surviving passage with the two-tier strategy: primary split at punctuation boundaries (periods, commas, semicolons, colons, exclamation marks, question marks), falling back to a midpoint split that guarantees each segment has at least three words. Use the resulting pairs in the TTARAG loss L_adapt = -sum_i log P(suffix_i | prefix_i, q; theta), tokenizing the full text and masking prefix tokens during label preparation so loss applies only to suffix prediction. Cap the pairs used per query (max_adapt_pairs=3, with 3-5 pairs reported optimal) and reset model parameters to their pre-trained state before each adaptation iteration. The segmentation itself matters: the CRAG ablation shows removing prefix-suffix splitting (next-token prediction on whole passages) drops accuracy (e.g. 31.9 vs 30.8 for Llama-3.1-8b-it), confirming split pairs are the source of the learning signal.

## Related tools

- **TTARAG** (Framework whose adaptation mechanism consumes the prefix-suffix pairs as self-supervised learning signals for test-time parameter updates) — https://github.com/sunxin000/TTARAG
- **Llama-3.1-8B-Instruct** (Base LLM whose parameters are adapted on the split prefix-suffix pairs during test-time adaptation)
- **ms-marco-MiniLM-L-6-v2** (Cross-encoder used to select/rerank retrieval content that is then split into prefix-suffix pairs)

## Examples

```
python parallel_evaluation.py --do_generate --dataset pubmedqa --model_name models/Llama-3.1-8B-Instruct --do_tta --learning_rate 1e-5 --max_adapt_pairs 3 --cross_encoder_name models/ms-marco-MiniLM-L-6-v2
```

## Evaluation signals

- Each split segment contains at least three words when the punctuation-boundary split falls back to midpoint splitting
- No passage below the configured minimum length threshold enters the pair construction step
- Prefix tokens are masked in the label preparation so loss is computed only on suffix prediction
- Up to max_adapt_pairs=3 prefix-suffix pairs are consumed per query with learning rate in the 1e-6 to 1e-5 range
- Removing the splitting (whole-passage next-token prediction) consistently lowers CRAG accuracy vs full TTARAG across Llama-3.1-8B-Instruct, Llama-2-7b-chat, and ChatGLM-3-6b (e.g. 31.9 vs 30.8 on Llama-3.1-8b-it)

## Limitations

- Performance is reported as not sensitive to the number of adaptation pairs and learning rate, but very short passages must be discarded by length filtering and cannot contribute pairs
- Segmentation must be removed entirely to reproduce the 'wo seg' ablation; partially splitting changes the learning signal
- Model parameters must be reset to the original pre-trained state before each adaptation iteration; skipping the reset invalidates per-query adaptation

## Evidence

- [methods] Length Filtering. To ensure sufficient context for meaningful adaptation, passages shorter than a configured minimum length threshold are filtered out.: [evidence span withheld — anchor status: not_found]
