---
name: llm-inference-latency-profiling
description: Use when comparing computational cost of an inference-time adaptation
  method against baseline prompting methods, and you need per-query seconds measured
  on identical hardware (one NVIDIA A100) over a fixed benchmark.
when_to_use_negative:
- Evaluating answer quality (perfect/acceptable/missing/incorrect CRAG scoring) rather
  than inference-time cost — use the CRAG evaluation workflow instead
- Measuring training-time or pre-computation cost of the method; this skill profiles
  only per-query inference latency on a single A100
- Benchmarking on multi-GPU or tensor-parallel setups where the 1.75s-2.60s per-query
  profile does not transfer
tools:
- name: TTARAG
  role: Test-time adaptation method whose per-query latency is profiled across 1-5
    adaptation pairs
  repo: https://github.com/sunxin000/TTARAG
- name: Llama-3.1-8B-Instruct
  role: Generation model timed in both naive-RAG and TTARAG configurations
  repo: https://github.com/sunxin000/TTARAG
- name: NVIDIA A100 GPU
  role: Single-GPU hardware platform for all latency measurements
- name: parallel_evaluation.py
  role: TTARAG repository script used to run parallel evaluation and result saving
    from which wall-clock time is recorded
  repo: https://github.com/sunxin000/TTARAG
- name: MiniLM-v6
  role: Reranker producing the top-20-sentence retrieval content fed to the timed
    pipelines
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1109/icassp55912.2026.11464478
    title: Predict the Retrieval! Test Time Adaptation for Retrieval Augmented Generation
  source_content_hash: sha256:81805b64b2454cdcd784ab3e3f6d5dbe2150203fbe4774df6a66f6e3401b7f1c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sunxin000/TTARAG
  tools:
  - Llama-3.1-8B-Instruct
  - MiniLM-v6
  - NVIDIA A100 GPU
  - TTARAG
  - parallel_evaluation.py
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# llm-inference-latency-profiling

## Summary

Measures wall-clock inference latency per query for TTARAG adaptation-pair settings versus baseline prompting (Chain-of-Thought, naive-RAG) on one NVIDIA A100 GPU over the 2,706-question CRAG benchmark, to verify that test-time adaptation stays computationally efficient relative to CoT's 4.32s per query.

## When to use

Use when comparing computational cost of an inference-time adaptation method against baseline prompting methods, and you need per-query seconds measured on identical hardware (one NVIDIA A100) over a fixed benchmark.

## When NOT to use

- Evaluating answer quality (perfect/acceptable/missing/incorrect CRAG scoring) rather than inference-time cost — use the CRAG evaluation workflow instead
- Measuring training-time or pre-computation cost of the method; this skill profiles only per-query inference latency on a single A100
- Benchmarking on multi-GPU or tensor-parallel setups where the 1.75s-2.60s per-query profile does not transfer

## Inputs

- 2,706-question CRAG benchmark with pre-retrieved content (Brave Search HTML pages reranked by MiniLM-v6 to top 20 sentences)
- Llama-3.1-8B-Instruct generation model weights (models/Llama-3.1-8B-Instruct)
- Cross-encoder ms-marco-MiniLM-L-6-v2 for TTA sentence selection
- One NVIDIA A100 GPU

## Outputs

- Total wall-clock inference time (seconds) per method and per TTARAG adaptation-pair setting
- Average processing time per query (seconds) for 1-pair through 5-pair TTARAG, CoT, and naive-RAG
- Latency comparison table showing TTARAG averages below CoT's 4.32s per query

## How to apply

Load the 2,706-question CRAG benchmark with pre-retrieved content (Brave Search HTML pages reranked by MiniLM-v6 to the top 20 sentences). Run naive-RAG generation with Llama-3.1-8B-Instruct via parallel_evaluation.py on one A100, recording total wall-clock time, then run the Chain-of-Thought baseline on the same queries. Run TTARAG for each adaptation-pair setting from 1 to 5 pairs using a learning rate in the optimal 1e-6 to 1e-5 range, resetting model parameters to their pre-trained state per query, and record total inference time per setting. Aggregate into a table with total and average per-query seconds for 1-pair through 5-pair TTARAG, CoT, and naive-RAG; judge correctness by checking that averages track the reported profile (1.75s at 1-pair, 2.60s at 5-pair, both below CoT's 4.32s) with naive-RAG fastest.

## Related tools

- **TTARAG** (Test-time adaptation method whose per-query latency is profiled across 1-5 adaptation pairs) — https://github.com/sunxin000/TTARAG
- **Llama-3.1-8B-Instruct** (Generation model timed in both naive-RAG and TTARAG configurations) — https://github.com/sunxin000/TTARAG
- **NVIDIA A100 GPU** (Single-GPU hardware platform for all latency measurements)
- **parallel_evaluation.py** (TTARAG repository script used to run parallel evaluation and result saving from which wall-clock time is recorded) — https://github.com/sunxin000/TTARAG
- **MiniLM-v6** (Reranker producing the top-20-sentence retrieval content fed to the timed pipelines)

## Examples

```
python parallel_evaluation.py --do_generate --dataset pubmedqa --model_name models/Llama-3.1-8B-Instruct --do_tta --learning_rate 1e-5 --max_adapt_pairs 3 --cross_encoder_name models/ms-marco-MiniLM-L-6-v2
```

## Evaluation signals

- Average per-query time for 1-pair TTARAG is approximately 1.75s and 5-pair approximately 2.60s
- All TTARAG adaptation-pair settings (1-5 pairs) average below CoT's 4.32s per query
- Naive-RAG baseline is the fastest configuration in the latency table
- Latency scales monotonically with the number of adaptation pairs (1 to 5)
- All measurements come from the same 2,706-query CRAG set on one NVIDIA A100 GPU

## Limitations

- Latency figures are specific to one NVIDIA A100 GPU and the 2,706-query CRAG benchmark; they do not transfer to other hardware
- TTARAG requires resetting model parameters to the pre-trained state per query, which adds overhead captured in the per-query time
- Balance between effectiveness and efficiency depends on adaptation-pair count; 3-5 pairs yield the best balance per the paper

## Evidence

- [methods] Average processing time per query ranges from 1.75s (1-pair) to 2.60s (5-pair), lower than CoT's 4.32s: [evidence span withheld — anchor status: not_found]
- [methods] Optimal learning-rate range and pair balance: [evidence span withheld — anchor status: not_found]
