---
name: prefix-suffix-self-supervised-passage-prediction
description: Use when a RAG system faces distribution shift in a specialized domain
  (e.g., BioASQ, PubMedQA) and must adapt during inference without new labeled data
  or model retraining.
when_to_use_negative:
- When labeled in-domain adaptation data is available and full supervised fine-tuning
  or retraining is feasible — TTARAG is designed for adaptation without new labeled
  data
- When retrieved passages are too short to yield meaningful prefix-suffix context
  (below the configured minimum length threshold they are filtered out)
- When the naive-RAG baseline already matches the target domain distribution and shows
  no suboptimal generalization from distribution shift
edam_operation: http://edamontology.org/operation_3281
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: TTARAG
  role: Test-time adaptation framework implementing the prefix-suffix self-supervised
    suffix-prediction training loop (models/tta_module.py)
  repo: https://github.com/sunxin000/TTARAG
- name: Pyserini
  role: Standard BM25 retriever used to fetch the top 5 relevant passages that supply
    the prefix-suffix pairs
- name: Llama-3.1-8B-Instruct
  role: Generation backbone whose parameters are reset and adapted via the suffix-prediction
    loop
- name: ms-marco-MiniLM-L-6-v2
  role: Cross-encoder used for sentence selection in the TTA-enabled RAGModel run
  repo: https://github.com/sunxin000/TTARAG
- name: NVIDIA A100 GPU
  role: Hardware for running the parameter adaptation and generation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1109/icassp55912.2026.11464478
    title: Predict the Retrieval! Test Time Adaptation for Retrieval Augmented Generation
  source_content_hash: sha256:367fd30d03e98d853b2c356be787ccff8f13a2801f2b26b26dd1866906302db1
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
  - ms-marco-MiniLM-L-6-v2
  edam_operation: http://edamontology.org/operation_3281
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# prefix-suffix self-supervised passage prediction

## Summary

Generates self-supervised adaptation signals for RAG by splitting retrieved passages into prefix-suffix pairs and training the language model at test time to predict suffixes from prefixes, automatically tuning parameters (theta -> theta') to the target domain without labeled data or retraining.

## When to use

Use when a RAG system faces distribution shift in a specialized domain (e.g., BioASQ, PubMedQA) and must adapt during inference without new labeled data or model retraining.

## When NOT to use

- When labeled in-domain adaptation data is available and full supervised fine-tuning or retraining is feasible — TTARAG is designed for adaptation without new labeled data
- When retrieved passages are too short to yield meaningful prefix-suffix context (below the configured minimum length threshold they are filtered out)
- When the naive-RAG baseline already matches the target domain distribution and shows no suboptimal generalization from distribution shift

## Inputs

- Question and top 5 BM25-retrieved passages (Pyserini) from a domain corpus (e.g., PubMed abstracts, biomedical articles)
- Pre-trained LLM weights (e.g., Llama-3.1-8B-Instruct, Llama-2-7b-chat, ChatGLM-3-6b) reset to pre-trained state
- TTAConfig settings: learning rate (1e-5), max_adapt_pairs (3), gradient accumulation steps (2), max norm clipping threshold
- Minimum passage length threshold for length filtering

## Outputs

- Adapted model parameters theta' for the current test question
- Final generated answer using adapted parameters theta'
- Prediction files saved under predictions/ and summaries under results/<dataset>/

## How to apply

Retrieve the top 5 relevant passages per question with a BM25 retriever (Pyserini) over the domain corpus, then apply a two-tier passage splitting strategy to build prefix-suffix pairs, first filtering out passages shorter than a configured minimum length threshold so there is sufficient context for meaningful adaptation. Reset model parameters to their original pre-trained state to ensure a clean starting point for each adaptation iteration, then adapt on a small number of pairs (3-5 works best; the paper uses 3 pairs with learning rate 1e-5): mask prefix tokens during label preparation, compute loss on the suffix-prediction task, accumulate gradients over two steps, clip them to a maximum norm threshold, and update parameters with AdamW. After adaptation, generate the final response using the adapted parameters theta'; the rationale is that the model learns to predict retrieved content, enabling automatic parameter adjustment to the target domain. The method is robust: performance is not sensitive to the number of adaptation pairs or learning rate, and processing overhead (1.75s-2.60s per query for 1-5 pairs) is far lower than Chain-of-Thought's 4.32s.

## Related tools

- **TTARAG** (Test-time adaptation framework implementing the prefix-suffix self-supervised suffix-prediction training loop (models/tta_module.py)) — https://github.com/sunxin000/TTARAG
- **Pyserini** (Standard BM25 retriever used to fetch the top 5 relevant passages that supply the prefix-suffix pairs)
- **Llama-3.1-8B-Instruct** (Generation backbone whose parameters are reset and adapted via the suffix-prediction loop)
- **ms-marco-MiniLM-L-6-v2** (Cross-encoder used for sentence selection in the TTA-enabled RAGModel run) — https://github.com/sunxin000/TTARAG
- **NVIDIA A100 GPU** (Hardware for running the parameter adaptation and generation)

## Examples

```
python parallel_evaluation.py --do_generate --dataset pubmedqa --model_name models/Llama-3.1-8B-Instruct --do_tta --learning_rate 1e-5 --max_adapt_pairs 3 --cross_encoder_name models/ms-marco-MiniLM-L-6-v2
```

## Evaluation signals

- Adapted-model (theta') answers must outscore the naive-RAG baseline on the target domain — e.g., BioASQ 75.0 vs 55.6 (+19.4%) and PubMedQA +10.8% for Llama-3.1-8b-it
- Confirm parameters were reset to the pre-trained state before each adaptation iteration (clean starting point per question)
- Verify gradient accumulation ran over 2 steps with norm clipping before each AdamW update
- LLM-as-a-judge (Qwen2.5-72B-Instruct) scoring should show ~98% agreement with human annotators on a random sample of 100 examples
- Per-query adaptation overhead should stay roughly in the 1.75s-2.60s range for 1-5 adaptation pairs, well below CoT's 4.32s

## Limitations

- The six specialized domains used in the experiments are not named in the discussion section, limiting direct transfer guidance
- Passages shorter than the configured minimum length threshold are filtered out, so very short retrieval results cannot drive adaptation
- Performance depends on retrieval quality; the method relies on retrieved content as the only self-supervised signal (no labeled data)
- No changelog found in the associated TTARAG repository, and base-model/implementation details are not fully specified in the discussion

## Evidence

- [methods] Optimal hyperparameters: [evidence span withheld — anchor status: not_found]
