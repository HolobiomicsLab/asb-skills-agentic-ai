---
name: llm-test-time-parameter-adaptation
description: Use when a RAG pipeline underperforms on a specialized domain due to
  distribution shifts and you need parameter-level adaptation at inference time using
  only retrieved passages as self-supervised signal.
when_to_use_negative:
- When the target domain matches the pre-training distribution well enough that a
  naive-RAG, Chain-of-Thought, or In-Context Learning baseline already suffices —
  adaptation adds per-query overhead (1.75s-2.60s per query)
- When retrieved passages are shorter than the configured minimum length threshold,
  leaving too little context for meaningful adaptation
- When new labeled data and full model retraining are available/acceptable, since
  this skill is designed for inference-time adaptation without labels
edam_operation: http://edamontology.org/operation_3281
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0218
tools:
- name: Llama-3.1-8B-Instruct
  role: Base LLM whose parameters are reset and adapted via prefix-suffix prediction
    at test time
  repo: https://github.com/sunxin000/TTARAG
- name: TTARAG
  role: The test-time adaptation framework being reconstructed; code and models/tta_module.py
    live in this repository
  repo: https://github.com/sunxin000/TTARAG
- name: Pyserini
  role: Supplies the BM25 top-5 retrieved passages that serve as the self-supervised
    adaptation signal
- name: NVIDIA A100 GPU
  role: Hardware used to run the adaptation and generation (e.g. processing 2,706
    queries on one GPU)
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1109/icassp55912.2026.11464478
    title: Predict the Retrieval! Test Time Adaptation for Retrieval Augmented Generation
  source_content_hash: sha256:2ec40f3fa3cdd5376f173b5246b2f18eab6d518f71b55bcfe8d039e77a18e8af
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sunxin000/TTARAG
  tools:
  - ChatGLM-3-6b
  - Llama-2-7b-chat
  - Llama-3.1-8B-Instruct
  - NVIDIA A100 GPU
  - Pyserini
  - Qwen2.5-72B-Instruct
  - TTARAG
  edam_operation: http://edamontology.org/operation_3281
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# llm-test-time-parameter-adaptation

## Summary

Reconstructs the TTARAG test-time adaptation module that dynamically updates an LLM's parameters during inference by predicting retrieved content (prefix-suffix pairs), enabling RAG systems to adjust to specialized domains without labeled data or retraining.

## When to use

Use when a RAG pipeline underperforms on a specialized domain due to distribution shifts and you need parameter-level adaptation at inference time using only retrieved passages as self-supervised signal.

## When NOT to use

- When the target domain matches the pre-training distribution well enough that a naive-RAG, Chain-of-Thought, or In-Context Learning baseline already suffices — adaptation adds per-query overhead (1.75s-2.60s per query)
- When retrieved passages are shorter than the configured minimum length threshold, leaving too little context for meaningful adaptation
- When new labeled data and full model retraining are available/acceptable, since this skill is designed for inference-time adaptation without labels

## Inputs

- Top 5 retrieved passages per question (e.g. from a BM25 retriever implemented with Pyserini over PubMed abstracts/biomedical articles)
- Pre-trained instruction LLM weights (e.g. models/Llama-3.1-8B-Instruct)
- Query/question text q
- TTAConfig knobs (learning_rate, max_adapt_pairs)

## Outputs

- Adapted model parameters (theta') used for final response generation
- Generated answer per test question with adapted parameters
- models/tta_module.py implementing TTAConfig-compatible adaptation

## How to apply

Filter retrieved passages with length filtering, discarding passages shorter than the configured minimum length threshold so each has sufficient context for meaningful adaptation. Split each remaining passage into prefix-suffix pairs using the two-tier strategy: primary split at punctuation boundaries (periods, commas, semicolons, colons, exclamation marks, question marks), with a fallback midpoint split ensuring each segment has at least three words. Reset the LLM parameters (e.g. Llama-3.1-8B-Instruct) to their original pre-trained state before each adaptation iteration, then compute the self-supervised loss L_adapt = -sum_i log P(suffix_i | prefix_i, q; theta), tokenizing the full text and masking prefix tokens in label preparation so the loss applies only to suffix prediction. Accumulate gradients over two steps, clip them to a maximum norm threshold, and update with AdamW at a learning rate in the 1e-6 to 1e-5 range (e.g. 1e-5), using up to max_adapt_pairs=3 prefix-suffix pairs per query; the paper reports 3-5 pairs balance effectiveness and efficiency. Generate the final response with the adapted parameters (theta'), and validate end-to-end on a small batch of retrieved passages, checking that the produced module matches the models/tta_module.py TTARAG interface (TTAConfig knobs: learning_rate, max_adapt_pairs).

## Related tools

- **Llama-3.1-8B-Instruct** (Base LLM whose parameters are reset and adapted via prefix-suffix prediction at test time) — https://github.com/sunxin000/TTARAG
- **TTARAG** (The test-time adaptation framework being reconstructed; code and models/tta_module.py live in this repository) — https://github.com/sunxin000/TTARAG
- **Pyserini** (Supplies the BM25 top-5 retrieved passages that serve as the self-supervised adaptation signal)
- **NVIDIA A100 GPU** (Hardware used to run the adaptation and generation (e.g. processing 2,706 queries on one GPU))

## Examples

```
python parallel_evaluation.py --do_generate --dataset pubmedqa --model_name models/Llama-3.1-8B-Instruct --do_tta --learning_rate 1e-5 --max_adapt_pairs 3
```

## Evaluation signals

- Module runs end-to-end on a small batch of retrieved passages without errors
- Parameter updates follow Equation 2 with reset-to-pretrained initialization and two-step gradient accumulation with norm clipping
- Loss is computed only on suffix tokens, verified by prefix-token masking during label preparation
- Produced tta_module.py matches the models/tta_module.py interface (TTAConfig knobs: learning_rate, max_adapt_pairs)
- Downstream: adapted model's accuracy exceeds the naive-RAG baseline, e.g. BioASQ +19.4% (75.0 vs 55.6) and PubMedQA +10.8% for Llama-3.1-8b-it

## Limitations

- Performance is not sensitive to the number of adaptation pairs and learning rate, so tuning offers limited leverage; 3-5 pairs and 1e-6 to 1e-5 learning rates are recommended
- Passages shorter than the minimum length threshold must be discarded, reducing the number of usable adaptation pairs
- Parameters must be reset to the original pre-trained state for each adaptation iteration to ensure a clean starting point
- Average processing time per query increases with pair count, from 1.75s (1-pair) to 2.60s (5-pair)

## Evidence

- [methods] Length Filtering. To ensure sufficient context for meaningful adaptation, passages shorter than a configured minimum length threshold are filtered out.: [evidence span withheld — anchor status: not_found]
- [methods] learning rates between 1e-6 to 1e-5 provide optimal performance, with 3-5 adaptation pairs yielding the best balance between effectiveness and efficiency: [evidence span withheld — anchor status: not_found]
