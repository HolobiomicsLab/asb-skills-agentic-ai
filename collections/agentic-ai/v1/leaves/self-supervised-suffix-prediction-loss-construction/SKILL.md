---
name: self-supervised-suffix-prediction-loss-construction
description: 'Use when adapting an RAG system at test time to a specialized domain
  without labeled data: retrieved passages must be converted into a self-supervised
  prefix-suffix prediction loss for parameter updates.'
when_to_use_negative:
- Retrieved passages are all shorter than the configured minimum length threshold,
  leaving no usable prefix-suffix pairs for adaptation
- The task requires domain adaptation with labeled data or full model retraining rather
  than a lightweight inference-time update
- You need a static evaluation pipeline with fixed parameters — TTARAG resets parameters
  per query, so it is unsuitable when per-query parameter mutation is undesirable
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: TTARAG
  role: Test-time adaptation framework whose self-supervised objective trains the
    model to predict retrieved content
  repo: https://github.com/sunxin000/TTARAG
- name: Llama-3.1-8B-Instruct
  role: Base LLM whose parameters are reset per query and updated via the prefix-suffix
    suffix-prediction loss
- name: ms-marco-MiniLM-L-6-v2
  role: Cross-encoder supplying filtered retrieval content from which prefix-suffix
    pairs are constructed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1109/icassp55912.2026.11464478
    title: Predict the Retrieval! Test Time Adaptation for Retrieval Augmented Generation
  source_content_hash: sha256:d96f42aad28d9f47b222c90aad1bdc034d46af1f5d525dacf6d00f9b744bc844
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sunxin000/TTARAG
  tools:
  - Llama-3.1-8B-Instruct
  - TTARAG
  - ms-marco-MiniLM-L-6-v2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# self-supervised-suffix-prediction-loss-construction

## Summary

Constructs the TTARAG self-supervised adaptation objective by splitting retrieved passages into prefix-suffix pairs and computing the token-level negative log-likelihood loss L_adapt = -sum_i log P(suffix_i | prefix_i, q; theta), with prefix tokens masked during label preparation so the model learns to predict retrieved content.

## When to use

Use when adapting an RAG system at test time to a specialized domain without labeled data: retrieved passages must be converted into a self-supervised prefix-suffix prediction loss for parameter updates.

## When NOT to use

- Retrieved passages are all shorter than the configured minimum length threshold, leaving no usable prefix-suffix pairs for adaptation
- The task requires domain adaptation with labeled data or full model retraining rather than a lightweight inference-time update
- You need a static evaluation pipeline with fixed parameters — TTARAG resets parameters per query, so it is unsuitable when per-query parameter mutation is undesirable

## Inputs

- Retrieved passages (e.g. top 20 sentences selected by MiniLM-v6 reranking from up to 50 HTML pages fetched via Brave Search API, or top 5 BM25 passages from a corpus)
- The input query q
- A pre-trained LLM such as Llama-3.1-8B-Instruct with tokenizer
- TTAConfig parameters: learning_rate (1e-6 to 1e-5), max_adapt_pairs (e.g. 3), minimum passage length threshold

## Outputs

- Prefix-suffix pairs per retrieved passage
- Adaptation loss L_adapt = -sum_i log P(suffix_i | prefix_i, q; theta)
- Updated model parameters (adapted LLM weights) used for final response generation

## How to apply

First filter out retrieved passages shorter than the configured minimum length threshold to ensure sufficient context for meaningful adaptation. Split each remaining passage into prefix-suffix pairs using the two-tier strategy: primary split at punctuation boundaries (periods, commas, semicolons, colons, exclamation marks, question marks) with a fallback midpoint split that ensures each segment has at least three words. Tokenize the full text and mask prefix tokens during label preparation, then compute L_adapt = -sum_i log P(suffix_i | prefix_i, q; theta) so loss applies only to suffix prediction given the prefix and the query. Accumulate gradients over two steps, clip to the maximum norm threshold, and update parameters with AdamW at a learning rate in the 1e-6 to 1e-5 range (e.g. 1e-5), resetting model parameters to their pre-trained state before each adaptation iteration and using up to max_adapt_pairs=3 pairs per query. The rationale is that predicting retrieved content provides an automatic, label-free signal for adjusting parameters to the target domain.

## Related tools

- **TTARAG** (Test-time adaptation framework whose self-supervised objective trains the model to predict retrieved content) — https://github.com/sunxin000/TTARAG
- **Llama-3.1-8B-Instruct** (Base LLM whose parameters are reset per query and updated via the prefix-suffix suffix-prediction loss)
- **ms-marco-MiniLM-L-6-v2** (Cross-encoder supplying filtered retrieval content from which prefix-suffix pairs are constructed)

## Examples

```
python parallel_evaluation.py --do_generate --dataset pubmedqa --model_name models/Llama-3.1-8B-Instruct --do_tta --learning_rate 1e-5 --max_adapt_pairs 3 --cross_encoder_name models/ms-marco-MiniLM-L-6-v2
```

## Evaluation signals

- Module runs end-to-end on a small batch of retrieved passages without errors
- Parameter updates follow the paper's Equation 2, with reset-to-pre-trained initialization before each adaptation iteration and two-step gradient accumulation with norm clipping
- Loss is computed only on suffix tokens: verify prefix tokens are masked in label preparation
- Each prefix-suffix segment has at least three words and the primary split lands on punctuation boundaries
- Produced tta_module.py matches the models/tta_module.py interface, with TTAConfig knobs learning_rate and max_adapt_pairs honored (e.g. --learning_rate 1e-5 --max_adapt_pairs 3)

## Limitations

- TTARAG performance is not sensitive to the number of adaptation pairs and learning rate, but 3-5 pairs with learning rates between 1e-6 to 1e-5 give the best effectiveness/efficiency balance
- Adaptation adds per-query processing time (1.75s at 1 pair to 2.60s at 5 pairs), requiring an NVIDIA A100-class GPU; 2,706 CRAG queries were processed on one A100
- Passages shorter than the minimum length threshold are discarded, so very short retrieval content yields no adaptation signal
- The paper does not name the six specialized domains or report quantitative metrics in the discussion section, and no changelog exists in the repository

## Evidence

- [methods] Length filtering: [evidence span withheld — anchor status: not_found]
- [methods] Hyperparameter ranges: [evidence span withheld — anchor status: not_found]
