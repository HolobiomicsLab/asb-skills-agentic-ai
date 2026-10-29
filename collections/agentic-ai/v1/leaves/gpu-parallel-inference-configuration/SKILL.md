---
name: gpu-parallel-inference-configuration
description: Use when running RAG or test-time-adaptation generation over a question-answer
  benchmark (e.g. CRAG's 2,706 queries, PubMedQA, BioASQ) with Llama-3.1-8B-Instruct
  on one NVIDIA A100 GPU and you need parallelized, reproducible evaluation with saved
  predictions.
when_to_use_negative:
- You only need single-query debugging of the RAG or TTA pipeline — use local_evaluation.py
  instead of the parallel runner
- Your task is evaluation/judging of already-generated answers only (LLM-as-a-judge
  with Qwen2.5-72B-Instruct) rather than GPU inference configuration
- No local model weights are available and --model_name cannot point to valid Llama-3.1-8B-Instruct
  or encoder directories
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Llama-3.1-8B-Instruct
  role: Generation model whose parallel inference and TTA adaptation is configured
    via --model_name
  repo: https://github.com/sunxin000/TTARAG
- name: TTARAG
  role: Framework providing parallel_evaluation.py, models/user_config.py RAGModel
    selection, and TTAConfig knobs
  repo: https://github.com/sunxin000/TTARAG
- name: NVIDIA A100 GPU
  role: Single-GPU target hardware; 2,706 CRAG queries processed on one A100
- name: ms-marco-MiniLM-L-6-v2
  role: Cross-encoder supplied via --cross_encoder_name for TTA sentence selection
    during adaptation runs
  repo: https://github.com/sunxin000/TTARAG
- name: Qwen2.5-72B-Instruct
  role: LLM-as-a-judge used downstream to score the generated answers for semantic
    equivalence
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1109/icassp55912.2026.11464478
    title: Predict the Retrieval! Test Time Adaptation for Retrieval Augmented Generation
  source_content_hash: sha256:7f166b5c9df7eddab1473f8e57f6fc069d2807a3eb9e054a4b0c7228e16b0fab
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5294117647058824
n_claims: 17
---

# gpu-parallel-inference-configuration

## Summary

Configure and run large-model (Llama-3.1-8B-Instruct) answer generation and test-time adaptation in parallel on a single NVIDIA A100 GPU via the TTARAG repository's parallel_evaluation.py, using batch_size, tensor_parallel_size, gpu_memory_utilization, --do_tta, --learning_rate and --max_adapt_pairs flags, saving predictions/ and results/<dataset>/ artifacts.

## When to use

Use when running RAG or test-time-adaptation generation over a question-answer benchmark (e.g. CRAG's 2,706 queries, PubMedQA, BioASQ) with Llama-3.1-8B-Instruct on one NVIDIA A100 GPU and you need parallelized, reproducible evaluation with saved predictions.

## When NOT to use

- You only need single-query debugging of the RAG or TTA pipeline — use local_evaluation.py instead of the parallel runner
- Your task is evaluation/judging of already-generated answers only (LLM-as-a-judge with Qwen2.5-72B-Instruct) rather than GPU inference configuration
- No local model weights are available and --model_name cannot point to valid Llama-3.1-8B-Instruct or encoder directories

## Inputs

- Benchmark question-answer pairs with retrieved passages (CRAG 2,706 QA pairs; PubMedQA via dataset/PubMedQAAdapter with load_dataset('qiaojin/PubMedQA','pqa_labeled'); BioASQ)
- Local model weights for Llama-3.1-8B-Instruct (models/Llama-3.1-8B-Instruct) and sentence encoders (allMiniLM, ms-marco-MiniLM-L-6-v2)
- Optional existing prediction files passed via --predictions_path

## Outputs

- Generated/TTA-adapted answer predictions saved under predictions/
- Aggregate evaluation summaries saved under results/<dataset>/
- Per-query processing-time statistics (e.g. 1.75s-2.60s per query for 1-5 adaptation pairs)

## How to apply

Instantiate the model class chosen in models/user_config.py (default RAGModel for TTARAG runs) and launch parallel_evaluation.py from the TTARAG/ folder with --do_generate, --dataset (e.g. pubmedqa or CRAG queries), and --model_name models/Llama-3.1-8B-Instruct. Pin GPU configuration with --batch_size 1, --tensor_parallel_size 1 and --gpu_memory_utilization 0.85 as shown in the README. For adaptation runs add --do_tta --learning_rate 1e-5 --max_adapt_pairs 3 and --cross_encoder_name models/ms-marco-MiniLM-L-6-v2; the paper's sweep covers learning rates 1e-6 to 1e-5 and 1-5 adaptation pairs, resetting model parameters to their pre-trained state before each adaptation iteration. Reuse existing generations by omitting --do_generate and passing --predictions_path. Predictions are written under predictions/ and aggregate summaries under results/<dataset>/; throughput reference is 2,706 CRAG queries processed on one NVIDIA A100 GPU with per-query time of 1.75s (1-pair) to 2.60s (5-pair).

## Related tools

- **Llama-3.1-8B-Instruct** (Generation model whose parallel inference and TTA adaptation is configured via --model_name) — https://github.com/sunxin000/TTARAG
- **TTARAG** (Framework providing parallel_evaluation.py, models/user_config.py RAGModel selection, and TTAConfig knobs) — https://github.com/sunxin000/TTARAG
- **NVIDIA A100 GPU** (Single-GPU target hardware; 2,706 CRAG queries processed on one A100)
- **ms-marco-MiniLM-L-6-v2** (Cross-encoder supplied via --cross_encoder_name for TTA sentence selection during adaptation runs) — https://github.com/sunxin000/TTARAG
- **Qwen2.5-72B-Instruct** (LLM-as-a-judge used downstream to score the generated answers for semantic equivalence)

## Examples

```
python parallel_evaluation.py --do_generate --dataset pubmedqa --model_name models/Llama-3.1-8B-Instruct --batch_size 1 --tensor_parallel_size 1 --gpu_memory_utilization 0.85 --do_tta --learning_rate 1e-5 --max_adapt_pairs 3 --cross_encoder_name models/ms-marco-MiniLM-L-6-v2
```

## Evaluation signals

- Predictions directory populated per dataset and summaries present under results/<dataset>/ after the run
- GPU configuration flags honored: run completes with --tensor_parallel_size 1 and --gpu_memory_utilization 0.85 on one A100 without OOM
- TTA sweep reproduces near-flat accuracy curves across learning rates 1e-6 to 1e-5 and 1-5 adaptation pairs, with TTARAG above naive-RAG in almost all settings
- Per-query wall-clock time falls in the ~1.75s-2.60s range (below CoT's 4.32s) for 1-5 adaptation pairs
- Re-running with --predictions_path (omitting --do_generate) reproduces identical summaries from saved predictions

## Limitations

- Requires local model/encoder weights in the expected paths (models/Llama-3.1-8B-Instruct, models/allMiniLM) or CLI overrides
- Configuration is demonstrated for a single A100 GPU (tensor_parallel_size 1); multi-GPU setups are not covered in the README
- No changelog found in the associated TTARAG repository, so version drift in flag behavior is possible

## Evidence
