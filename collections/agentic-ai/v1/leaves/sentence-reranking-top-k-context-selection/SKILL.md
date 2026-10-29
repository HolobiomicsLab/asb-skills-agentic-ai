---
name: sentence-reranking-top-k-context-selection
description: Use when retrieved context consists of many full HTML pages (e.g. up
  to 50 pages from the Brave Search API) that must be condensed into a small top-K
  sentence set before feeding an LLM for question answering.
when_to_use_negative:
- When the retrieval source already provides a small set of clean passages (e.g. BM25
  top-5 passages from a curated corpus), sentence-level reranking of HTML boilerplate
  adds overhead without benefit
- When the downstream model requires full-document context rather than a fixed top-20
  sentence budget
- When queries must be answered without web retrieval (e.g. closed-book prompting
  baselines such as CoT or ICL)
edam_operation: http://edamontology.org/operation_0306
tools:
- name: MiniLM-v6
  role: Cross-encoder used to rerank all sentences in the retrieved HTML pages before
    top-20 selection
- name: ms-marco-MiniLM-L-6-v2
  role: Concrete local model path/config for the reranking cross-encoder passed via
    --cross_encoder_name in parallel_evaluation.py
  repo: https://github.com/sunxin000/TTARAG
- name: Brave Search API
  role: Source of the up-to-50 full HTML pages per question that provide the sentences
    to be reranked
- name: TTARAG
  role: Downstream consumer of the top-20 sentence context, splitting it into prefix-suffix
    pairs for test-time adaptation
  repo: https://github.com/sunxin000/TTARAG
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1109/icassp55912.2026.11464478
    title: Predict the Retrieval! Test Time Adaptation for Retrieval Augmented Generation
  source_content_hash: sha256:20719a510e13e54f597017b031397a49dd717341751e58aa234b44ed0c6f9cc9
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
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# sentence-reranking-top-k-context-selection

## Summary

Reranks all sentences extracted from retrieved HTML pages with a lightweight cross-encoder (MiniLM-v6, implemented as ms-marco-MiniLM-L-6-v2) and keeps only the top 20 sentences as the retrieval content fed to the RAG model. This condenses up to 50 noisy web pages into a compact, high-relevance context for TTARAG and baseline RAG runs on CRAG.

## When to use

Use when retrieved context consists of many full HTML pages (e.g. up to 50 pages from the Brave Search API) that must be condensed into a small top-K sentence set before feeding an LLM for question answering.

## When NOT to use

- When the retrieval source already provides a small set of clean passages (e.g. BM25 top-5 passages from a curated corpus), sentence-level reranking of HTML boilerplate adds overhead without benefit
- When the downstream model requires full-document context rather than a fixed top-20 sentence budget
- When queries must be answered without web retrieval (e.g. closed-book prompting baselines such as CoT or ICL)

## Inputs

- Up to 50 full HTML pages retrieved per question via the Brave Search API
- Question/query string from the CRAG benchmark QA pair
- Cross-encoder model weights (ms-marco-MiniLM-L-6-v2 / allMiniLM) placed under models/

## Outputs

- Top 20 sentences ranked by cross-encoder relevance, used as the retrieval content for answer generation and test-time adaptation
- Per-sentence relevance scores (implicit in the ranking)

## How to apply

First retrieve up to 50 full HTML pages per question via the Brave Search API and split them into sentences. Then run the MiniLM-v6 cross-encoder (configured as models/ms-marco-MiniLM-L-6-v2 in the TTARAG repo via --cross_encoder_name) to score every sentence against the query, and select the top 20 sentences as the retrieval content. This reranked top-20 context is the input for both the naive-RAG baseline and TTARAG test-time adaptation, where passages are further split into prefix-suffix pairs. The rationale is that raw HTML pages contain mostly irrelevant boilerplate; sentence-level reranking maximizes signal density within the token budget that the 8B-class backbone and the suffix-prediction adaptation loop can process efficiently.

## Related tools

- **MiniLM-v6** (Cross-encoder used to rerank all sentences in the retrieved HTML pages before top-20 selection)
- **ms-marco-MiniLM-L-6-v2** (Concrete local model path/config for the reranking cross-encoder passed via --cross_encoder_name in parallel_evaluation.py) — https://github.com/sunxin000/TTARAG
- **Brave Search API** (Source of the up-to-50 full HTML pages per question that provide the sentences to be reranked)
- **TTARAG** (Downstream consumer of the top-20 sentence context, splitting it into prefix-suffix pairs for test-time adaptation) — https://github.com/sunxin000/TTARAG

## Examples

```
python parallel_evaluation.py --do_generate --dataset pubmedqa --model_name models/Llama-3.1-8B-Instruct --do_tta --learning_rate 1e-5 --max_adapt_pairs 3 --cross_encoder_name models/ms-marco-MiniLM-L-6-v2
```

## Evaluation signals

- The selected context contains exactly 20 sentences per question (top-K = 20)
- Reranking runs over all sentences from all retrieved HTML pages, not just the first page
- The cross-encoder checkpoint resolves to models/ms-marco-MiniLM-L-6-v2 (or the configured embedding model path) at inference time
- Downstream QA accuracy on CRAG (LLM-as-a-judge Qwen2.5-72B-Instruct scoring) is computed on answers generated from this reranked top-20 context
- Predictions saved under predictions/ and summaries under results/<dataset>/ reflect the reranked-context configuration

## Limitations

- The top-20 cap is a fixed budget; questions whose evidence is spread across many long pages may lose relevant sentences
- Reranking quality depends on the MiniLM checkpoint; the README requires manually prepared local weights (default models/allMiniLM) and will fail if they are missing
- The article does not report ablations over K, so the top-20 choice is not validated against other budgets

## Evidence
