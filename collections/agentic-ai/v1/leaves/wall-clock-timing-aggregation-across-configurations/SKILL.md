---
name: wall-clock-timing-aggregation-across-configurations
description: Use when comparing inference-time cost of a test-time adaptation method
  against baseline prompting methods (CoT, ICL, naive-RAG) on the same benchmark and
  GPU, and you need a per-configuration timing table.
when_to_use_negative:
- You are measuring answer quality rather than efficiency — use the LLM-as-a-judge
  (Qwen2.5-72B-Instruct) accuracy evaluation instead of wall-clock timing.
- Hardware or batching differs between configurations (e.g. multiple GPUs, batch_size
  > 1) — timing comparisons across mismatched hardware/batch settings are invalid.
- Only a single configuration is being run with no baseline to compare against — a
  timing table requires at least one reference method such as naive-RAG or CoT.
edam_operation: http://edamontology.org/operation_2426
tools:
- name: TTARAG
  role: Test-time adaptation RAG pipeline whose 1–5 adaptation-pair configurations
    are timed and compared
  repo: https://github.com/sunxin000/TTARAG
- name: Llama-3.1-8B-Instruct
  role: Generation model used for all timed configurations (naive-RAG, CoT baseline,
    TTARAG)
- name: NVIDIA A100 GPU
  role: Fixed single-GPU hardware on which all 2,706 queries are processed for each
    configuration's wall-clock timing
- name: MiniLM-v6
  role: Sentence reranker producing the fixed top-20-sentence retrieval content used
    across all timed runs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1109/icassp55912.2026.11464478
    title: Predict the Retrieval! Test Time Adaptation for Retrieval Augmented Generation
  source_content_hash: sha256:bc384100353c82269a4b4461bd0f7353a937901bd0b783ae74fb2464b9885273
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sunxin000/TTARAG
  tools:
  - Llama-3.1-8B-Instruct
  - MiniLM-v6
  - NVIDIA A100 GPU
  - TTARAG
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# wall-clock-timing-aggregation-across-configurations

## Summary

A benchmarking skill for measuring and aggregating total and per-query wall-clock inference time across multiple method configurations (TTARAG 1–5 adaptation pairs, Chain-of-Thought, naive-RAG) on fixed hardware, to quantify computational efficiency of a test-time adaptation RAG pipeline.

## When to use

Use when comparing inference-time cost of a test-time adaptation method against baseline prompting methods (CoT, ICL, naive-RAG) on the same benchmark and GPU, and you need a per-configuration timing table.

## When NOT to use

- You are measuring answer quality rather than efficiency — use the LLM-as-a-judge (Qwen2.5-72B-Instruct) accuracy evaluation instead of wall-clock timing.
- Hardware or batching differs between configurations (e.g. multiple GPUs, batch_size > 1) — timing comparisons across mismatched hardware/batch settings are invalid.
- Only a single configuration is being run with no baseline to compare against — a timing table requires at least one reference method such as naive-RAG or CoT.

## Inputs

- CRAG benchmark: 2,706 question-answer pairs with pre-retrieved content (Brave Search HTML pages reranked by MiniLM-v6 to top 20 sentences)
- Llama-3.1-8B-Instruct generation model weights (models/Llama-3.1-8B-Instruct)
- TTARAG repository (parallel_evaluation.py, rag_llama_baseline.py, cot_baseline.py)
- One NVIDIA A100 GPU with fixed batch_size 1 and tensor_parallel_size 1

## Outputs

- Table of total wall-clock inference time (seconds) per configuration
- Average per-query processing time (seconds/query) for 1-pair through 5-pair TTARAG, CoT, and naive-RAG
- Efficiency comparison showing TTARAG overhead relative to CoT's 4.32s per query

## How to apply

Run each configuration serially on a single NVIDIA A100 GPU over the full 2,706-query CRAG benchmark with pre-retrieved content (Brave Search HTML pages reranked by MiniLM-v6 to the top 20 sentences), recording total wall-clock time per run. Execute naive-RAG generation with Llama-3.1-8B-Instruct via parallel_evaluation.py (no adaptation) as the fastest reference point, then the Chain-of-Thought baseline, then TTARAG for each adaptation-pair setting from 1 to 5 pairs using a learning rate in the optimal 1e-6 to 1e-5 range. For every TTARAG run, reset model parameters to the pre-trained state per query so timing reflects true per-query adaptation cost. Divide each total time by 2,706 to obtain average seconds per query, and assemble a table with total (s) and average per-query (s) columns for 1-pair through 5-pair TTARAG, CoT, and naive-RAG. The rationale is that test-time adaptation adds overhead per query, so timing must isolate the adaptation loop (suffix-prediction training steps plus parameter reset) from baseline generation cost to show the efficiency trade-off.

## Related tools

- **TTARAG** (Test-time adaptation RAG pipeline whose 1–5 adaptation-pair configurations are timed and compared) — https://github.com/sunxin000/TTARAG
- **Llama-3.1-8B-Instruct** (Generation model used for all timed configurations (naive-RAG, CoT baseline, TTARAG))
- **NVIDIA A100 GPU** (Fixed single-GPU hardware on which all 2,706 queries are processed for each configuration's wall-clock timing)
- **MiniLM-v6** (Sentence reranker producing the fixed top-20-sentence retrieval content used across all timed runs)

## Examples

```
python parallel_evaluation.py --do_generate --dataset crag --model_name models/Llama-3.1-8B-Instruct --do_tta --learning_rate 1e-5 --max_adapt_pairs 3 --cross_encoder_name models/ms-marco-MiniLM-L-6-v2
```

## Evaluation signals

- Average per-query time for 1-pair TTARAG falls near 1.75s and for 5-pair near 2.60s (per-query time should grow monotonically with the number of adaptation pairs).
- All TTARAG per-query times (1.75s–2.60s) are substantially below Chain-of-Thought's 4.32s per query.
- Naive-RAG (no adaptation) is the fastest configuration in the aggregated table.
- Each configuration's total time divided by 2,706 queries reproduces the reported average within measurement noise.
- All runs use identical hardware (one NVIDIA A100), identical batch_size 1 / tensor_parallel_size 1 settings, and the same pre-retrieved CRAG content.

## Limitations

- Timing is hardware- and software-stack dependent: reported values (1.75s–2.60s TTARAG, 4.32s CoT) were measured on a single NVIDIA A100 and will not transfer directly to other GPUs.
- TTARAG timing depends on the number of adaptation pairs; 1-pair is fastest but 3–5 pairs yield the best effectiveness-efficiency balance, so a single timing number misrepresents the trade-off.
- Model parameters must be reset to the pre-trained state per query for fair timing; forgetting the reset inflates or deflates per-query adaptation cost.
- TTARAG performance is reportedly not sensitive to the number of adaptation pairs and learning rate, but efficiency cost does scale with pairs — quality insensitivity should not be read as cost insensitivity.

## Evidence

- [methods] Learning rates between 1e-6 to 1e-5 provide optimal performance, with 3-5 adaptation pairs yielding the best balance between effectiveness and efficiency: [evidence span withheld — anchor status: not_found]
