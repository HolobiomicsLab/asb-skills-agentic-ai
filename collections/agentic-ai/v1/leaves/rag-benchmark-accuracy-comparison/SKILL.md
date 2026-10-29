---
name: rag-benchmark-accuracy-comparison
description: Use when you must tabulate and compare question-answering accuracy across
  RAG configurations (method vs ablation, or hyperparameter sweep cells) on CRAG-style
  QA pairs with retrieved passages.
when_to_use_negative:
- When the goal is exact-match scoring against gold strings — the protocol here is
  semantic equivalence via LLM-as-a-judge, not exact match
- When the benchmark lacks retrieved passages or retrieval content, since the comparison
  depends on the top-20 retrieved sentences per query
- When you need a fully ablation-free single-method evaluation with no comparison
  configuration to tabulate against
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3365
tools:
- name: TTARAG
  role: Test-time adaptation method whose accuracy is compared against the no-segmentation
    ablation on CRAG
  repo: https://github.com/sunxin000/TTARAG
- name: Llama-3.1-8B-Instruct
  role: Generation model for producing answers under each configuration
- name: Qwen2.5-72B-Instruct
  role: LLM-as-a-judge scoring semantic equivalence between predictions and reference
    answers
- name: MiniLM-v6
  role: Reranks sentences in retrieved HTML and selects the top-20 retrieval content
    per query
- name: NVIDIA A100 GPU
  role: Single-GPU hardware used to process the 2,706 CRAG queries
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1109/icassp55912.2026.11464478
    title: Predict the Retrieval! Test Time Adaptation for Retrieval Augmented Generation
  source_content_hash: sha256:8010b2c2d2141267fd1785548a07f48a5fd146455d17847053d7bd8bd42978e2
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3365
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# rag-benchmark-accuracy-comparison

## Summary

Compares accuracy of RAG variants (e.g., full TTARAG vs a no-segmentation ablation, or hyperparameter settings) on the CRAG benchmark (2,706 QA pairs) by generating answers with adapted/adapted-per-configuration parameters and scoring them with the LLM-as-a-judge semantic-equivalence protocol (Qwen2.5-72B-Instruct) rather than exact match.

## When to use

Use when you must tabulate and compare question-answering accuracy across RAG configurations (method vs ablation, or hyperparameter sweep cells) on CRAG-style QA pairs with retrieved passages.

## When NOT to use

- When the goal is exact-match scoring against gold strings — the protocol here is semantic equivalence via LLM-as-a-judge, not exact match
- When the benchmark lacks retrieved passages or retrieval content, since the comparison depends on the top-20 retrieved sentences per query
- When you need a fully ablation-free single-method evaluation with no comparison configuration to tabulate against

## Inputs

- CRAG benchmark question-answer pairs (2,706 pairs) with top-20 retrieved sentences per query
- PubMedQA (qiaojin/PubMedQA, pqa_labeled, split=train) as an alternative benchmark when run via --dataset pubmedqa
- Retrieval content produced by BM25 top-5 passage retrieval and MiniLM-v6 top-20 sentence reranking
- Per-configuration predictions (or --predictions_path to reuse existing ones)

## Outputs

- predictions/ files with generated answers per configuration
- results/<dataset>/ summaries
- Comparison table (e.g., results_table.tsv) of CRAG overall accuracy per configuration/model

## How to apply

Load the CRAG benchmark queries with their top-20 retrieved sentences (BM25 retrieval of top-5 passages followed by MiniLM-v6 sentence reranking) and run each configuration through parallel_evaluation.py with --do_generate, adding --do_tta with --learning_rate 1e-5 and --max_adapt_pairs 3 for full TTARAG, or the ablation variant that performs next-token prediction on whole passages without prefix-suffix splitting. Reset model parameters to their pre-trained state before each adaptation iteration so configurations are comparable, and generate answers with the adapted parameters. Score all generated answers against gold references using the LLM-as-a-judge (Qwen2.5-72B-Instruct) semantic-equivalence protocol, which the paper reports shows 98% agreement with human annotators on 100 sampled examples. Tabulate accuracy per configuration (e.g., TTARAG at 31.9/30.5/25.7 vs wo-seg at 30.8/30.1/25.0 across Llama-3.1-8B-Instruct, Llama-2-7b-chat, and ChatGLM-3-6b) and save comparisons such as results_table.tsv, with predictions under predictions/ and summaries under results/<dataset>/.

## Related tools

- **TTARAG** (Test-time adaptation method whose accuracy is compared against the no-segmentation ablation on CRAG) — https://github.com/sunxin000/TTARAG
- **Llama-3.1-8B-Instruct** (Generation model for producing answers under each configuration)
- **Qwen2.5-72B-Instruct** (LLM-as-a-judge scoring semantic equivalence between predictions and reference answers)
- **MiniLM-v6** (Reranks sentences in retrieved HTML and selects the top-20 retrieval content per query)
- **NVIDIA A100 GPU** (Single-GPU hardware used to process the 2,706 CRAG queries)

## Examples

```
python parallel_evaluation.py --do_generate --dataset pubmedqa --model_name models/Llama-3.1-8B-Instruct --do_tta --learning_rate 1e-5 --max_adapt_pairs 3 --cross_encoder_name models/ms-marco-MiniLM-L-6-v2
```

## Evaluation signals

- Accuracy pairs for the ablation reproduce the paper's Table 4 values: TTARAG at 31.9/30.5/25.7 and wo-seg at 30.8/30.1/25.0 across the three models
- The segmentation strategy shows consistent positive deltas over the no-segmentation ablation across all three model architectures
- For hyperparameter sweeps, peak accuracy falls within learning rates 1e-6 to 1e-5 with 3-5 adaptation pairs, with near-flat curves indicating insensitivity
- LLM-judge agreement with human annotators on a 100-example random sample should be around the reported 98%
- Summaries are written to results/<dataset>/ and predictions to predictions/ with per-configuration rows present in the tabulated output

## Limitations

- Processing 2,706 CRAG queries requires substantial compute (single NVIDIA A100 GPU in the paper); per-query TTA processing time ranges from 1.75s (1 pair) to 2.60s (5 pairs)
- Accuracy results depend on resetting model parameters to the pre-trained state between adaptation iterations; skipping this contaminates cross-configuration comparisons
- The six specialized domains and the base model/implementation details are not fully specified in some sections of the paper

## Evidence

- [methods] LLM-as-a-judge protocol: [evidence span withheld — anchor status: not_found]
- [methods] Judge-human agreement: [evidence span withheld — anchor status: not_found]
- [methods] Optimal hyperparameter range: [evidence span withheld — anchor status: not_found]
