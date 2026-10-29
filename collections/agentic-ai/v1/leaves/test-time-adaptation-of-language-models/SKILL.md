---
name: test-time-adaptation-of-language-models
description: Use when a RAG system must answer queries from a specialized domain (e.g.,
  CRAG's Finance/Sports/Music/Movie/Open) where the backbone LLM underperforms due
  to distribution shift and no domain training data exists.
when_to_use_negative:
- When labeled domain training data is available for full fine-tuning or the model
  can be retrained offline — TTARAG is designed for adaptation without new labeled
  data or model retraining
- When retrieved passages are too short to form meaningful prefix-suffix context;
  passages below the minimum length threshold are filtered out and adaptation quality
  degrades
- When inference-time compute per query is strictly capped well below the adapted-parameters
  cost (1.75s-2.60s per query for 1-5 pairs), such as latency-critical production
  serving
edam_operation: http://edamontology.org/operation_0305
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: TTARAG
  role: Core test-time adaptation module (models/tta_module.py) implementing prefix-suffix
    self-supervised training and adapted-parameter generation
  repo: https://github.com/sunxin000/TTARAG
- name: parallel_evaluation.py
  role: Entry point for parallel generation/evaluation with --do_tta, --learning_rate,
    and --max_adapt_pairs options
  repo: https://github.com/sunxin000/TTARAG
- name: Llama-3.1-8B-Instruct
  role: Default backbone generation model whose parameters are adapted per query
  repo: https://github.com/sunxin000/TTARAG
- name: ms-marco-MiniLM-L-6-v2
  role: Cross-encoder used for sentence selection passed via --cross_encoder_name
  repo: https://github.com/sunxin000/TTARAG
- name: MiniLM-v6
  role: Reranks all sentences in the retrieved HTML and selects the top 20 sentences
    as retrieval content
- name: Brave Search API
  role: Source of up to 50 full HTML pages per question used as adaptation and retrieval
    material
- name: Qwen2.5-72B-Instruct
  role: LLM-as-a-judge scoring semantic equivalence between predictions and reference
    answers
- name: NVIDIA A100 GPU
  role: Single-GPU hardware for running 2,706 CRAG queries with adaptation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1109/icassp55912.2026.11464478
    title: Predict the Retrieval! Test Time Adaptation for Retrieval Augmented Generation
  source_content_hash: sha256:dc490729e8804733eed387dd06f5850f020a39d44f2ac8cfe4e23368d3f717f7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sunxin000/TTARAG
  tools:
  - Brave Search API
  - Llama-3.1-8B-Instruct
  - MiniLM-v6
  - NVIDIA A100 GPU
  - Qwen2.5-72B-Instruct
  - TTARAG
  - ms-marco-MiniLM-L-6-v2
  - parallel_evaluation.py
  edam_operation: http://edamontology.org/operation_0305
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# test-time-adaptation-of-language-models

## Summary

TTARAG adapts a retrieval-augmented language model's parameters at inference time via self-supervised suffix-prediction on retrieved passages, countering distribution shifts in specialized domains without new labeled data or retraining.

## When to use

Use when a RAG system must answer queries from a specialized domain (e.g., CRAG's Finance/Sports/Music/Movie/Open) where the backbone LLM underperforms due to distribution shift and no domain training data exists.

## When NOT to use

- When labeled domain training data is available for full fine-tuning or the model can be retrained offline — TTARAG is designed for adaptation without new labeled data or model retraining
- When retrieved passages are too short to form meaningful prefix-suffix context; passages below the minimum length threshold are filtered out and adaptation quality degrades
- When inference-time compute per query is strictly capped well below the adapted-parameters cost (1.75s-2.60s per query for 1-5 pairs), such as latency-critical production serving

## Inputs

- A benchmark QA set (e.g., CRAG's 2,706 question-answer pairs across five domains) or PubMedQA (pqa_labeled split)
- Retrieved passages: up to 50 full HTML pages per question from the Brave Search API, or top 5 BM25 passages via Pyserini
- A pretrained backbone LLM such as Llama-3.1-8B-Instruct with local weights placed under models/
- A cross-encoder for sentence selection (models/ms-marco-MiniLM-L-6-v2) and sentence embedding model (models/allMiniLM)

## Outputs

- Generated answers per query produced with adapted parameters
- Prediction files under predictions/ and per-dataset summaries under results/<dataset>/
- Accuracy aggregates per domain and overall (e.g., crag_accuracy_table.csv) judged by Qwen2.5-72B-Instruct semantic equivalence

## How to apply

For each query, first reset the model parameters to their pre-trained state, then split the retrieved passages into prefix-suffix pairs using a two-tier strategy, filtering out passages shorter than the configured minimum length threshold. Train with the suffix-prediction loss (prefix tokens masked during label preparation) using AdamW with gradient accumulation over two steps and max-norm gradient clipping; learning rates between 1e-6 and 1e-5 and 3-5 adaptation pairs are optimal, and the method is not highly sensitive to these hyperparameters. After adaptation, generate the final response with the adapted parameters. In the paper's CRAG setup, up to 50 HTML pages retrieved via the Brave Search API are reranked sentence-wise with MiniLM-v6 and the top 20 sentences form the retrieval content. Score predictions with an LLM-as-a-judge (Qwen2.5-72B-Instruct) checking semantic equivalence against reference answers.

## Related tools

- **TTARAG** (Core test-time adaptation module (models/tta_module.py) implementing prefix-suffix self-supervised training and adapted-parameter generation) — https://github.com/sunxin000/TTARAG
- **parallel_evaluation.py** (Entry point for parallel generation/evaluation with --do_tta, --learning_rate, and --max_adapt_pairs options) — https://github.com/sunxin000/TTARAG
- **Llama-3.1-8B-Instruct** (Default backbone generation model whose parameters are adapted per query) — https://github.com/sunxin000/TTARAG
- **ms-marco-MiniLM-L-6-v2** (Cross-encoder used for sentence selection passed via --cross_encoder_name) — https://github.com/sunxin000/TTARAG
- **MiniLM-v6** (Reranks all sentences in the retrieved HTML and selects the top 20 sentences as retrieval content)
- **Brave Search API** (Source of up to 50 full HTML pages per question used as adaptation and retrieval material)
- **Qwen2.5-72B-Instruct** (LLM-as-a-judge scoring semantic equivalence between predictions and reference answers)
- **NVIDIA A100 GPU** (Single-GPU hardware for running 2,706 CRAG queries with adaptation)

## Examples

```
python parallel_evaluation.py --do_generate --dataset pubmedqa --model_name models/Llama-3.1-8B-Instruct --do_tta --learning_rate 1e-5 --max_adapt_pairs 3 --cross_encoder_name models/ms-marco-MiniLM-L-6-v2
```

## Evaluation signals

- The LLM-judger should show ~98% agreement with human annotators on a random 100-example sample
- TTARAG should beat naive-RAG, CoT, and ICL baselines in most settings (best in 19 of 24 experimental settings across six specialized domains)
- Average processing time per query should fall roughly in the 1.75s (1-pair) to 2.60s (5-pair) range, below CoT's 4.32s
- Parameter reset per query must be verified so each adaptation starts from the pre-trained state; results should be stable across learning rates 1e-6 to 1e-5 and 3-5 adaptation pairs
- Prediction files must land under predictions/ with summaries under results/<dataset>/ when reusing via --predictions_path without --do_generate

## Limitations

- The six specialized domains and quantitative improvements are not fully detailed in the discussion; verify domain coverage and metrics from the full tables before reuse
- Performance is not sensitive to the number of adaptation pairs and learning rate within the tested range, but extremely short passages are filtered out and cannot support meaningful adaptation
- The base language model and implementation details of TTARAG may not be specified in all paper sections; consult models/config.py (TTAConfig, RAGConfig) in the repository

## Evidence

- [abstract] we propose TTARAG, a test-time adaptation method that dynamically updates the language model's parameters during inference to improve RAG system performance in specialized domains.: [evidence span withheld — anchor status: not_found]
- [methods] passages shorter than a configured minimum length threshold are filtered out: [evidence span withheld — anchor status: not_found]
- [methods] learning rates between 1e-6 to 1e-5 provide optimal performance, with 3-5 adaptation pairs yielding the best balance between effectiveness and efficiency: [evidence span withheld — anchor status: not_found]
