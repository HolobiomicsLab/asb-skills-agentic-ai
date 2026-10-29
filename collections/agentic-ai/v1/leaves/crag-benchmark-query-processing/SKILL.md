---
name: crag-benchmark-query-processing
description: Use when a RAG experiment needs the CRAG question-answer pairs paired
  with pre-retrieved, sentence-level search content, or when inference-time cost per
  query must be benchmarked against prompting baselines like CoT.
when_to_use_negative:
- The study needs raw end-to-end retrieval variation (BM25 top-5 passages via Pyserini)
  rather than a fixed pre-retrieved, reranked pool
- The target dataset is not one supported by the repository's dataset adapters (e.g.
  custom corpora not wired into DatasetConfig)
- Evaluation requires human-rated answer quality (perfect/acceptable/missing/incorrect)
  rather than the automated correct/missing/incorrect scoring
edam_operation: http://edamontology.org/operation_0306
tools:
- name: parallel_evaluation.py
  role: Runs parallel generation and evaluation over the benchmark questions and saves
    predictions and summaries
  repo: https://github.com/sunxin000/TTARAG
- name: Llama-3.1-8B-Instruct
  role: Generation model for RAG responses, passed via --model_name
  repo: https://github.com/sunxin000/TTARAG
- name: MiniLM-v6 / ms-marco-MiniLM-L-6-v2
  role: Cross-encoder reranking of HTML sentences, selecting the top 20 sentences
    as retrieval content
  repo: https://github.com/sunxin000/TTARAG
- name: Brave Search API
  role: Source of up to 50 full HTML pages per query in the retrieval content attached
    to CRAG questions
- name: Qwen2.5-72B-Instruct
  role: LLM-as-a-judge determining semantic equivalence between predictions and reference
    answers
- name: NVIDIA A100 GPU
  role: Single-GPU hardware used to time inference over the 2,706 queries
- name: CRAG
  role: Benchmark providing the 2,706 question-answer pairs, dataset guide, and auto-evaluation
    scripts
  repo: https://github.com/facebookresearch/CRAG/blob/main/docs/dataset.md
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1109/icassp55912.2026.11464478
    title: Predict the Retrieval! Test Time Adaptation for Retrieval Augmented Generation
  source_content_hash: sha256:3181ffb2e8edc3e5f1623c2ab560a7895f0be69ddb6d7db01c071ba7148ac52c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sunxin000/TTARAG
  tools:
  - Brave Search API
  - CRAG
  - Llama-3.1-8B-Instruct
  - MiniLM-v6
  - MiniLM-v6 / ms-marco-MiniLM-L-6-v2
  - NVIDIA A100 GPU
  - Qwen2.5-72B-Instruct
  - TTARAG
  - parallel_evaluation.py
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# crag-benchmark-query-processing

## Summary

Loads and prepares the 2,706-question CRAG benchmark with its pre-retrieved content — Brave Search HTML pages reranked by MiniLM-v6 down to the top 20 sentences — so that RAG methods such as TTARAG, naive-RAG, and Chain-of-Thought can be evaluated under identical retrieval conditions on one NVIDIA A100 GPU.

## When to use

Use when a RAG experiment needs the CRAG question-answer pairs paired with pre-retrieved, sentence-level search content, or when inference-time cost per query must be benchmarked against prompting baselines like CoT.

## When NOT to use

- The study needs raw end-to-end retrieval variation (BM25 top-5 passages via Pyserini) rather than a fixed pre-retrieved, reranked pool
- The target dataset is not one supported by the repository's dataset adapters (e.g. custom corpora not wired into DatasetConfig)
- Evaluation requires human-rated answer quality (perfect/acceptable/missing/incorrect) rather than the automated correct/missing/incorrect scoring

## Inputs

- CRAG benchmark: 2,706 question-answer pairs with pre-retrieved Brave Search HTML content (facebookresearch/CRAG dataset guide)
- Retrieval content reranked by MiniLM-v6 (ms-marco-MiniLM-L-6-v2), top 20 sentences per question
- Llama-3.1-8B-Instruct model weights placed at models/Llama-3.1-8B-Instruct
- PubMedQA (pqa_labeled split via qiaojin/PubMedQA) as an alternative dataset handled by dataset/PubMedQAAdapter

## Outputs

- Per-query predictions saved under predictions/ and summaries under results/<dataset>/
- Total and average per-query inference times (seconds) for each configuration (1-pair through 5-pair TTARAG, naive-RAG, CoT)
- Correctness scores using the CRAG scheme: correct (+1), missing (0), incorrect (−1)

## How to apply

Load the 2,706 question-answer pairs from the CRAG benchmark and attach each question's retrieval content: up to 50 full HTML pages fetched through the Brave Search API, with all sentences reranked by MiniLM-v6 (ms-marco-MiniLM-L-6-v2 cross-encoder) and the top 20 sentences kept. Run generation with Llama-3.1-8B-Instruct via the TTARAG repository's parallel_evaluation.py, choosing the model class through models/user_config.py (default RAGModel) and toggling TTA with --do_tta, --learning_rate (within the optimal 1e-6 to 1e-5 range), and --max_adapt_pairs (1–5). For efficiency profiling, record total wall-clock time on one NVIDIA A100 GPU for each configuration and divide by 2,706 to get average per-query time. Scores come from the CRAG evaluation scheme: correct (+1), missing (0), incorrect (−1), using rule-based matching plus an LLM judge (Qwen2.5-72B-Instruct) checking semantic equivalence with reference answers. The rationale is that a fixed, pre-reranked retrieval pool isolates the effect of the generation/adaptation method from retrieval variance.

## Related tools

- **parallel_evaluation.py** (Runs parallel generation and evaluation over the benchmark questions and saves predictions and summaries) — https://github.com/sunxin000/TTARAG
- **Llama-3.1-8B-Instruct** (Generation model for RAG responses, passed via --model_name) — https://github.com/sunxin000/TTARAG
- **MiniLM-v6 / ms-marco-MiniLM-L-6-v2** (Cross-encoder reranking of HTML sentences, selecting the top 20 sentences as retrieval content) — https://github.com/sunxin000/TTARAG
- **Brave Search API** (Source of up to 50 full HTML pages per query in the retrieval content attached to CRAG questions)
- **Qwen2.5-72B-Instruct** (LLM-as-a-judge determining semantic equivalence between predictions and reference answers)
- **NVIDIA A100 GPU** (Single-GPU hardware used to time inference over the 2,706 queries)
- **CRAG** (Benchmark providing the 2,706 question-answer pairs, dataset guide, and auto-evaluation scripts) — https://github.com/facebookresearch/CRAG/blob/main/docs/dataset.md

## Examples

```
python parallel_evaluation.py --do_generate --dataset pubmedqa --split train --model_name models/Llama-3.1-8B-Instruct --batch_size 1 --tensor_parallel_size 1 --gpu_memory_utilization 0.85
```

## Evaluation signals

- Total query count processed equals 2,706 (or the full chosen dataset split) with one prediction per question saved under predictions/
- Average per-query time for 1-pair TTARAG lands near 1.75s and 5-pair near 2.60s, both below Chain-of-Thought's 4.32s
- Naive-RAG (no --do_tta) is the fastest configuration in the timing table
- Learning-rate settings stay within the optimal 1e-6 to 1e-5 range and parameter resets occur per query before each adaptation
- Scores conform to the CRAG auto-evaluation scheme: correct (+1), missing (0), incorrect (−1)

## Limitations

- Performance of TTARAG is reported as not sensitive to the number of adaptation pairs and learning rate, so timing differences across 1–5 pairs are modest
- The LLM judge (Qwen2.5-72B-Instruct) showed 98% agreement with human annotators on only a random sample of 100 examples, so automated scoring is not perfect
- CRAG requires downloading the official dataset separately (see docs/dataset.md) and its license is CC BY-NC 4.0, restricting commercial use

## Evidence

- [methods] CRAG contains 2,706 question-answer pairs: [evidence span withheld — anchor status: not_found]
- [methods] MiniLM-v6 reranking to top 20 sentences: "Then MiniLM-v6 is employed to reranker all the sentence in the HTML and select top 20 sentences as the retrieval content"
- [methods] Timing findings per query: "The average processing time per query ranges from 1.75s (1-pair) to 2.60s (5-pair), which is substantially lower than CoT’s 4.32s"
- [readme] Parallel evaluation entry point: "parallel_evaluation.py: parallel evaluation and result saving"
- [readme] CRAG scoring scheme: [evidence span withheld — anchor status: not_found]
- [readme] TTA CLI options for the benchmark run: "--do_tta \
  --learning_rate 1e-5 \
  --max_adapt_pairs 3 \
  --cross_encoder_name models/ms-marco-MiniLM-L-6-v2"
