---
name: retrieval-augmented-prompting-baseline-configuration
description: Use when evaluating a new RAG method (e.g., test-time adaptation) and
  you need identical queries, retrieval content, and backbone LLMs for naive-RAG,
  CoT, and ICL baselines to make comparisons fair.
when_to_use_negative:
- The target method is being trained or fine-tuned rather than evaluated against prompting
  baselines — use a training pipeline instead
- The benchmark requires the official CRAG scoring (correct/missing/incorrect with
  -1 penalty) rather than LLM-judged semantic equivalence
- Only one condition is being run and no baseline comparison is intended; the shared-retrieval
  harness adds setup overhead without benefit
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: parallel_evaluation.py
  role: Runs naive-RAG, CoT, ICL, and TTA conditions in parallel over a dataset and
    saves predictions and summaries
  repo: https://github.com/sunxin000/TTARAG
- name: MiniLM-v6
  role: Reranks all sentences in retrieved HTML pages and selects the top 20 sentences
    as shared retrieval content for baselines
- name: Brave Search API
  role: Source of up to 50 full HTML pages per query for retrieval-content construction
- name: Pyserini
  role: Standard BM25 retriever fetching the top 5 relevant passages for corpus-based
    datasets (e.g., PubMedQA)
- name: Qwen2.5-72B-Instruct
  role: LLM-as-a-judge scoring semantic equivalence between baseline predictions and
    reference answers
- name: Llama-3.1-8B-Instruct
  role: Backbone generation model run identically across naive-RAG, CoT, and ICL conditions
- name: ms-marco-MiniLM-L-6-v2
  role: Cross-encoder used for sentence selection in the RAG pipeline invoked from
    parallel_evaluation.py
  repo: https://github.com/sunxin000/TTARAG
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1109/icassp55912.2026.11464478
    title: Predict the Retrieval! Test Time Adaptation for Retrieval Augmented Generation
  source_content_hash: sha256:6e1a310af64180f2d324ac4f1873c6c6b183353a7052b66846bdf72a04780900
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sunxin000/TTARAG
  tools:
  - Brave Search API
  - Llama-3.1-8B-Instruct
  - MiniLM-v6
  - NVIDIA A100 GPU
  - Pyserini
  - Qwen2.5-72B-Instruct
  - TTARAG
  - ms-marco-MiniLM-L-6-v2
  - parallel_evaluation.py
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# retrieval-augmented-prompting-baseline-configuration

## Summary

Configures and runs retrieval-augmented and prompting baselines (naive-RAG, Chain-of-Thought, In-Context Learning) so that test-time adaptation methods like TTARAG can be compared on equal footing against standard RAG systems.

## When to use

Use when evaluating a new RAG method (e.g., test-time adaptation) and you need identical queries, retrieval content, and backbone LLMs for naive-RAG, CoT, and ICL baselines to make comparisons fair.

## When NOT to use

- The target method is being trained or fine-tuned rather than evaluated against prompting baselines — use a training pipeline instead
- The benchmark requires the official CRAG scoring (correct/missing/incorrect with -1 penalty) rather than LLM-judged semantic equivalence
- Only one condition is being run and no baseline comparison is intended; the shared-retrieval harness adds setup overhead without benefit

## Inputs

- QA benchmark queries (e.g., CRAG's 2,706 question-answer pairs, or PubMedQA pqa_labeled train split)
- Retrieved HTML pages (up to 50 per query from the Brave Search API) or BM25 top-5 passages from Pyserini
- Backbone LLM weights (e.g., models/Llama-3.1-8B-Instruct)
- Prompt templates from the prompts/ directory (CoT and ICL variants)
- models/config.py RAGConfig and TTAConfig settings (model_name, embedding_model_name)

## Outputs

- Per-query model predictions saved under predictions/
- Per-dataset accuracy summaries under results/<dataset>/
- Accuracy (%) tables aggregated per domain and per model-condition pair (e.g., crag_accuracy_table.csv)

## How to apply

Retrieve content once per query and share it across all baseline conditions: fetch up to 50 full HTML pages via the Brave Search API (or BM25 top-5 with Pyserini for corpus-based datasets), rerank all sentences with MiniLM-v6, and select the top 20 sentences as the retrieval context for the naive-RAG baseline. Run each backbone (e.g., Llama-3.1-8B-Instruct) through parallel_evaluation.py without --do_tta for naive-RAG, and with the CoT and ICL prompt templates from the prompts/ directory on the same queries. Keep model config, batch size, tensor_parallel_size, and gpu_memory_utilization identical across conditions (e.g., batch_size 1, tensor_parallel_size 1, gpu_memory_utilization 0.85) so differences reflect the method, not the harness. Reuse cached predictions via --predictions_path instead of regenerating when only scoring changes. Score all conditions with the same LLM-as-a-judge (Qwen2.5-72B-Instruct) for semantic equivalence, then aggregate accuracy per domain.

## Related tools

- **parallel_evaluation.py** (Runs naive-RAG, CoT, ICL, and TTA conditions in parallel over a dataset and saves predictions and summaries) — https://github.com/sunxin000/TTARAG
- **MiniLM-v6** (Reranks all sentences in retrieved HTML pages and selects the top 20 sentences as shared retrieval content for baselines)
- **Brave Search API** (Source of up to 50 full HTML pages per query for retrieval-content construction)
- **Pyserini** (Standard BM25 retriever fetching the top 5 relevant passages for corpus-based datasets (e.g., PubMedQA))
- **Qwen2.5-72B-Instruct** (LLM-as-a-judge scoring semantic equivalence between baseline predictions and reference answers)
- **Llama-3.1-8B-Instruct** (Backbone generation model run identically across naive-RAG, CoT, and ICL conditions)
- **ms-marco-MiniLM-L-6-v2** (Cross-encoder used for sentence selection in the RAG pipeline invoked from parallel_evaluation.py) — https://github.com/sunxin000/TTARAG

## Examples

```
python parallel_evaluation.py --do_generate --dataset pubmedqa --split train --model_name models/Llama-3.1-8B-Instruct --batch_size 1 --tensor_parallel_size 1 --gpu_memory_utilization 0.85
```

## Evaluation signals

- All baseline conditions consume the identical top-20 reranked sentence set per query — verify no condition-specific retrieval drift
- Predictions directory contains one file per query per condition, and results/<dataset>/ summaries cover every model-condition pair
- LLM-judger agreement with human annotators is at the reported ~98% level on a 100-example random sample
- Naive-RAG accuracy is reproducibly lower than TTA results (TTARAG best in 19 of 24 settings) when the same harness and queries are used
- Skipping --do_generate with --predictions_path reproduces the same aggregated scores as a fresh generation run

## Limitations

- Processing 2,706 queries requires a single NVIDIA A100 GPU; smaller GPUs may need batch/tensor-parallel adjustments
- The official CRAG auto-evaluation uses rule-based matching plus LLM assessment with a -1 incorrect penalty, which differs from the paper's binary semantic-equivalence judging — scores are not directly interchangeable
- Model weights must be placed at the configured local paths (models/Llama-3.1-8B-Instruct, models/allMiniLM) or overridden via --model_name / config; missing weights cause startup failure
- TTA performance is insensitive to adaptation pairs and learning rate in the 1e-6 to 1e-5 range, but baseline comparisons still depend on fixed retrieval content

## Evidence

- [methods] LLM-as-a-judge agreement: [evidence span withheld — anchor status: not_found]
- [intro] Baseline comparison claim: [evidence span withheld — anchor status: not_found]
