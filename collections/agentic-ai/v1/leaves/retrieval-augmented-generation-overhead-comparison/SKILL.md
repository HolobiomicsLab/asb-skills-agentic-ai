---
name: retrieval-augmented-generation-overhead-comparison
description: Use when you must quantify whether a test-time adaptation step adds acceptable
  inference latency relative to prompting baselines (CoT, ICL, naive-RAG) on a fixed
  hardware/GPU setup.
when_to_use_negative:
- The goal is to compare answer quality/accuracy metrics (e.g., LLM-judger correctness
  scores) rather than computational overhead — use the quality-evaluation pipeline
  instead.
- Input questions lack pre-retrieved content and the retrieval stage (Brave Search
  API + MiniLM-v6 reranking) has not been run; retrieval time would confound the inference-time
  comparison.
- Hardware differs from the single NVIDIA A100 setup used to produce the reference
  profile (1.75s–2.60s TTARAG vs 4.32s CoT), making the reported per-query benchmarks
  non-comparable.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: TTARAG
  role: Test-time adaptation module (models/tta_module.py) whose per-query adaptation
    cost is being measured across 1–5 adaptation pairs
  repo: https://github.com/sunxin000/TTARAG
- name: Llama-3.1-8B-Instruct
  role: Generation model used for naive-RAG, CoT, and TTARAG configurations so timing
    differences isolate the adaptation/prompting overhead
  repo: https://github.com/sunxin000/TTARAG
- name: NVIDIA A100 GPU
  role: Single-GPU hardware reference on which the 2,706-query CRAG benchmark timings
    are measured
- name: MiniLM-v6
  role: Sentence reranker that produces the top 20 retrieval-content sentences used
    as pre-retrieved input to all timed configurations
- name: CRAG
  role: 2,706-question evaluation benchmark providing the fixed query set for wall-clock
    timing
  repo: https://github.com/facebookresearch/CRAG
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1109/icassp55912.2026.11464478
    title: Predict the Retrieval! Test Time Adaptation for Retrieval Augmented Generation
  source_content_hash: sha256:38cda46540d76063cc6c26fde72e04aa7d79e5af6172a240d81e2058f50a4760
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sunxin000/TTARAG
  tools:
  - CRAG
  - Llama-3.1-8B-Instruct
  - MiniLM-v6
  - NVIDIA A100 GPU
  - TTARAG
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# retrieval-augmented-generation-overhead-comparison

## Summary

Benchmark and tabulate the inference-time cost per query of test-time-adapted RAG (TTARAG, 1–5 adaptation pairs) against Chain-of-Thought and naive-RAG baselines, using the CRAG benchmark on a single NVIDIA A100 GPU with Llama-3.1-8B-Instruct.

## When to use

Use when you must quantify whether a test-time adaptation step adds acceptable inference latency relative to prompting baselines (CoT, ICL, naive-RAG) on a fixed hardware/GPU setup.

## When NOT to use

- The goal is to compare answer quality/accuracy metrics (e.g., LLM-judger correctness scores) rather than computational overhead — use the quality-evaluation pipeline instead.
- Input questions lack pre-retrieved content and the retrieval stage (Brave Search API + MiniLM-v6 reranking) has not been run; retrieval time would confound the inference-time comparison.
- Hardware differs from the single NVIDIA A100 setup used to produce the reference profile (1.75s–2.60s TTARAG vs 4.32s CoT), making the reported per-query benchmarks non-comparable.

## Inputs

- CRAG benchmark (2,706 question-answer pairs) with pre-retrieved content (top 20 MiniLM-v6-reranked sentences per query)
- Llama-3.1-8B-Instruct model weights placed at models/Llama-3.1-8B-Instruct
- TTARAG repository scripts: parallel_evaluation.py, cot_baseline.py, models/tta_module.py
- One NVIDIA A100 GPU

## Outputs

- Table of total wall-clock inference time (seconds) and average per-query time (seconds) for 1-pair to 5-pair TTARAG, CoT, and naive-RAG
- Prediction files under predictions/ and summaries under results/<dataset>/ from parallel_evaluation.py

## How to apply

Run the full evaluation on the 2,706-question CRAG benchmark with its pre-retrieved content (Brave Search HTML pages reranked by MiniLM-v6 to the top 20 sentences) via parallel_evaluation.py in the TTARAG repository. First time naive-RAG generation with Llama-3.1-8B-Instruct (no adaptation, RAGModel class, batch_size 1, tensor_parallel_size 1) to establish the fastest baseline. Then run the Chain-of-Thought baseline (cot_baseline.py) on the same queries and GPU, recording total and per-query wall-clock time. Finally run TTARAG with --do_tta for 1 through 5 max_adapt_pairs, using a learning rate in the optimal 1e-6 to 1e-5 range and resetting model parameters to the pre-trained state before each query's adaptation, recording total inference time per setting. Aggregate total seconds and average per-query seconds into a table for 1-pair through 5-pair TTARAG, CoT, and naive-RAG; the expected profile is ~1.75s/query (1-pair) to ~2.60s/query (5-pair), both below CoT's 4.32s.

## Related tools

- **TTARAG** (Test-time adaptation module (models/tta_module.py) whose per-query adaptation cost is being measured across 1–5 adaptation pairs) — https://github.com/sunxin000/TTARAG
- **Llama-3.1-8B-Instruct** (Generation model used for naive-RAG, CoT, and TTARAG configurations so timing differences isolate the adaptation/prompting overhead) — https://github.com/sunxin000/TTARAG
- **NVIDIA A100 GPU** (Single-GPU hardware reference on which the 2,706-query CRAG benchmark timings are measured)
- **MiniLM-v6** (Sentence reranker that produces the top 20 retrieval-content sentences used as pre-retrieved input to all timed configurations)
- **CRAG** (2,706-question evaluation benchmark providing the fixed query set for wall-clock timing) — https://github.com/facebookresearch/CRAG

## Examples

```
python parallel_evaluation.py --do_generate --dataset crag --model_name models/Llama-3.1-8B-Instruct --do_tta --learning_rate 1e-5 --max_adapt_pairs 3 --cross_encoder_name models/ms-marco-MiniLM-L-6-v2
```

## Evaluation signals

- Average per-query time for 1-pair TTARAG lands near 1.75s and 5-pair near 2.60s on the 2,706-query CRAG benchmark.
- Both TTARAG settings (1.75s–2.60s per query) fall substantially below Chain-of-Thought's 4.32s per query.
- Naive-RAG (no adaptation, no TTA) is the fastest configuration in the table.
- Per-query time increases monotonically with the number of adaptation pairs from 1 to 5.
- All configurations run on the same hardware (one NVIDIA A100) and same 2,706-query set so timing differences are attributable to the method.

## Limitations

- Reported timings are specific to one NVIDIA A100 GPU; other hardware will not reproduce the 1.75s–2.60s per-query profile.
- TTARAG's overhead scales with the number of adaptation pairs; 3–5 pairs give the best effectiveness/efficiency balance per the article.
- Performance of TTARAG is not sensitive to the number of adaptation pairs and learning rate, so small timing variations across settings should not be over-interpreted.

## Evidence

- [methods] Optimal learning-rate range for the adaptation runs: [evidence span withheld — anchor status: not_found]
