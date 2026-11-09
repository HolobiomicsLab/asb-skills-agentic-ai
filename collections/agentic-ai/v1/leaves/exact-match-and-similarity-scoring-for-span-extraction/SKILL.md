---
name: exact-match-and-similarity-scoring-for-span-extraction
description: Use when evaluating free-text span extraction outputs (e.g., cause-effect
  phrase pairs) against gold annotations and you need both strict exact-match rates
  and softer similarity-based credit per model, dataset, and prompting strategy.
when_to_use_negative:
- Evaluating causal detection (Experiment 1, binary identification of whether a causal
  link exists) rather than span extraction — use the C_detect scoring path with main.py
  instead
- Extracted spans are summaries or paraphrases rather than verbatim phrase pulls —
  the exact-match component will understate performance; rely on the similarity-scoring
  files only if partial credit is acceptable
- Datasets outside the benchmark's 12 families (CausalNet, CausalProbe, SemEval2010Task8,
  MedCaus, CauseNet, FinCausal, CausalBench, CRASS, Coling22, ECI-B, PubMed, COPA)
  without comparable gold span annotations
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3376
tools:
- name: Python
  role: Runs the Experiment 2 inference and scoring pipeline (exp2.py) and pandas-based
    aggregation into per-model metrics
  repo: https://github.com/sydneyanuyah/CausalDiscovery
- name: Qwen2.5-Coder-32B-Instruct
  role: Best extraction model (C_extract ≈ 47.12%) used as the reference point for
    validating the scoring pipeline's output ordering
  repo: https://github.com/sydneyanuyah/CausalDiscovery
- name: DeepSeek-R1-Distill-Llama-70B
  role: Evaluated model whose extraction spans are scored against gold annotations
    alongside the other 12 models
  repo: https://github.com/sydneyanuyah/CausalDiscovery
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1109/bigdata66926.2025.11401449
    title: Benchmarking LLMs for Pairwise Causal Discovery in Biomedical and Multi-Domain
      Contexts
  source_content_hash: sha256:30542efea55f3a798570be55698c41dffd53a911c67da7b145088690ed29427e
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3376
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# Exact-match and similarity scoring for span extraction

## Summary

Scores LLM-generated cause-effect spans against gold annotations from the CausalDiscovery benchmark's Experiment 2 (task slices 2a-2h) using exact-match and similarity scoring, producing per-model, per-benchmark, per-reasoning-mode metrics aggregated into the composite C_extract score.

## When to use

Use when evaluating free-text span extraction outputs (e.g., cause-effect phrase pairs) against gold annotations and you need both strict exact-match rates and softer similarity-based credit per model, dataset, and prompting strategy.

## When NOT to use

- Evaluating causal detection (Experiment 1, binary identification of whether a causal link exists) rather than span extraction — use the C_detect scoring path with main.py instead
- Extracted spans are summaries or paraphrases rather than verbatim phrase pulls — the exact-match component will understate performance; rely on the similarity-scoring files only if partial credit is acceptable
- Datasets outside the benchmark's 12 families (CausalNet, CausalProbe, SemEval2010Task8, MedCaus, CauseNet, FinCausal, CausalBench, CRASS, Coling22, ECI-B, PubMed, COPA) without comparable gold span annotations

## Inputs

- Consolidated Experiment 2 dataset: experiments/experiment2/data/Experiment2.csv (8,925 rows, task slices 2a-2h)
- Gold cause-effect span annotations for each example
- Generated cause-effect span outputs from 13 open-source LLMs under 5 prompting strategies (written to results/experiment2/)

## Outputs

- Exact-match scoring files in analysis/
- Similarity-scoring files in analysis/
- Problem-case files and error-bucket files in analysis/
- Per-model, per-benchmark, per-reasoning-mode metrics X_{m,b,r} and composite C_extract scores
- Results table formatted like Table 5 and appendix Tables V-XIV (one row per model x prompting strategy)

## How to apply

Run the Experiment 2 inference script (python exp2.py --model MODEL_NAME_OR_PATH --data data/Experiment2.csv --exp_folder OUTPUT_FOLDER) over the consolidated 8,925-row Experiment2.csv covering task slices 2a-2h, using the exposed defaults batch size 64, top-k 40, temperature 0.7, maximum new-token count 256, and seed 4000. Then score each generated cause-effect span pair against the gold annotations with exact-match scoring (full credit only for verbatim span recovery) and similarity scoring (partial credit for near-matches), collecting problem-case and error-bucket files into analysis/. Aggregate raw match outcomes into per-model, per-benchmark, per-reasoning-mode metrics X_{m,b,r} and compute the composite extraction score C_extract for each of the 13 models under each of the five prompting strategies (zero-shot, few-shot ICL, chain-of-thought, Least-to-Most, ReAct). Compile one row per model x prompting strategy into a results table like Table 5 and appendix Tables V-XIV; expect the best model (Qwen2.5-Coder-32B-Instruct) to top out near 47.12% C_extract, so exact-match scores in this regime are strict and low absolute values are normal.

## Related tools

- **Python** (Runs the Experiment 2 inference and scoring pipeline (exp2.py) and pandas-based aggregation into per-model metrics) — https://github.com/sydneyanuyah/CausalDiscovery
- **Qwen2.5-Coder-32B-Instruct** (Best extraction model (C_extract ≈ 47.12%) used as the reference point for validating the scoring pipeline's output ordering) — https://github.com/sydneyanuyah/CausalDiscovery
- **DeepSeek-R1-Distill-Llama-70B** (Evaluated model whose extraction spans are scored against gold annotations alongside the other 12 models) — https://github.com/sydneyanuyah/CausalDiscovery

## Examples

```
python exp2.py --model Qwen/Qwen2.5-Coder-32B-Instruct --data data/Experiment2.csv --exp_folder OUTPUT_FOLDER
```

## Evaluation signals

- analysis/ directory contains all four artifact types: exact-match, similarity-scoring, problem-case, and error-bucket files
- Results table reproduces the reported ordering and magnitude of scores, with Qwen2.5-Coder-32B-Instruct attaining C_extract ≈ 47.12% as the best extraction model
- Per-model, per-benchmark, per-reasoning-mode metrics X_{m,b,r} cover all 13 models across the 2a-2h task slices for each of the five prompting strategies
- All rows of the 8,925-row Experiment2.csv are scored with no unscored generations, and outputs/timing files in results/experiment2/ are grouped by model and prompting strategy

## Limitations

- Best extraction performer reached only 47.12% C_extract, indicating major deficiencies in current models' ability to extract exact cause and effect phrases — low absolute scores are expected
- Models performed best on simple, explicit, single-sentence relations; performance plummeted for implicit relationships, links spanning multiple sentences, and texts with multiple causal pairs
- Exact-match scoring gives no credit for near-misses; similarity scoring must be consulted alongside it to distinguish span-boundary errors from outright extraction failures

## Evidence

- [other] Score the generated cause-effect spans against gold annotations using exact-match and similarity scoring, and collect problem-case and error-bucket files in analysis/.: [evidence span withheld — anchor status: not_found]
- [abstract] the best for extraction, Qwen2.5-Coder-32B-Instruct, reached just 47.12% ($C_{extract}$): "the best for extraction, Qwen2.5-Coder-32B-Instruct, reached just 47.12\% ($C_{extract}$)"
- [methods] `analysis/` contains exact-match, similarity-scoring, problem-case, and error-bucket files.: "`analysis/` contains exact-match, similarity-scoring, problem-case, and error-bucket files."
- [other] Aggregate the raw match outcomes into per-model, per-benchmark, per-reasoning-mode metrics X_{m,b,r} and compute the composite extraction score C_extract for each model.: [evidence span withheld — anchor status: not_found]
- [abstract] Models performed best on simple, explicit, single-sentence relations. However, performance plummeted for more difficult (and realistic) cases, such as implicit relationships, links spanning multiple: "Models performed best on simple, explicit, single-sentence relations. However, performance plummeted for more difficult (and realistic) cases, such as implicit relationships, links spanning multiple"
