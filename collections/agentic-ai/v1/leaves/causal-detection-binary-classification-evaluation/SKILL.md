---
name: causal-detection-binary-classification-evaluation
description: Use when you need to benchmark LLMs on binary causal-link detection across
  heterogeneous causal datasets (CausalNet, CausalProbe, SemEval2010Task8, MedCaus,
  CauseNet, FinCausal, CRASS, COPA, etc.) and aggregate results into a C_detect table.
when_to_use_negative:
- The goal is causal extraction (Task 2, slices 2a-2h) rather than binary detection
  — use the exp2.py workflow with C_extract scoring instead
- You need similarity-based scoring of free-text cause-effect phrases; detection here
  uses exact-match scoring of binary predictions only
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3376
tools:
- name: Python
  role: Runtime for the Experiment 1 inference and scoring scripts
  repo: github.com/sydneyanuyah/CausalDiscovery
- name: DeepSeek-R1-Distill-Llama-70B
  role: Best-performing detection model; reference for C_detect validation (~49.57%)
  repo: github.com/sydneyanuyah/CausalDiscovery
- name: Qwen2.5-Coder-32B-Instruct
  role: One of the 13 evaluated models scored on detection accuracy
  repo: github.com/sydneyanuyah/CausalDiscovery
- name: DeepSeek-R1-0528-Qwen3-8B
  role: One of the 13 evaluated models scored on detection accuracy
  repo: github.com/sydneyanuyah/CausalDiscovery
- name: Llama-3-8B
  role: One of the 13 evaluated models scored on detection accuracy
  repo: github.com/sydneyanuyah/CausalDiscovery
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1109/bigdata66926.2025.11401449
    title: Benchmarking LLMs for Pairwise Causal Discovery in Biomedical and Multi-Domain
      Contexts
  source_content_hash: sha256:720b8ac99bb8ed9a2f201d87752da295149ec99cb3628c9007a3ab208c69b6f7
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3376
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.88
n_claims: 25
---

# causal-detection-binary-classification-evaluation

## Summary

Reproduces the paper's Causal Detection (Task 1a-1d) evaluation: run 13 open-source LLMs over a 3,008-row binary causal-detection dataset under 4 prompting strategies, score exact-match predictions, and compute per-benchmark accuracies D_{m,b} and the mean competence score C_detect (best model: DeepSeek-R1-Distill-Llama-70B at ~49.57%).

## When to use

Use when you need to benchmark LLMs on binary causal-link detection across heterogeneous causal datasets (CausalNet, CausalProbe, SemEval2010Task8, MedCaus, CauseNet, FinCausal, CRASS, COPA, etc.) and aggregate results into a C_detect table.

## When NOT to use

- The goal is causal extraction (Task 2, slices 2a-2h) rather than binary detection — use the exp2.py workflow with C_extract scoring instead
- You need similarity-based scoring of free-text cause-effect phrases; detection here uses exact-match scoring of binary predictions only

## Inputs

- experiments/experiment1/data/exp1.csv (causal-detection dataset, 3,008 rows with binary gold labels)
- Four prompt-definition files: instruction-only, few-shot in-context learning, chain-of-thought, hybrid CoT+FICL
- Hugging Face model names/paths for the 13 open-source LLMs (e.g., DeepSeek-R1-Distill-Llama-70B, Llama-3.2-3B, Qwen2.5-Coder-32B-Instruct)

## Outputs

- Per-model, per-benchmark binary-detection accuracies D_{m,b} across dataset families (CausalNet, CausalProbe, SemEval2010Task8, MedCaus, CauseNet, FinCausal, CausalBench, CRASS, Coling22, ECI-B, PubMed, COPA)
- Per-model mean competence score C_detect
- experiment1_results_table.csv in the Table 5 row/column layout

## How to apply

Load the causal-detection dataset from experiments/experiment1/ (data/exp1.csv, 3,008 rows) together with the four prompt-definition files (instruction-only, few-shot in-context learning, chain-of-thought, and hybrid CoT+FICL). For each of the 13 evaluated models, run 'python main.py --data data/exp1.csv --model_name MODEL_NAME_OR_PATH' once per prompting strategy, using seed 4000 and 4-bit model loading. Parse generated outputs and score each binary causal-detection prediction against the gold label via exact-match scoring. Aggregate per-model, per-benchmark accuracies D_{m,b} across the dataset families, compute C_detect as the mean D_{m,b} per model, and compile results into experiment1_results_table.csv in the row/column layout of Table 5. Validate that the best detection model, DeepSeek-R1-Distill-Llama-70B, reproduces a mean C_detect of approximately 49.57%.

## Related tools

- **Python** (Runtime for the Experiment 1 inference and scoring scripts) — github.com/sydneyanuyah/CausalDiscovery
- **DeepSeek-R1-Distill-Llama-70B** (Best-performing detection model; reference for C_detect validation (~49.57%)) — github.com/sydneyanuyah/CausalDiscovery
- **Qwen2.5-Coder-32B-Instruct** (One of the 13 evaluated models scored on detection accuracy) — github.com/sydneyanuyah/CausalDiscovery
- **DeepSeek-R1-0528-Qwen3-8B** (One of the 13 evaluated models scored on detection accuracy) — github.com/sydneyanuyah/CausalDiscovery
- **Llama-3-8B** (One of the 13 evaluated models scored on detection accuracy) — github.com/sydneyanuyah/CausalDiscovery

## Examples

```
cd experiments/experiment1; python main.py --data data/exp1.csv --model_name deepseek-ai/DeepSeek-R1-Distill-Llama-70B
```

## Evaluation signals

- Reproduced Table 5 shows DeepSeek-R1-Distill-Llama-70B with mean C_detect ≈ 49.57% as the top detection model
- All 13 models × 4 prompting strategies are represented with per-benchmark D_{m,b} entries in experiment1_results_table.csv
- Binary predictions match gold labels via exact-match scoring with no unparsed model outputs
- Inference logs confirm seed 4000 and 4-bit model loading were used for every run
- Dataset coverage: all 3,008 rows of exp1.csv across the 12 dataset families are scored

## Limitations

- Even the best model detects fewer than half of causal links correctly (C_detect ≈ 49.57%), so absolute performance is low
- Models performed best on simple, explicit, single-sentence relations; performance plummeted for implicit relationships and links spanning multiple sentences
- Experiment 1 uses 4-bit model loading, which may quantitatively affect scores versus full-precision inference

## Evidence

- [abstract] The best model for detection, DeepSeek-R1-Distill-Llama-70B, only achieved a mean score of 49.57\% ($C_{detect}$): "The best model for detection, DeepSeek-R1-Distill-Llama-70B, only achieved a mean score of 49.57\% ($C_{detect}$)"
- [methods] Experiment 1 inference script: "python main.py \
  --data data/exp1.csv \
  --model_name MODEL_NAME_OR_PATH"
- [methods] Seed and loading: "The scripts use seed `4000`. The Experiment 1 script uses 4-bit model loading."
- [methods] Exact-match scoring: "`analysis/` contains exact-match, similarity-scoring, problem-case, and error-bucket files."
- [readme] Dataset size: "`experiments/experiment1/` contains the causal-detection data, prompt definitions, inference script, and model outputs. The dataset file contains 3,008 rows."
