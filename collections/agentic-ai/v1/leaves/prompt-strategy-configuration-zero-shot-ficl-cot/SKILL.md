---
name: prompt-strategy-configuration-zero-shot-ficl-cot
description: Use when running the benchmark's Experiment 1 or Experiment 2 inference
  over causal-detection or causal-extraction data and each of the 13 open-source models
  must be evaluated once per prompting strategy for a fair cross-model comparison.
when_to_use_negative:
- The goal is a single-model production inference run rather than a controlled comparison
  of prompting strategies across models
- The models under evaluation are closed-source/API-only — the scripts assume local
  4-bit loading of open-weight models
- The analysis targets fine-grained extraction sub-task slices (2a-2h) at the slice
  level, where per-slice evaluation rather than whole-strategy configuration is the
  focus
edam_operation: http://edamontology.org/operation_0305
edam_topics:
- http://edamontology.org/topic_3376
tools:
- name: Python
  role: Runtime for the Experiment 1 (main.py) and Experiment 2 (exp2.py) inference
    scripts that apply each prompting strategy to the causal datasets
  repo: github.com/sydneyanuyah/CausalDiscovery
- name: DeepSeek-R1-Distill-Llama-70B
  role: Best-performing detection model (C_detect 49.57%) evaluated under all four
    prompting strategies
  repo: github.com/sydneyanuyah/CausalDiscovery
- name: Qwen2.5-Coder-32B-Instruct
  role: Best-performing extraction model (C_extract 47.12%) evaluated under the Experiment
    2 prompting strategies including Least-to-Most and ReAct
  repo: github.com/sydneyanuyah/CausalDiscovery
- name: DeepSeek-R1-0528-Qwen3-8B
  role: One of the 13 open-source models run once per prompting strategy with seed
    4000 and 4-bit loading
  repo: github.com/sydneyanuyah/CausalDiscovery
- name: Mixtral-8x7B-Instruct-v0.1
  role: One of the 13 open-source models benchmarked across prompting strategies on
    the causal-detection and extraction tasks
  repo: github.com/sydneyanuyah/CausalDiscovery
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1109/bigdata66926.2025.11401449
    title: Benchmarking LLMs for Pairwise Causal Discovery in Biomedical and Multi-Domain
      Contexts
  source_content_hash: sha256:9c5399351f11f2180b78af68076955b4beab0d3ee839ad50403baf20c26c7111
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
  edam_operation: http://edamontology.org/operation_0305
  edam_topics:
  - http://edamontology.org/topic_3376
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8695652173913043
n_claims: 23
---

# prompt-strategy-configuration-zero-shot-ficl-cot

## Summary

Configure and execute the four prompting strategies used in this causal-discovery benchmark — instruction-only (zero-shot), few-shot in-context learning (FICL), chain-of-thought (CoT), and hybrid CoT+FICL — against open-source LLMs so that detection/extraction scores are comparable across models. Prompting strategy is treated as an experimental factor, not an afterthought.

## When to use

Use when running the benchmark's Experiment 1 or Experiment 2 inference over causal-detection or causal-extraction data and each of the 13 open-source models must be evaluated once per prompting strategy for a fair cross-model comparison.

## When NOT to use

- The goal is a single-model production inference run rather than a controlled comparison of prompting strategies across models
- The models under evaluation are closed-source/API-only — the scripts assume local 4-bit loading of open-weight models
- The analysis targets fine-grained extraction sub-task slices (2a-2h) at the slice level, where per-slice evaluation rather than whole-strategy configuration is the focus

## Inputs

- experiments/experiment1/data/exp1.csv (causal-detection dataset, 3,008 rows)
- experiments/experiment2/data/Experiment2.csv (causal-extraction dataset, 8,925 rows) with task slices 2a-2h
- Four Experiment 1 prompt-definition files (instruction-only, few-shot ICL, chain-of-thought, hybrid CoT+FICL)
- Hugging Face model name or path for one of the 13 evaluated open-source LLMs

## Outputs

- Per-model, per-prompting-strategy generated predictions (raw model outputs)
- Scored outputs in results/ and analysis/ (exact-match, similarity-scoring, problem-case, and error-bucket files)
- Per-model, per-benchmark accuracy metrics D_{m,b} and mean competence scores (C_detect, C_extract) compiled into results tables

## How to apply

For Experiment 1 (causal detection), run the inference script 'python main.py --data data/exp1.csv --model_name MODEL_NAME_OR_PATH' once per prompting strategy, pairing the 3,008-row dataset in experiments/experiment1/ with the four prompt-definition files: instruction-only, few-shot in-context learning, chain-of-thought, and hybrid chain-of-thought with few-shot in-context learning. Use seed 4000 and 4-bit model loading so runs are reproducible across models such as DeepSeek-R1-Distill-Llama-70B and Qwen2.5-Coder-32B-Instruct. For Experiment 2 (causal extraction), use 'python exp2.py' with the documented defaults (batch size 64, top-k 40, temperature 0.7, max new tokens 256, seed 4000) over the 8,925-row consolidated dataset and its task slices 2a-2h; here the strategy set also includes Least-to-Most and ReAct. Parse each strategy's generated outputs and score binary detection predictions by exact match, then aggregate into per-model, per-benchmark metrics (D_{m,b}) before computing mean competence scores such as C_detect. The rationale for testing multiple strategies is that prompting format materially changes scores — the paper observed models perform best on simple, explicit, single-sentence relations but drop on implicit or multi-sentence causal links, so only a strategy-controlled design isolates model skill from prompt effects.

## Related tools

- **Python** (Runtime for the Experiment 1 (main.py) and Experiment 2 (exp2.py) inference scripts that apply each prompting strategy to the causal datasets) — github.com/sydneyanuyah/CausalDiscovery
- **DeepSeek-R1-Distill-Llama-70B** (Best-performing detection model (C_detect 49.57%) evaluated under all four prompting strategies) — github.com/sydneyanuyah/CausalDiscovery
- **Qwen2.5-Coder-32B-Instruct** (Best-performing extraction model (C_extract 47.12%) evaluated under the Experiment 2 prompting strategies including Least-to-Most and ReAct) — github.com/sydneyanuyah/CausalDiscovery
- **DeepSeek-R1-0528-Qwen3-8B** (One of the 13 open-source models run once per prompting strategy with seed 4000 and 4-bit loading) — github.com/sydneyanuyah/CausalDiscovery
- **Mixtral-8x7B-Instruct-v0.1** (One of the 13 open-source models benchmarked across prompting strategies on the causal-detection and extraction tasks) — github.com/sydneyanuyah/CausalDiscovery

## Examples

```
cd experiments/experiment1 && python main.py --data data/exp1.csv --model_name MODEL_NAME_OR_PATH
```

## Evaluation signals

- Reference check: the best detection configuration reproduces DeepSeek-R1-Distill-Llama-70B achieving a mean C_detect of approximately 49.57%
- Reference check: the best extraction configuration reproduces Qwen2.5-Coder-32B-Instruct reaching C_extract of 47.12%
- Completeness: each of the 13 models must appear with results for each of the four Experiment 1 prompting strategies (instruction-only, FICL, CoT, hybrid)
- Reproducibility: runs use seed 4000; re-running a strategy with the same seed, 4-bit loading, and temperature 0.7 (Experiment 2) should match stored outputs in results/
- Scoring integrity: binary detection predictions scored by exact match against gold labels; analysis/ files must contain exact-match, similarity-scoring, problem-case, and error-bucket outputs

## Limitations

- Even the best model under the best prompting strategy detects fewer than half of causal links correctly (C_detect 49.57%), so no strategy configuration yields strong absolute performance
- Performance plummets on difficult, realistic cases — implicit relationships, causal links spanning multiple sentences, and texts with multiple causal pairs — regardless of prompting strategy
- The discussion section of the paper contains no changelog or limitations text, so additional caveats may be undocumented

## Evidence

- [abstract] We tested various prompting methods, from simple instructions (zero-shot) to more complex strategies like Chain-of-Thought (CoT) and Few-shot In-Context Learning (FICL).: "We tested various prompting methods, from simple instructions (zero-shot) to more complex strategies like Chain-of-Thought (CoT) and Few-shot In-Context Learning (FICL)."
- [methods] python main.py --data data/exp1.csv --model_name MODEL_NAME_OR_PATH: "python main.py \
  --data data/exp1.csv \
  --model_name MODEL_NAME_OR_PATH"
- [readme] The scripts use seed `4000`. The Experiment 1 script uses 4-bit model loading.: "The scripts use seed `4000`. The Experiment 1 script uses 4-bit model loading."
- [abstract] The best model for detection, DeepSeek-R1-Distill-Llama-70B, only achieved a mean score of 49.57\% ($C_{detect}$): "The best model for detection, DeepSeek-R1-Distill-Llama-70B, only achieved a mean score of 49.57\% ($C_{detect}$)"
- [abstract] Models performed best on simple, explicit, single-sentence relations. However, performance plummeted for more difficult (and realistic) cases: "Models performed best on simple, explicit, single-sentence relations. However, performance plummeted for more difficult (and realistic) cases, such as implicit relationships, links spanning multiple"
- [readme] The Experiment 2 script exposes defaults of batch size `64`, top-k `40`, temperature `0.7`, and maximum new-token count `256`.: "The Experiment 2 script exposes defaults of batch size `64`, top-k `40`, temperature `0.7`, and maximum new-token count `256`."
