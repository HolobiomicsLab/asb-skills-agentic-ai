---
name: perturbation-strategy-ablation-analysis
description: Use when benchmarking a visual jailbreak method whose adversarial image
  perturbation module exposes multiple perturbation-strategy conditions (Full/Patch/Grid)
  and you must quantify how strategy choice affects both ASR and MIFR.
when_to_use_negative:
- The analysis goal is a single-condition attack run and no strategy comparison is
  needed (Full, Patch, and Grid runs are redundant).
- The method under study has no image perturbation module or does not define Full/Patch/Grid
  perturbation-strategy conditions.
- Only response quality (MIFR) is of interest with a fixed perturbation strategy —
  the ablation design and per-strategy ASR comparison are unnecessary overhead.
edam_operation: http://edamontology.org/operation_2945
tools:
- name: JPS
  role: Reference implementation providing the attack factory (attack/attack_factory.py,
    qwen2vl_7b_attack.py), model/dataset factories, batch configuration runner (run_config.py,
    main.py), and quality_evaluate.py for the MIFR metric
  repo: https://github.com/thu-coai/JPS
- name: Qwen2-VL-7B
  role: Target multimodal LLM whose adversarial images are optimized under each perturbation
    strategy condition
- name: GPT-4o Mini Judge
  role: One of the configured judges used to compute per-condition Attack Success
    Rate
- name: LlamaGuard3 Judge
  role: One of the configured judges used to compute per-condition Attack Success
    Rate
- name: HarmBench Judge
  role: One of the configured judges used to compute per-condition Attack Success
    Rate
  repo: https://github.com/centerforaisafety/HarmBench
- name: AdvBench
  role: Standard jailbreak evaluation benchmark supplying prompts for attack and inference
    phases
  repo: https://github.com/RICommunity/TAP/blob/main/data/advbench_subset.csv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1145/3746027.3754561
    title: 'JPS: Jailbreak Multimodal Large Language Models with Collaborative Visual
      Perturbation and Textual Steering'
  source_content_hash: sha256:1d90940419cde11575173d57e3f7d3bf4a60c68564f969c13f0cb1343d60b643
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thu-coai/JPS
  tools:
  - AdvBench
  - CUDA
  - GPT-4o Mini Judge
  - HarmBench Judge
  - JPS
  - LlamaGuard3 Judge
  - Python
  - Qwen2-VL-7B
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# perturbation-strategy-ablation-analysis

## Summary

Reproduces the JPS paper's perturbation-strategy ablation by running the target-guided adversarial image perturbation module under its three defined conditions — Full (full-image), Patch (patch-based), and Grid (grid-based) — and comparing Attack Success Rate (ASR) and Malicious Intent Fulfillment Rate (MIFR) per strategy.

## When to use

Use when benchmarking a visual jailbreak method whose adversarial image perturbation module exposes multiple perturbation-strategy conditions (Full/Patch/Grid) and you must quantify how strategy choice affects both ASR and MIFR.

## When NOT to use

- The analysis goal is a single-condition attack run and no strategy comparison is needed (Full, Patch, and Grid runs are redundant).
- The method under study has no image perturbation module or does not define Full/Patch/Grid perturbation-strategy conditions.
- Only response quality (MIFR) is of interest with a fixed perturbation strategy — the ablation design and per-strategy ASR comparison are unnecessary overhead.

## Inputs

- AdvBench dataset (and AdvBench-Subset, 50-sample optimization subset) loaded via JPS data loaders
- Target multimodal LLM (Qwen2-VL-7B, InternVL2-8B, or MiniGPT4-13B) via the JPS model factory
- YAML attack configuration files specifying attack_steps, learning_rate, perturbation_constraint (L-infinity), and revision_iteration_num
- Perturbation-strategy condition setting: attack type 'Full', 'Patch', or 'Grid'

## Outputs

- Adversarial images per perturbation strategy (results/images/, e.g., *_iterationN_.png per revision iteration)
- Model responses per condition (results/generate/)
- Per-condition ASR computed by GPT-4o Mini Judge, LlamaGuard3 Judge, and HarmBench Judge (results/eval/)
- Per-condition MIFR quality scores from quality_evaluate.py
- Aggregated comparison table (perturbation_strategy_comparison.csv) of ASR and MIFR per strategy

## How to apply

Load AdvBench and the target model (e.g., Qwen2-VL-7B) via the JPS dataset loaders and model factory configured through YAML files in config/. For each of the three perturbation strategies — Full (full-image perturbation), Patch (patch-based perturbation), and Grid (grid-based perturbation) — run the attack (optimization) phase with the documented visual perturbation parameters: attack_steps for optimization steps, learning_rate for the perturbation, an L-infinity perturbation_constraint, and revision_iteration_num for revision iterations. Then run the inference (transfer) phase to generate model responses from the optimized adversarial images, compute per-condition ASR with the configured judges (GPT-4o Mini Judge, LlamaGuard3 Judge, HarmBench Judge), and compute MIFR per condition with quality_evaluate.py. Aggregate ASR and MIFR across strategies into a single comparison table (e.g., perturbation_strategy_comparison.csv); the expected pattern is that perturbation strategy choice affects both ASR and MIFR.

## Related tools

- **JPS** (Reference implementation providing the attack factory (attack/attack_factory.py, qwen2vl_7b_attack.py), model/dataset factories, batch configuration runner (run_config.py, main.py), and quality_evaluate.py for the MIFR metric) — https://github.com/thu-coai/JPS
- **Qwen2-VL-7B** (Target multimodal LLM whose adversarial images are optimized under each perturbation strategy condition)
- **GPT-4o Mini Judge** (One of the configured judges used to compute per-condition Attack Success Rate)
- **LlamaGuard3 Judge** (One of the configured judges used to compute per-condition Attack Success Rate)
- **HarmBench Judge** (One of the configured judges used to compute per-condition Attack Success Rate) — https://github.com/centerforaisafety/HarmBench
- **AdvBench** (Standard jailbreak evaluation benchmark supplying prompts for attack and inference phases) — https://github.com/RICommunity/TAP/blob/main/data/advbench_subset.csv

## Examples

```
python run_config.py --config_dir './config/advbench' --type attack --gpus 4,5,6,7
```

## Evaluation signals

- The judge-computed ASR per condition is consistent with the response files in results/generate/.
- MIFR scores per condition are reproducible by re-running quality_evaluate.py on the same generated responses.
- The aggregated table reproduces the reported qualitative pattern that perturbation strategy choice affects both ASR and MIFR.
- Each condition used the same documented attack parameters (attack_steps, learning_rate, L-infinity perturbation_constraint, revision_iteration_num) so conditions differ only by perturbation strategy.
- Optimization artifacts exist per condition and per revision iteration (e.g., *_iteration0_.png through *_iteration5_.png with matching steering-prompt .txt files).

## Limitations

- Optimization is performed on only a 50-sample AdvBench-subset, and components are then transferred to other datasets, so ablation results depend on this small optimization set.
- The available index lacks a discussion section ('No discussion section content available'), so no discussion-level caveats about strategy choice are recorded.
- Requires CUDA-compatible GPU with at least 24GB memory (recommended) to run the attack phase for all three conditions.

## Evidence

- [methods] Full/Patch/Grid perturbation conditions defined in the article's methods: "- **Full**: Full-image perturbation ... - **Patch**: Patch-based perturbation ... - **Grid**: Grid-based perturbation"
- [abstract] JPS target-guided visual perturbation rationale: "JPS utilizes target-guided adversarial image perturbations for effective safety bypass"
- [readme] Attack phase entry point for optimization runs: "python run_config.py --config_dir './config/advbench_subset' --type attack --gpus 4,5,6,7"
- [readme] Visual perturbation attack configuration parameters: [evidence span withheld — anchor status: not_found]
- [readme] MIFR quality evaluation script: "quality_evaluate.py        # MIFR quality evaluation"
