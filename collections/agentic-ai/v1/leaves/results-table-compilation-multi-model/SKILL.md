---
name: results-table-compilation-multi-model
description: Use when you have evaluated outputs and scoring files for multiple models
  × datasets × prompting strategies and must consolidate them into one comparative
  results table with composite skill scores.
when_to_use_negative:
- You are analyzing causal-detection (Experiment 1) outputs rather than the 2a-2h
  extraction slices — Experiment 1 uses a different script (main.py), 4-bit loading,
  and different prompting strategies
- You only have a single model's raw generations without scored exact-match/similarity
  outputs, so no X_{m,b,r} metrics can be aggregated
- You need statistical significance testing across models rather than descriptive
  metric compilation
edam_operation: http://edamontology.org/operation_2409
tools:
- name: Python
  role: Pandas-based loading and aggregation of the consolidated Experiment 2 dataset
    and scored outputs into metrics X_{m,b,r} and C_extract
  repo: https://github.com/sydneyanuyah/CausalDiscovery
- name: Qwen2.5-Coder-32B-Instruct
  role: Expected top extraction model (C_extract ≈ 47.12%) used as the validation
    anchor for the compiled table's ordering
  repo: https://github.com/sydneyanuyah/CausalDiscovery
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1109/bigdata66926.2025.11401449
    title: Benchmarking LLMs for Pairwise Causal Discovery in Biomedical and Multi-Domain
      Contexts
  source_content_hash: sha256:e770c79b46638ad0a1e9d55f111cad1a4ecca52bba20d281e0f7e4bc662fdae7
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# Results-table compilation across models, benchmarks, and prompting strategies

## Summary

Aggregates raw exact-match/similarity outcomes from multi-LLM causal-extraction runs into per-model, per-benchmark, per-reasoning-mode metrics (X_{m,b,r}) and composite scores (C_extract), formatted as a single results table like Table 5 and appendix Tables V-XIV of the CausalDiscovery benchmark.

## When to use

Use when you have evaluated outputs and scoring files for multiple models × datasets × prompting strategies and must consolidate them into one comparative results table with composite skill scores.

## When NOT to use

- You are analyzing causal-detection (Experiment 1) outputs rather than the 2a-2h extraction slices — Experiment 1 uses a different script (main.py), 4-bit loading, and different prompting strategies
- You only have a single model's raw generations without scored exact-match/similarity outputs, so no X_{m,b,r} metrics can be aggregated
- You need statistical significance testing across models rather than descriptive metric compilation

## Inputs

- Evaluated model outputs and timing files grouped by model and prompting strategy (results/experiment2/)
- Consolidated Experiment 2 CSV (experiments/experiment2/data/Experiment2.csv, 8,925 rows, slices 2a-2h)
- Exact-match, similarity-scoring, problem-case, and error-bucket files from analysis/

## Outputs

- Single results table formatted like Table 5 and appendix Tables V-XIV, one row per model × prompting strategy
- Per-model, per-benchmark, per-reasoning-mode metrics X_{m,b,r}
- Composite extraction score C_extract per model

## How to apply

Load the consolidated Experiment 2 dataset (experiments/experiment2/data/Experiment2.csv, 8,925 rows covering task slices 2a-2h) with pandas, then for each of the 13 models and 5 prompting strategies (zero-shot instruction-only, few-shot in-context learning, chain-of-thought, Least-to-Most, ReAct) read the scored outputs grouped under results/experiment2/. Aggregate raw exact-match and similarity-scoring outcomes into per-model, per-benchmark, per-reasoning-mode metrics X_{m,b,r} using the seed-4000 run outputs, and compute the composite extraction score C_extract per model. Compile all rows into one table formatted like Table 5 (and appendix Tables V-XIV), one row per model × prompting strategy. Validate by checking that Qwen2.5-Coder-32B-Instruct attains C_extract ≈ 47.12% as the best extraction model and that the table reproduces the reported ordering and magnitudes.

## Related tools

- **Python** (Pandas-based loading and aggregation of the consolidated Experiment 2 dataset and scored outputs into metrics X_{m,b,r} and C_extract) — https://github.com/sydneyanuyah/CausalDiscovery
- **Qwen2.5-Coder-32B-Instruct** (Expected top extraction model (C_extract ≈ 47.12%) used as the validation anchor for the compiled table's ordering) — https://github.com/sydneyanuyah/CausalDiscovery

## Examples

```
import pandas as pd; df = pd.read_csv('experiments/experiment2/data/Experiment2.csv')
```

## Evaluation signals

- Qwen2.5-Coder-32B-Instruct attains C_extract ≈ 47.12% as the best extraction model in the compiled table
- Table reproduces the reported ordering and magnitude of scores relative to Table 5 / appendix Tables V-XIV
- One row exists for each of the 13 models × 5 prompting strategies (instruction-only/zero-shot, few-shot in-context learning, chain-of-thought, Least-to-Most, ReAct)
- Metrics aggregate exact-match and similarity-scoring outcomes consistently across the 2a-2h task slices of the 8,925-row dataset

## Limitations

- Even the best model (Qwen2.5-Coder-32B-Instruct) reached only 47.12% C_extract, so compiled tables reflect major model deficiencies rather than near-ceiling performance
- Scores are sensitive to prompting strategy; inference used fixed defaults (seed 4000, temperature 0.7, top-k 40, batch size 64, max 256 new tokens), so recompilation assumes those settings
- No discussion/limitations section content is available in the provided document, so additional caveats are not extractable

## Evidence

- [methods] Aggregate the raw match outcomes into per-model, per-benchmark, per-reasoning-mode metrics X_{m,b,r} and compute the composite extraction score C_extract for each model.: [evidence span withheld — anchor status: not_found]
- [methods] Qwen2.5-Coder-32B-Instruct attaining C_extract ≈ 47.12% as the best extraction model.: [evidence span withheld — anchor status: not_found]
- [abstract] the best for extraction, Qwen2.5-Coder-32B-Instruct, reached just 47.12\% ($C_{extract}$): "the best for extraction, Qwen2.5-Coder-32B-Instruct, reached just 47.12\% ($C_{extract}$)"
- [readme] results/experiment2/ contains evaluated outputs and timing files grouped by model and prompting strategy.: "`results/experiment2/` contains evaluated outputs and timing files grouped by model and prompting strategy."
- [methods] `analysis/` contains exact-match, similarity-scoring, problem-case, and error-bucket files.: "`analysis/` contains exact-match, similarity-scoring, problem-case, and error-bucket files."
