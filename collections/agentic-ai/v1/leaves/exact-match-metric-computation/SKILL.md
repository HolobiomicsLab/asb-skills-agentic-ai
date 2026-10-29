---
name: exact-match-metric-computation
description: Use when scoring binary causal-detection predictions or extracted cause-effect
  pairs from LLM outputs in the CausalDiscovery benchmark against gold labels to produce
  per-model accuracy tables like Table 5.
when_to_use_negative:
- When outputs require partial credit rather than strict string identity — e.g., extraction
  phrases that are semantically correct but lexically different — similarity scoring
  is the more appropriate metric in this pipeline
- When the goal is ranking models on tasks outside causal detection/extraction (e.g.,
  task slices 2a-2h sub-evaluations) without aggregating to C_detect/C_extract
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3376
tools:
- name: Python
  role: Language used to run inference and compute exact-match/similarity scores via
    the main.py and exp2.py scripts
  repo: https://github.com/sydneyanuyah/CausalDiscovery
- name: DeepSeek-R1-Distill-Llama-70B
  role: Best-performing detection model whose outputs are scored with exact match;
    reference C_detect ≈ 49.57% for validation
  repo: https://github.com/sydneyanuyah/CausalDiscovery
- name: Qwen2.5-Coder-32B-Instruct
  role: Best-performing extraction model whose outputs are scored (C_extract 47.12%)
  repo: https://github.com/sydneyanuyah/CausalDiscovery
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1109/bigdata66926.2025.11401449
    title: Benchmarking LLMs for Pairwise Causal Discovery in Biomedical and Multi-Domain
      Contexts
  source_content_hash: sha256:ffe67994888492611979665e654e15012e54114483b3b381d489d73ab9d29bd4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sydneyanuyah/CausalDiscovery
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3376
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6818181818181818
n_claims: 22
---

# exact-match-metric-computation

## Summary

Scores model-generated causal-detection/extraction outputs against gold labels using exact-match scoring, then aggregates per-model, per-benchmark accuracies (D_{m,b}) into capability scores such as C_detect (best 49.57%) and C_extract (best 47.12%).

## When to use

Use when scoring binary causal-detection predictions or extracted cause-effect pairs from LLM outputs in the CausalDiscovery benchmark against gold labels to produce per-model accuracy tables like Table 5.

## When NOT to use

- When outputs require partial credit rather than strict string identity — e.g., extraction phrases that are semantically correct but lexically different — similarity scoring is the more appropriate metric in this pipeline
- When the goal is ranking models on tasks outside causal detection/extraction (e.g., task slices 2a-2h sub-evaluations) without aggregating to C_detect/C_extract

## Inputs

- Parsed LLM generation outputs from experiments/experiment1/ (causal detection, 3,008-row dataset)
- Parsed LLM generation outputs from results/experiment2/ grouped by model and prompting strategy (extraction, 8,925-row consolidated dataset)
- Gold labels from the CSV data files (data/exp1.csv, data/Experiment2.csv)

## Outputs

- Exact-match scoring files in analysis/ (per-model, per-example correctness)
- Per-model, per-benchmark accuracy metrics D_{m,b} and mean capability scores (C_detect, C_extract) compiled into an experiment1_results_table.csv in Table 5 row/column layout
- Similarity-scoring, problem-case, and error-bucket files for extraction outputs

## How to apply

After running inference (e.g., Experiment 1 with seed 4000 and 4-bit model loading via `python main.py --data data/exp1.csv --model_name MODEL_NAME_OR_PATH`), parse each model's generated output and compare it to the gold label using exact-match scoring for binary causal-detection predictions. For Experiment 2 extraction outputs, apply both exact-match and similarity scoring, since extracted cause-effect phrases may be lexically valid but not string-identical. Aggregate exact-match predictions into per-model, per-benchmark accuracies D_{m,b} across the 12 dataset families (CausalNet, CausalProbe, SemEval2010Task8, MedCaus, CauseNet, FinCausal, CausalBench, CRASS, Coling22, ECI-B, PubMed, COPA). Compute C_detect as the mean D_{m,b} across benchmarks per model and compile results into a Table 5-style CSV; validate against the reference result that DeepSeek-R1-Distill-Llama-70B achieves C_detect ≈ 49.57%. Route flagged disagreements into problem-case and error-bucket analyses.

## Related tools

- **Python** (Language used to run inference and compute exact-match/similarity scores via the main.py and exp2.py scripts) — https://github.com/sydneyanuyah/CausalDiscovery
- **DeepSeek-R1-Distill-Llama-70B** (Best-performing detection model whose outputs are scored with exact match; reference C_detect ≈ 49.57% for validation) — https://github.com/sydneyanuyah/CausalDiscovery
- **Qwen2.5-Coder-32B-Instruct** (Best-performing extraction model whose outputs are scored (C_extract 47.12%)) — https://github.com/sydneyanuyah/CausalDiscovery

## Examples

```
python main.py --data data/exp1.csv --model_name DeepSeek-R1-Distill-Llama-70B
```

## Evaluation signals

- Validation check: the compiled results table reproduces the reported reference — DeepSeek-R1-Distill-Llama-70B achieves a mean C_detect of approximately 49.57%
- Schema check: D_{m,b} and C_detect values populate the row/column layout of Table 5 (experiment1_results_table.csv) for all 13 models
- Consistency check: exact-match files in analysis/ cover every (model, prompting strategy, benchmark) combination from the inference runs
- Extraction cross-check: similarity-scoring files exist alongside exact-match files to catch semantically valid but lexically different cause-effect pairs

## Limitations

- Exact-match scoring is brittle for extraction: cause-effect phrases that are semantically correct but not string-identical are marked wrong, so similarity scoring is needed alongside it
- Even the best models score below 50% (C_detect 49.57%, C_extract 47.12%), so scores near ceiling should not be expected and model differences are small
- Model performance plummets on implicit relationships, links spanning multiple sentences, and texts with multiple causal pairs, so aggregate exact-match scores mask slice-level failure modes

## Evidence

- [methods] analysis/ contains exact-match, similarity-scoring, problem-case, and error-bucket files.: "`analysis/` contains exact-match, similarity-scoring, problem-case, and error-bucket files."
- [abstract] Best detection model achieved 49.57% C_detect.: "The best model for detection, DeepSeek-R1-Distill-Llama-70B, only achieved a mean score of 49.57\% ($C_{detect}$)"
- [abstract] Extraction best model reached 47.12% C_extract.: "the best for extraction, Qwen2.5-Coder-32B-Instruct, reached just 47.12\% ($C_{extract}$)"
- [other] Validation via workflow: verify best detection model achieves ~49.57% mean C_detect.: [evidence span withheld — anchor status: not_found]
- [readme] README: analysis directory contents description.: "- `analysis/` contains exact-match, similarity-scoring, problem-case, and error-bucket files."
