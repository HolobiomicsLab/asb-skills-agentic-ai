---
name: per-model-benchmark-metric-aggregation
description: Use when raw model-vs-gold match outcomes exist for multiple LLMs, datasets,
  or prompting strategies and must be rolled up into per-model metrics like C_extract
  ≈ 47.12% (Qwen2.5-Coder-32B-Instruct).
when_to_use_negative:
- The goal is to score individual generations against gold spans — use the exact-match/similarity
  scoring step itself, not this aggregation skill
- Only one model or one prompting strategy was run and no cross-model ranking or composite
  score is needed
- The task is causal detection (Experiment 1, 3,008-row exp1.csv) and the target metric
  is C_detect rather than C_extract — aggregate via the Experiment 1 pipeline instead
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Qwen2.5-Coder-32B-Instruct
  role: Best extraction performer whose C_extract (47.12%) serves as the reference
    magnitude when validating the aggregated table
  repo: https://github.com/sydneyanuyah/CausalDiscovery
- name: DeepSeek-R1-Distill-Llama-70B
  role: Best detection performer whose C_detect (49.57%) anchors validation of the
    aggregated detection table
  repo: https://github.com/sydneyanuyah/CausalDiscovery
- name: Python
  role: Aggregation script language used to consolidate raw match outcomes into X_{m,b,r}
    metrics and composite scores
  repo: https://github.com/sydneyanuyah/CausalDiscovery
- name: Qwen2.5-7B-Instruct
  role: One of the 13 benchmarked models contributing rows to the per-model aggregation
    table
  repo: https://github.com/sydneyanuyah/CausalDiscovery
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1109/bigdata66926.2025.11401449
    title: Benchmarking LLMs for Pairwise Causal Discovery in Biomedical and Multi-Domain
      Contexts
  source_content_hash: sha256:77497e5616228aeb44445889fbc815214345e5ba3507dc7708c5bc26d97c3310
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sydneyanuyah/CausalDiscovery
  tools:
  - DeepSeek-R1-Distill-Llama-70B
  - Hugging Face Transformers
  - Python
  - Qwen2.5-7B-Instruct
  - Qwen2.5-Coder-32B-Instruct
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Per-model benchmark metric aggregation

## Summary

Aggregates raw exact-match and similarity-scored generation outcomes from the CausalDiscovery benchmark into per-model, per-dataset, per-prompting-strategy metrics and computes composite skill scores (C_extract, C_detect), formatted as a results table analogous to Table 5.

## When to use

Use when raw model-vs-gold match outcomes exist for multiple LLMs, datasets, or prompting strategies and must be rolled up into per-model metrics like C_extract ≈ 47.12% (Qwen2.5-Coder-32B-Instruct).

## When NOT to use

- The goal is to score individual generations against gold spans — use the exact-match/similarity scoring step itself, not this aggregation skill
- Only one model or one prompting strategy was run and no cross-model ranking or composite score is needed
- The task is causal detection (Experiment 1, 3,008-row exp1.csv) and the target metric is C_detect rather than C_extract — aggregate via the Experiment 1 pipeline instead

## Inputs

- Evaluated generation outputs and timing files in results/experiment2/ grouped by model and prompting strategy
- analysis/ exact-match and similarity-scoring files, plus problem-case and error-bucket files
- Consolidated Experiment 2 dataset (experiments/experiment2/data/Experiment2.csv, 8,925 rows; task slices 2a-2h) with gold cause-effect span annotations
- Prompt definitions for the five Experiment 2 strategies and four Experiment 1 strategies

## Outputs

- Per-model, per-benchmark, per-reasoning-mode metrics X_{m,b,r}
- Composite extraction score C_extract per model (and C_detect for Experiment 1)
- Single results table (Table 5 / appendix Tables V-XIV format), one row per model x prompting strategy

## How to apply

Start from the evaluated outputs in results/experiment2/ grouped by model and prompting strategy (zero-shot/instruction-only, few-shot in-context learning, chain-of-thought, Least-to-Most, ReAct) and the scored files in analysis/ (exact-match, similarity-scoring, problem-case, error-bucket). Aggregate raw match outcomes into per-model, per-benchmark, per-reasoning-mode metrics X_{m,b,r} across the 13 models and task slices 2a-2h of the 8,925-row Experiment2.csv. Compute the composite extraction score C_extract per model (and C_detect for Experiment 1) from these aggregates rather than averaging individual runs, so that difficulty effects from implicit, multi-sentence, and multi-pair cases remain visible. Compile the metrics into a single results table with one row per model x prompting strategy, mirroring Table 5 and appendix Tables V-XIV. Validate by checking that the ordering and magnitudes reproduce the reported results, with Qwen2.5-Coder-32B-Instruct as the best extraction model at C_extract ≈ 47.12% and DeepSeek-R1-Distill-Llama-70B the best detection model at C_detect ≈ 49.57%.

## Related tools

- **Qwen2.5-Coder-32B-Instruct** (Best extraction performer whose C_extract (47.12%) serves as the reference magnitude when validating the aggregated table) — https://github.com/sydneyanuyah/CausalDiscovery
- **DeepSeek-R1-Distill-Llama-70B** (Best detection performer whose C_detect (49.57%) anchors validation of the aggregated detection table) — https://github.com/sydneyanuyah/CausalDiscovery
- **Python** (Aggregation script language used to consolidate raw match outcomes into X_{m,b,r} metrics and composite scores) — https://github.com/sydneyanuyah/CausalDiscovery
- **Qwen2.5-7B-Instruct** (One of the 13 benchmarked models contributing rows to the per-model aggregation table) — https://github.com/sydneyanuyah/CausalDiscovery

## Examples

```
python exp2.py --model MODEL_NAME_OR_PATH --data data/Experiment2.csv --exp_folder OUTPUT_FOLDER
```

## Evaluation signals

- The compiled table reproduces the reported ordering and magnitude of scores, with Qwen2.5-Coder-32B-Instruct attaining C_extract ≈ 47.12% as the best extraction model
- Detection aggregation yields DeepSeek-R1-Distill-Llama-70B as best model with mean C_detect of 49.57%
- Every one of the 13 models x 5 Experiment-2 prompting strategies appears as a row in the results table
- Metrics X_{m,b,r} are populated for all benchmark datasets (e.g., CausalNet, SemEval2010Task8, FinCausal, CRASS) and reasoning modes with no missing cells
- Row counts reconcile with the source data (8,925 Experiment 2 rows; 3,008 Experiment 1 rows)

## Limitations

- Best composite scores are below 50% (C_extract 47.12%, C_detect 49.57%), indicating major deficiencies in current models' extraction/detection ability — aggregations inherit this ceiling
- Performance plummets for difficult cases such as implicit relationships, links spanning multiple sentences, and texts with multiple causal pairs, so per-slice aggregation is needed to avoid masking slice-level failure
- No discussion/limitations section was extractable from the provided document, so additional caveats may exist in the full paper

## Evidence

- [methods] Aggregate the raw match outcomes into per-model, per-benchmark, per-reasoning-mode metrics X_{m,b,r} and compute the composite extraction score C_extract: [evidence span withheld — anchor status: not_found]
- [abstract] the best for extraction, Qwen2.5-Coder-32B-Instruct, reached just 47.12\% ($C_{extract}$): "the best for extraction, Qwen2.5-Coder-32B-Instruct, reached just 47.12\% ($C_{extract}$)"
- [readme] results/experiment2/ contains evaluated outputs and timing files grouped by model and prompting strategy.: "`results/experiment2/` contains evaluated outputs and timing files grouped by model and prompting strategy."
- [readme] `analysis/` contains exact-match, similarity-scoring, problem-case, and error-bucket files.: "`analysis/` contains exact-match, similarity-scoring, problem-case, and error-bucket files."
- [results] confirm the table reproduces the reported ordering and magnitude of scores, with Qwen2.5-Coder-32B-Instruct attaining C_extract ≈ 47.12%: [evidence span withheld — anchor status: not_found]
