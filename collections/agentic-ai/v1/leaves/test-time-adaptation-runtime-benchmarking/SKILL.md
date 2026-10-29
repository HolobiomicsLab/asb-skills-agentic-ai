---
name: test-time-adaptation-runtime-benchmarking
description: Use when you must quantify the computational cost of a test-time adaptation
  loop added to a RAG pipeline, e.g. comparing average seconds per query for 1-5 adaptation
  pairs against CoT or naive-RAG on one A100 GPU.
when_to_use_negative:
- You need end-to-end training-time cost, not inference-time cost — this skill only
  measures per-query adaptation and generation on a pre-trained model.
- Your benchmark differs in size or retrieval setup from the 2,706-query CRAG configuration;
  per-query timings will not be comparable to the published 1.75s-2.60s profile.
- The task targets accuracy/quality metrics rather than runtime; use the main evaluation
  workflow instead.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: TTARAG
  role: Test-time adaptation framework whose parallel_evaluation.py entry point runs
    timed inference with --do_tta, --learning_rate, and --max_adapt_pairs options
  repo: https://github.com/sunxin000/TTARAG
- name: Llama-3.1-8B-Instruct
  role: Generation model whose per-query adaptation and response generation is being
    timed
- name: NVIDIA A100 GPU
  role: Single-GPU hardware reference for all timed runs, matching the paper's measurement
    setup
- name: MiniLM-v6
  role: Sentence reranker that produces the top-20-sentence retrieval content fed
    to the timed RAG configurations
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1109/icassp55912.2026.11464478
    title: Predict the Retrieval! Test Time Adaptation for Retrieval Augmented Generation
  source_content_hash: sha256:f60ee56e254d50b48fd170a2613e40a51bbf0a2a3789970088f44592cf76325a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sunxin000/TTARAG
  tools:
  - Llama-3.1-8B-Instruct
  - MiniLM-v6
  - NVIDIA A100 GPU
  - TTARAG
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# test-time-adaptation-runtime-benchmarking

## Summary

Benchmark the wall-clock inference-time overhead of test-time adaptation for RAG (TTARAG) against baseline prompting methods (Chain-of-Thought, naive-RAG) on identical queries and hardware. It establishes that per-query adaptation (1-5 pairs) stays cheaper than CoT while improving answer quality.

## When to use

Use when you must quantify the computational cost of a test-time adaptation loop added to a RAG pipeline, e.g. comparing average seconds per query for 1-5 adaptation pairs against CoT or naive-RAG on one A100 GPU.

## When NOT to use

- You need end-to-end training-time cost, not inference-time cost — this skill only measures per-query adaptation and generation on a pre-trained model.
- Your benchmark differs in size or retrieval setup from the 2,706-query CRAG configuration; per-query timings will not be comparable to the published 1.75s-2.60s profile.
- The task targets accuracy/quality metrics rather than runtime; use the main evaluation workflow instead.

## Inputs

- CRAG benchmark: 2,706 question-answer pairs with pre-retrieved content (Brave Search HTML pages reranked by MiniLM-v6 to top 20 sentences)
- Llama-3.1-8B-Instruct model weights
- TTARAG repository code (parallel_evaluation.py, models/tta_module.py)
- One NVIDIA A100 GPU

## Outputs

- Table of total inference time (seconds) and average per-query time (seconds) for 1-pair through 5-pair TTARAG, CoT, and naive-RAG
- Predictions saved under predictions/ and summaries under results/<dataset>/

## How to apply

Run all configurations on the same 2,706-query CRAG benchmark with pre-retrieved content (Brave Search HTML pages reranked by MiniLM-v6 to the top 20 sentences) on one NVIDIA A100 GPU. Execute naive-RAG generation with Llama-3.1-8B-Instruct via the TTARAG repository's parallel_evaluation.py and record total wall-clock time; repeat for the Chain-of-Thought baseline; then run TTARAG for each adaptation-pair setting from 1 to 5 pairs using a learning rate in the optimal 1e-6 to 1e-5 range, resetting model parameters to the pre-trained state per query so timing reflects per-query adaptation rather than cumulative drift. Aggregate totals into a table with total (seconds) and average per-query (seconds) columns for 1-pair through 5-pair TTARAG, CoT, and naive-RAG. The rationale is that adaptation cost scales with the number of prefix-suffix pairs, so the 1-to-5-pair sweep brackets the efficiency frontier reported in the paper.

## Related tools

- **TTARAG** (Test-time adaptation framework whose parallel_evaluation.py entry point runs timed inference with --do_tta, --learning_rate, and --max_adapt_pairs options) — https://github.com/sunxin000/TTARAG
- **Llama-3.1-8B-Instruct** (Generation model whose per-query adaptation and response generation is being timed)
- **NVIDIA A100 GPU** (Single-GPU hardware reference for all timed runs, matching the paper's measurement setup)
- **MiniLM-v6** (Sentence reranker that produces the top-20-sentence retrieval content fed to the timed RAG configurations)

## Examples

```
python parallel_evaluation.py --do_generate --dataset crag --model_name models/Llama-3.1-8B-Instruct --do_tta --learning_rate 1e-5 --max_adapt_pairs 3 --cross_encoder_name models/ms-marco-MiniLM-L-6-v2
```

## Evaluation signals

- Average per-query time for 1-pair TTARAG falls around 1.75s and for 5-pair around 2.60s on the 2,706-query CRAG benchmark.
- Both 1-pair and 5-pair TTARAG average times are substantially below Chain-of-Thought's 4.32s per query.
- Naive-RAG (no adaptation) is the fastest configuration, confirming adaptation adds measurable but bounded overhead.
- Average per-query time increases monotonically with the number of adaptation pairs (1 through 5).
- Learning rate used lies within the reported optimal 1e-6 to 1e-5 range and parameters are reset to the pre-trained state per query, so timings are not contaminated by cross-query state.

## Limitations

- Timings are hardware-specific: results were measured on one NVIDIA A100 GPU and will not transfer directly to other accelerators or CPU inference.
- The runtime profile is tied to the CRAG benchmark's 2,706 queries and its pre-retrieval pipeline; different retrieval depths or corpora change per-query cost.
- The number of adaptation pairs trades efficiency against effectiveness — 3-5 pairs is the paper's best balance — so extreme settings may not reflect the reported profile.

## Evidence

- [methods] learning-rate range: [evidence span withheld — anchor status: not_found]
