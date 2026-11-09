---
name: learning-rate-sensitivity-visualization
description: 'Use when tuning a test-time adaptation loop for RAG: you need to choose
  a learning rate and adaptation-pair budget and demonstrate via sweep curves that
  performance is robust across settings.'
when_to_use_negative:
- Your system does not perform test-time adaptation (e.g. plain naive-RAG, Chain-of-Thought,
  or In-Context Learning baselines) — there are no adaptation hyperparameters to sweep.
- You lack the compute to re-adapt model parameters per configuration (e.g. no A100-class
  GPU); each setting requires resetting and re-running adaptation per query.
- Your goal is measuring sensitivity of retrieval quality (retriever/cross-encoder
  choice) rather than TTA adaptation hyperparameters.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Llama-3.1-8B-Instruct
  role: Base generation model whose parameters are adapted during the test-time sweep
  repo: https://github.com/sunxin000/TTARAG
- name: TTARAG
  role: Test-time adaptation framework providing the RAGModel class, TTAConfig, and
    parallel_evaluation.py entry point for the sweep
  repo: https://github.com/sunxin000/TTARAG
- name: ms-marco-MiniLM-L-6-v2
  role: Cross-encoder used with --cross_encoder_name for sentence selection during
    TTA runs
  repo: https://github.com/sunxin000/TTARAG
- name: Qwen2.5-72B-Instruct
  role: LLM-as-a-judge for semantic-equivalence scoring of generated answers against
    references
- name: NVIDIA A100 GPU
  role: Hardware for running the 2,706-query sweep across all learning-rate and pair-count
    configurations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1109/icassp55912.2026.11464478
    title: Predict the Retrieval! Test Time Adaptation for Retrieval Augmented Generation
  source_content_hash: sha256:9442174735d09d1d0917dccf90ddf2062009458e9739302ec4e31ec16284b41b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sunxin000/TTARAG
  tools:
  - Llama-3.1-8B-Instruct
  - MiniLM-v6
  - NVIDIA A100 GPU
  - Qwen2.5-72B-Instruct
  - TTARAG
  - ms-marco-MiniLM-L-6-v2
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# learning-rate-sensitivity-visualization

## Summary

Sweep TTARAG's test-time adaptation hyperparameters — learning rate (1e-6 to 1e-5) and number of prefix-suffix adaptation pairs (1–5) — on the CRAG benchmark with Llama-3.1-8B-Instruct, then plot accuracy versus each hyperparameter (Fig. 2 / Fig. 3 style) to visualize insensitivity and locate the optimal setting.

## When to use

Use when tuning a test-time adaptation loop for RAG: you need to choose a learning rate and adaptation-pair budget and demonstrate via sweep curves that performance is robust across settings.

## When NOT to use

- Your system does not perform test-time adaptation (e.g. plain naive-RAG, Chain-of-Thought, or In-Context Learning baselines) — there are no adaptation hyperparameters to sweep.
- You lack the compute to re-adapt model parameters per configuration (e.g. no A100-class GPU); each setting requires resetting and re-running adaptation per query.
- Your goal is measuring sensitivity of retrieval quality (retriever/cross-encoder choice) rather than TTA adaptation hyperparameters.

## Inputs

- CRAG benchmark queries (2,706 question-answer pairs) with retrieved passages from the TTARAG repository
- Llama-3.1-8B-Instruct model weights (models/Llama-3.1-8B-Instruct)
- Prefix-suffix pairs produced by two-tier passage splitting, after length filtering of short passages
- Gold reference answers for LLM-as-a-judge scoring (Qwen2.5-72B-Instruct)

## Outputs

- Sweep table of accuracy per (learning rate, number of adaptation pairs) configuration
- Accuracy-versus-learning-rate plot (Fig. 2 style) and accuracy-versus-adaptation-pairs plot (Fig. 3 style)
- Per-config generated answers saved under predictions/ and summaries under results/<dataset>/

## How to apply

Initialize the Llama-3.1-8B-Instruct model via the RAGModel class in models/user_config.py and load CRAG queries with retrieved passages from the TTARAG repository. For each learning rate in the 1e-6 to 1e-5 range and each adaptation-pair count from 1 to 5, run parallel_evaluation.py with --do_tta, --learning_rate and --max_adapt_pairs flags, resetting model parameters to their pre-trained state before each adaptation iteration. Adaptation uses AdamW updates on the self-supervised suffix-prediction task with two-step gradient accumulation and gradient clipping; answers are scored by the Qwen2.5-72B-Instruct LLM-as-a-judge semantic-equivalence protocol. Aggregate accuracy per (learning rate, pairs) configuration into a sweep table (predictions under predictions/, summaries under results/<dataset>/), then plot accuracy versus learning rate and versus number of adaptation pairs. The expected outcome is a near-flat curve pair — the paper reports learning rates between 1e-6 to 1e-5 with 3-5 adaptation pairs yield the best balance between effectiveness and efficiency, and TTARAG stays above the naive-rag baseline in almost all settings.

## Related tools

- **Llama-3.1-8B-Instruct** (Base generation model whose parameters are adapted during the test-time sweep) — https://github.com/sunxin000/TTARAG
- **TTARAG** (Test-time adaptation framework providing the RAGModel class, TTAConfig, and parallel_evaluation.py entry point for the sweep) — https://github.com/sunxin000/TTARAG
- **ms-marco-MiniLM-L-6-v2** (Cross-encoder used with --cross_encoder_name for sentence selection during TTA runs) — https://github.com/sunxin000/TTARAG
- **Qwen2.5-72B-Instruct** (LLM-as-a-judge for semantic-equivalence scoring of generated answers against references)
- **NVIDIA A100 GPU** (Hardware for running the 2,706-query sweep across all learning-rate and pair-count configurations)

## Examples

```
python parallel_evaluation.py --do_generate --dataset pubmedqa --model_name models/Llama-3.1-8B-Instruct --do_tta --learning_rate 1e-5 --max_adapt_pairs 3
```

## Evaluation signals

- Peak accuracy in the sweep table falls within the 1e-6 to 1e-5 learning-rate range with 3-5 adaptation pairs
- Accuracy-versus-learning-rate and accuracy-versus-pairs curves are near-flat, reflecting the paper's claim that TTARAG performance is not sensitive to either hyperparameter
- TTARAG accuracy sits above the naive-rag baseline in almost all swept configurations
- Model parameters are verifiably reset to their pre-trained state before each adaptation iteration (no cross-configuration leakage)
- Predictions and per-dataset summaries exist under predictions/ and results/<dataset>/ for every swept configuration

## Limitations

- Performance is reported as insensitive to learning rate and pair count, so the sweep mainly confirms robustness rather than finding a sharp optimum
- Processing time per query grows with pair count (1.75s for 1-pair to 2.60s for 5-pair), so the 3-5 pair recommendation trades efficiency against effectiveness
- The hyperparameter analysis was performed on CRAG (2,706 QA pairs) with Llama-3.1-8B-Instruct; transferring the sweep to other domains/models may require re-running it

## Evidence

- [methods] Learning rates between 1e-6 to 1e-5 provide optimal performance, with 3-5 adaptation pairs yielding the best balance between effectiveness and efficiency: [evidence span withheld — anchor status: not_found]
- [methods] TTARAG performance insensitivity to swept hyperparameters: [evidence span withheld — anchor status: not_found]
