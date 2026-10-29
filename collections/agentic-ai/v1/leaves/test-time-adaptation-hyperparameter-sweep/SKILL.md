---
name: test-time-adaptation-hyperparameter-sweep
description: Use when you need to determine optimal learning-rate and adaptation-pair
  settings for test-time adaptation of a RAG model (e.g. Llama-3.1-8B-Instruct with
  TTARAG) on a benchmark such as CRAG or PubMedQA.
when_to_use_negative:
- When the goal is a single best-configuration run rather than a hyperparameter sweep—use
  the known optimum (learning rate ~1e-5, 3 adaptation pairs) directly instead of
  re-tuning.
- When the downstream evaluation uses rule-based matching alone rather than the Qwen2.5-72B-Instruct
  semantic-equivalence judge, since sweep conclusions were validated under the paper's
  LLM-judge protocol.
- When the task targets domains far outside the paper's evaluation (CRAG, PubMedQA,
  BioASQ), as the flat-curve insensitivity finding may not transfer.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3489
tools:
- name: Llama-3.1-8B-Instruct
  role: Base generation model whose parameters are adapted during the test-time adaptation
    sweep
  repo: https://github.com/sunxin000/TTARAG
- name: TTARAG
  role: Test-time adaptation method being tuned; implements prefix-suffix self-supervised
    adaptation via models/tta_module.py
  repo: https://github.com/sunxin000/TTARAG
- name: Qwen2.5-72B-Instruct
  role: LLM-as-a-judge scoring semantic equivalence between predictions and reference
    answers for each configuration
- name: ms-marco-MiniLM-L-6-v2
  role: Cross-encoder used to select sentences for adaptation context during TTA runs
  repo: https://github.com/sunxin000/TTARAG
- name: NVIDIA A100 GPU
  role: GPU used to run the parallel sweep over 2,706 CRAG queries
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1109/icassp55912.2026.11464478
    title: Predict the Retrieval! Test Time Adaptation for Retrieval Augmented Generation
  source_content_hash: sha256:9d4f6959cc51cf25475861f1566c79773428c2f797f1a6420e1a78ac7c179c17
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3489
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# test-time-adaptation-hyperparameter-sweep

## Summary

A procedure for sweeping test-time adaptation (TTARAG) hyperparameters—learning rate (1e-6 to 1e-5) and number of prefix-suffix adaptation pairs (1–5)—over the CRAG benchmark to locate the configuration that maximizes LLM-judged question-answering accuracy.

## When to use

Use when you need to determine optimal learning-rate and adaptation-pair settings for test-time adaptation of a RAG model (e.g. Llama-3.1-8B-Instruct with TTARAG) on a benchmark such as CRAG or PubMedQA.

## When NOT to use

- When the goal is a single best-configuration run rather than a hyperparameter sweep—use the known optimum (learning rate ~1e-5, 3 adaptation pairs) directly instead of re-tuning.
- When the downstream evaluation uses rule-based matching alone rather than the Qwen2.5-72B-Instruct semantic-equivalence judge, since sweep conclusions were validated under the paper's LLM-judge protocol.
- When the task targets domains far outside the paper's evaluation (CRAG, PubMedQA, BioASQ), as the flat-curve insensitivity finding may not transfer.

## Inputs

- CRAG benchmark queries (2,706 question-answer pairs) with retrieved passages from the TTARAG repository
- Llama-3.1-8B-Instruct model weights (local path, e.g. models/Llama-3.1-8B-Instruct)
- Cross-encoder weights ms-marco-MiniLM-L-6-v2 for sentence selection
- Gold reference answers for LLM-as-a-judge scoring

## Outputs

- Sweep table of LLM-judged accuracy per (learning rate, number of adaptation pairs) configuration
- Prediction files under predictions/
- Result summaries under results/<dataset>/
- Accuracy-versus-learning-rate and accuracy-versus-adaptation-pairs plots (Fig. 2 / Fig. 3 style)

## How to apply

Load the benchmark queries with retrieved passages and initialize Llama-3.1-8B-Instruct via the RAGModel class in models/user_config.py. For each learning rate from 1e-6 to 1e-5 and each adaptation-pair count from 1 to 5, run parallel_evaluation.py with --do_tta, --learning_rate and --max_adapt_pairs, resetting model parameters to their pre-trained state before each adaptation iteration. In each run, adapt on top-ranked prefix-suffix pairs using AdamW with two-step gradient accumulation and gradient clipping, then generate answers with the adapted parameters. Score predictions with the Qwen2.5-72B-Instruct LLM-as-a-judge semantic-equivalence protocol from CRAG, aggregate accuracy per (learning rate, pairs) configuration into a sweep table saved under results/<dataset>/, and plot accuracy-versus-hyperparameter curves. The expected outcome is peak accuracy within 1e-6 to 1e-5 learning rate with 3-5 adaptation pairs and near-flat curves indicating insensitivity to both hyperparameters.

## Related tools

- **Llama-3.1-8B-Instruct** (Base generation model whose parameters are adapted during the test-time adaptation sweep) — https://github.com/sunxin000/TTARAG
- **TTARAG** (Test-time adaptation method being tuned; implements prefix-suffix self-supervised adaptation via models/tta_module.py) — https://github.com/sunxin000/TTARAG
- **Qwen2.5-72B-Instruct** (LLM-as-a-judge scoring semantic equivalence between predictions and reference answers for each configuration)
- **ms-marco-MiniLM-L-6-v2** (Cross-encoder used to select sentences for adaptation context during TTA runs) — https://github.com/sunxin000/TTARAG
- **NVIDIA A100 GPU** (GPU used to run the parallel sweep over 2,706 CRAG queries)

## Examples

```
python parallel_evaluation.py --do_generate --dataset pubmedqa --model_name models/Llama-3.1-8B-Instruct --do_tta --learning_rate 1e-5 --max_adapt_pairs 3 --cross_encoder_name models/ms-marco-MiniLM-L-6-v2
```

## Evaluation signals

- Peak LLM-judged accuracy falls within the 1e-6 to 1e-5 learning-rate range with 3-5 adaptation pairs.
- Accuracy-versus-hyperparameter curves are near-flat, confirming the paper's finding that TTARAG performance is not sensitive to learning rate or number of adaptation pairs.
- TTARAG accuracy exceeds the naive-rag baseline in almost all sweep settings.
- Predictions and summaries are written to the expected locations (predictions/ and results/<dataset>/) for every configuration.
- Average per-query processing time stays in the observed 1.75s (1-pair) to 2.60s (5-pair) band.

## Limitations

- The LLM-judge protocol was validated at 98% agreement with human annotators on only a random sample of 100 examples, so sweep scores carry judge-induced noise.
- TTARAG's insensitivity to learning rate and adaptation pairs was established on the paper's benchmarks; domain-specific sweeps may behave differently.
- Each configuration requires a full reset to pre-trained parameters before adaptation, so the sweep is compute-intensive (2,706 queries per configuration on one NVIDIA A100 GPU).

## Evidence

- [methods] learning rates between 1e-6 to 1e-5 provide optimal performance, with 3-5 adaptation pairs yielding the best balance between effectiveness and efficiency: [evidence span withheld — anchor status: not_found]
- [methods] Insensitivity finding: [evidence span withheld — anchor status: not_found]
