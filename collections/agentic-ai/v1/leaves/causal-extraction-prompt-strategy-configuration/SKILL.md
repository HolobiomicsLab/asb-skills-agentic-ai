---
name: causal-extraction-prompt-strategy-configuration
description: Use when you need to reproduce or extend benchmarking of open-source
  LLMs on exact cause-effect phrase extraction from text across task slices 2a-2h
  with multiple prompting strategies.
when_to_use_negative:
- The goal is causal detection (deciding whether a causal link exists, Experiment
  1) rather than extracting exact cause and effect phrases — use the main.py Experiment
  1 script instead
- Input text involves implicit relationships, links spanning multiple sentences, or
  multiple causal pairs — the article reports performance plummets on these harder
  cases, so expect unreliable extractions
- You need a closed-source or non-Hugging Face model — the pipeline is built around
  Hugging Face Transformers with the 13 evaluated open-source models
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3376
- http://edamontology.org/topic_3474
tools:
- name: Python
  role: Runtime for the exp2.py inference script, pandas loading of Experiment2.csv,
    and scoring/aggregation scripts
  repo: https://github.com/sydneyanuyah/CausalDiscovery
- name: Hugging Face Transformers
  role: Model loading and generation for the 13 open-source LLMs under configured
    sampling parameters (batch 64, top-k 40, temperature 0.7, max 256 new tokens,
    seed 4000)
  repo: https://github.com/sydneyanuyah/CausalDiscovery
- name: Qwen2.5-Coder-32B-Instruct
  role: Best-performing extraction model, attaining C_extract ≈ 47.12%; benchmark
    target when validating the pipeline
  repo: https://github.com/sydneyanuyah/CausalDiscovery
- name: DeepSeek-R1-Distill-Llama-70B
  role: One of the 13 evaluated extraction models and the best detection model (C_detect
    49.57%), usable as a comparison point
  repo: https://github.com/sydneyanuyah/CausalDiscovery
- name: Qwen2.5-7B-Instruct
  role: One of the 13 open-source models run through exp2.py across the five prompting
    strategies
  repo: https://github.com/sydneyanuyah/CausalDiscovery
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1109/bigdata66926.2025.11401449
    title: Benchmarking LLMs for Pairwise Causal Discovery in Biomedical and Multi-Domain
      Contexts
  source_content_hash: sha256:81b0315b00d1eba179ee6ba8972cf9e0a673fb0346fc1efba810932984f22653
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3376
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# causal-extraction-prompt-strategy-configuration

## Summary

Configure and run LLM-based causal extraction (Task 2, slices 2a-2h) with five prompting strategies (zero-shot, few-shot ICL, chain-of-thought, Least-to-Most, ReAct) using the CausalDiscovery repository's exp2.py inference script, then score exact cause-effect phrase spans against gold annotations to compute C_extract.

## When to use

Use when you need to reproduce or extend benchmarking of open-source LLMs on exact cause-effect phrase extraction from text across task slices 2a-2h with multiple prompting strategies.

## When NOT to use

- The goal is causal detection (deciding whether a causal link exists, Experiment 1) rather than extracting exact cause and effect phrases — use the main.py Experiment 1 script instead
- Input text involves implicit relationships, links spanning multiple sentences, or multiple causal pairs — the article reports performance plummets on these harder cases, so expect unreliable extractions
- You need a closed-source or non-Hugging Face model — the pipeline is built around Hugging Face Transformers with the 13 evaluated open-source models

## Inputs

- experiments/experiment2/data/Experiment2.csv (consolidated causal-extraction dataset, 8,925 rows, task slices 2a-2h)
- Prompt definitions for five strategies: instruction-only/zero-shot, few-shot ICL, chain-of-thought, Least-to-Most, ReAct
- Hugging Face model name or path for one of the 13 evaluated open-source LLMs

## Outputs

- Evaluated outputs and timing files in results/experiment2/ grouped by model and prompting strategy
- Exact-match, similarity-scoring, problem-case, and error-bucket files in analysis/
- Per-model, per-benchmark, per-reasoning-mode metrics X_{m,b,r} and composite C_extract scores formatted like Table 5 (appendix Tables V-XIV)

## How to apply

Load the consolidated Experiment 2 dataset (experiments/experiment2/data/Experiment2.csv, 8,925 rows covering task slices 2a-2h) with pandas and attach prompt definitions for the five strategies: general instruction-only/zero-shot, few-shot in-context learning, chain-of-thought, Least-to-Most, and ReAct. For each of the 13 models, invoke 'python exp2.py --model MODEL_NAME_OR_PATH --data data/Experiment2.csv --exp_folder OUTPUT_FOLDER', relying on the exposed defaults: batch size 64, top-k 40, temperature 0.7, maximum new-token count 256, and seed 4000. Evaluated outputs and timing files are written to results/experiment2/ grouped by model and prompting strategy. Score generated cause-effect spans against gold annotations using exact-match and similarity scoring, collecting problem-case and error-bucket files in analysis/. Aggregate per-model, per-benchmark, per-reasoning-mode metrics X_{m,b,r} into the composite extraction score C_extract and compile a results table per model x prompting strategy (Table 5 and appendix Tables V-XIV).

## Related tools

- **Python** (Runtime for the exp2.py inference script, pandas loading of Experiment2.csv, and scoring/aggregation scripts) — https://github.com/sydneyanuyah/CausalDiscovery
- **Hugging Face Transformers** (Model loading and generation for the 13 open-source LLMs under configured sampling parameters (batch 64, top-k 40, temperature 0.7, max 256 new tokens, seed 4000)) — https://github.com/sydneyanuyah/CausalDiscovery
- **Qwen2.5-Coder-32B-Instruct** (Best-performing extraction model, attaining C_extract ≈ 47.12%; benchmark target when validating the pipeline) — https://github.com/sydneyanuyah/CausalDiscovery
- **DeepSeek-R1-Distill-Llama-70B** (One of the 13 evaluated extraction models and the best detection model (C_detect 49.57%), usable as a comparison point) — https://github.com/sydneyanuyah/CausalDiscovery
- **Qwen2.5-7B-Instruct** (One of the 13 open-source models run through exp2.py across the five prompting strategies) — https://github.com/sydneyanuyah/CausalDiscovery

## Examples

```
python exp2.py --model Qwen/Qwen2.5-Coder-32B-Instruct --data data/Experiment2.csv --exp_folder results/experiment2/qwen25coder32b
```

## Evaluation signals

- The compiled results table reproduces the reported ordering and magnitude of scores, with Qwen2.5-Coder-32B-Instruct attaining C_extract ≈ 47.12% as the best extraction model
- Row count and schema of Experiment2.csv match 8,925 rows covering task slices 2a-2h before inference
- results/experiment2/ contains evaluated outputs and timing files for every model x prompting-strategy combination
- analysis/ contains exact-match, similarity-scoring, problem-case, and error-bucket files for each run
- Generation config matches exposed defaults: batch size 64, top-k 40, temperature 0.7, max new tokens 256, seed 4000

## Limitations

- Even the best extraction model reached only 47.12% C_extract, indicating major deficiencies in current models' exact cause-effect phrase extraction ability
- Performance plummets for implicit relationships, links spanning multiple sentences, and texts with multiple causal pairs
- The discussion/limitations section text was absent from the extracted article content, so additional caveats may exist in the full paper

## Evidence

- [abstract] the best for extraction, Qwen2.5-Coder-32B-Instruct, reached just 47.12% ($C_{extract}$): "the best for extraction, Qwen2.5-Coder-32B-Instruct, reached just 47.12\% ($C_{extract}$)"
- [methods] python exp2.py --model MODEL_NAME_OR_PATH --data data/Experiment2.csv --exp_folder OUTPUT_FOLDER: "python exp2.py \
  --model MODEL_NAME_OR_PATH \
  --data data/Experiment2.csv \
  --exp_folder OUTPUT_FOLDER"
- [readme] The Experiment 2 script exposes defaults of batch size 64, top-k 40, temperature 0.7, and maximum new-token count 256.: "The Experiment 2 script exposes defaults of batch size `64`, top-k `40`, temperature `0.7`, and maximum new-token count `256`."
- [readme] The consolidated dataset file contains 8,925 rows.: "`experiments/experiment2/` contains the causal-extraction data, task slices 2a-2h, prompt definitions, inference scripts, and shell launch files. The consolidated dataset file contains 8,925 rows."
- [abstract] performance plummeted for more difficult (and realistic) cases, such as implicit relationships, links spanning multiple: "However, performance plummeted for more difficult (and realistic) cases, such as implicit relationships, links spanning multiple"
