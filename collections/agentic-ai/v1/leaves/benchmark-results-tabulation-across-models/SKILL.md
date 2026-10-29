---
name: benchmark-results-tabulation-across-models
description: Use when binary predictions from multiple LLMs across several benchmark
  dataset families must be scored, aggregated into a model-by-benchmark accuracy matrix,
  and summarized as a per-model mean.
when_to_use_negative:
- Scoring the causal-extraction task (Experiment 2, task slices 2a-2h, 8,925-row dataset),
  which uses similarity scoring in addition to exact match rather than binary exact-match
  detection scoring
- When only a single model on a single benchmark is evaluated and no cross-model,
  cross-benchmark aggregation matrix is needed
- When probabilistic or ranked model outputs are involved rather than binary causal-detection
  predictions
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3372
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: Runs the Experiment 1 inference script (main.py --data data/exp1.csv --model_name
    MODEL_NAME_OR_PATH) and the scoring/aggregation into the results table
  repo: github.com/sydneyanuyah/CausalDiscovery
- name: DeepSeek-R1-Distill-Llama-70B
  role: Best-performing detection model used as the reference point for validation
    (~49.57% C_detect)
  repo: github.com/sydneyanuyah/CausalDiscovery
- name: Qwen2.5-Coder-32B-Instruct
  role: One of the 13 evaluated models contributing rows to the D_{m,b} matrix; best
    extraction model at 47.12% C_extract
  repo: github.com/sydneyanuyah/CausalDiscovery
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1109/bigdata66926.2025.11401449
    title: Benchmarking LLMs for Pairwise Causal Discovery in Biomedical and Multi-Domain
      Contexts
  source_content_hash: sha256:d1a4c014491c62812cc42b8c0dd14313f3245a807f38631043b0936e6e11a788
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepSeek-R1-0528-Qwen3-8B
  - DeepSeek-R1-Distill-Llama-70B
  - DeepSeek-R1-Distill-Qwen-32B
  - Llama-3-8B
  - Llama-3.1-8B-Instruct
  - Llama-3.2-3B
  - Llama-3.2-3B-Instruct
  - Meta-Llama-3.1-8B
  - Meta-Llama-3.3-70B-Instruct
  - Mistral-7B-Instruct-v0.3
  - Mixtral-8x7B-Instruct-v0.1
  - Python
  - Qwen2.5-7B-Instruct
  - Qwen2.5-Coder-32B-Instruct
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3372
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.85
n_claims: 20
---

# Benchmark results tabulation across models

## Summary

Aggregates per-model, per-benchmark accuracy metrics (D_{m,b}) from exact-match-scored causal-detection predictions into a single results table (experiment1_results_table.csv) laid out like Table 5, from which per-model mean competence scores (C_detect) are computed.

## When to use

Use when binary predictions from multiple LLMs across several benchmark dataset families must be scored, aggregated into a model-by-benchmark accuracy matrix, and summarized as a per-model mean.

## When NOT to use

- Scoring the causal-extraction task (Experiment 2, task slices 2a-2h, 8,925-row dataset), which uses similarity scoring in addition to exact match rather than binary exact-match detection scoring
- When only a single model on a single benchmark is evaluated and no cross-model, cross-benchmark aggregation matrix is needed
- When probabilistic or ranked model outputs are involved rather than binary causal-detection predictions

## Inputs

- experiments/experiment1/ causal-detection dataset (data/exp1.csv, 3,008 rows)
- Four prompt-definition files: instruction-only, few-shot in-context learning, chain-of-thought, hybrid CoT+FICL
- Generated model outputs per model and per prompting strategy
- Gold binary causal-detection labels for exact-match scoring

## Outputs

- experiment1_results_table.csv with D_{m,b} accuracy values in the row/column layout of Table 5
- Per-model mean competence score C_detect
- Per-model, per-benchmark accuracy metrics D_{m,b}

## How to apply

Run the Experiment 1 inference script once per model (13 open-source LLMs from DeepSeek-R1-0528-Qwen3-8B through Qwen2.5-Coder-32B-Instruct) and once per prompting strategy (instruction-only, few-shot in-context learning, chain-of-thought, and hybrid CoT+FICL) on the 3,008-row detection dataset at experiments/experiment1/. Parse generated outputs and score each binary causal-detection prediction against the gold label using exact-match scoring. Aggregate predictions into per-model, per-benchmark accuracy metrics D_{m,b} over the dataset families (CausalNet, CausalProbe, SemEval2010Task8, MedCaus, CauseNet, FinCausal, CausalBench, CRASS, Coling22, ECI-B, PubMed, COPA slices). Compute C_detect for each model as the mean D_{m,b} across benchmarks and compile all values into experiment1_results_table.csv mirroring the row/column layout of Table 5. Validate by checking that the best detection model, DeepSeek-R1-Distill-Llama-70B, reproduces a mean C_detect of approximately 49.57%.

## Related tools

- **Python** (Runs the Experiment 1 inference script (main.py --data data/exp1.csv --model_name MODEL_NAME_OR_PATH) and the scoring/aggregation into the results table) — github.com/sydneyanuyah/CausalDiscovery
- **DeepSeek-R1-Distill-Llama-70B** (Best-performing detection model used as the reference point for validation (~49.57% C_detect)) — github.com/sydneyanuyah/CausalDiscovery
- **Qwen2.5-Coder-32B-Instruct** (One of the 13 evaluated models contributing rows to the D_{m,b} matrix; best extraction model at 47.12% C_extract) — github.com/sydneyanuyah/CausalDiscovery

## Examples

```
cd experiments/experiment1; python main.py --data data/exp1.csv --model_name DeepSeek-R1-Distill-Llama-70B
```

## Evaluation signals

- Reproduced table matches the reported reference: DeepSeek-R1-Distill-Llama-70B achieves mean C_detect ≈ 49.57%
- Every (model, benchmark) cell in experiment1_results_table.csv has a defined D_{m,b} value for all 13 models across the 12 dataset-family slices
- C_detect equals the arithmetic mean of each model's D_{m,b} values across benchmarks (row/column consistency of the table)
- Each binary prediction matches its gold label under exact-match scoring with no unparseable model outputs left unscored
- All runs used seed 4000 and 4-bit model loading as specified, ensuring reproducible inference conditions

## Limitations

- Even the best detection model (DeepSeek-R1-Distill-Llama-70B, 49.57% C_detect) detects fewer than half of causal links correctly, so absolute scores are low
- Models performed best on simple, explicit, single-sentence relations but performance plummeted for implicit relationships, links spanning multiple sentences, and texts with multiple causal pairs, so aggregate C_detect can mask slice-level disparities
- Exact-match binary scoring captures detection only; it does not cover extraction quality, which requires similarity scoring

## Evidence

- [results] DeepSeek-R1-Distill-Llama-70B was the best-performing model for causal detection, achieving a mean score of 49.57% (C_detect): [evidence span withheld — anchor status: not_found]
- [methods] Exact-match scoring: "`analysis/` contains exact-match, similarity-scoring, problem-case, and error-bucket files."
- [methods] Seed and 4-bit loading: "The scripts use seed `4000`. The Experiment 1 script uses 4-bit model loading."
- [readme] Dataset row count: "`experiments/experiment1/` contains the causal-detection data, prompt definitions, inference script, and model outputs. The dataset file contains 3,008 rows."
- [readme] Inference command: "python main.py \
  --data data/exp1.csv \
  --model_name MODEL_NAME_OR_PATH"
