---
name: adversarial-image-perturbation-optimization
description: Use when optimizing adversarial images against a multimodal LLM (e.g.,
  Qwen2-VL-7B) on a benchmark subset such as the 50-sample AdvBench-Subset, with L∞-constrained
  perturbations and iterative visual-textual co-optimization.
when_to_use_negative:
- The task is inference/transfer evaluation with already-optimized components — use
  the inference phase (run_config.py --type inference) instead of re-optimizing.
- The goal is textual-only prompt revision without a visual component — the multi-agent
  revision system alone (multi_agent_group.py) covers that, not visual perturbation
  optimization.
- The target system is a text-only LLM with no image input channel — visual perturbation
  is inapplicable.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Qwen2-VL-7B
  role: Target multimodal model whose safety is bypassed by the optimized adversarial
    perturbation; example artifacts are saved under results/qwen2vl_7b/
- name: Python 3.8+
  role: Runtime for the JPS attack phase (main.py, run_config.py, attack_factory.py)
    that executes the gradient-based perturbation optimization
  repo: https://github.com/thu-coai/JPS
- name: CUDA
  role: GPU backend required for the optimization loop; CUDA-compatible GPU with at
    least 24GB memory recommended
- name: AdvBench-Subset
  role: 50-sample optimization subset (from the TAP repository) on which steering
    prompts and adversarial images are optimized
  repo: https://github.com/RICommunity/TAP/blob/main/data/advbench_subset.csv
- name: GPT-4o Mini Judge
  role: Judge used for quality assessment each revision round during the iterative
    co-optimization loop
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_005
  - task_007
  - task_001
  source_papers:
  - doi: 10.1145/3746027.3754561
    title: 'JPS: Jailbreak Multimodal Large Language Models with Collaborative Visual
      Perturbation and Textual Steering'
  source_content_hash: sha256:596b626a2d10396d855d55a07bdf2e99542fadbbf72c6326bb8e60543dc903d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thu-coai/JPS
  tools:
  - AdvBench
  - AdvBench-Subset
  - CUDA
  - GPT-4o Mini Judge
  - InternVL2-8B
  - MiniGPT4-13B
  - Python
  - Python 3.8+
  - Qwen2-VL-7B
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# adversarial-image-perturbation-optimization

## Summary

Gradient-based optimization of target-guided adversarial image perturbations under an L-infinity constraint, as implemented in the JPS jailbreak method to bypass MLLM safety mechanisms while being iteratively co-optimized with a textual steering prompt.

## When to use

Use when optimizing adversarial images against a multimodal LLM (e.g., Qwen2-VL-7B) on a benchmark subset such as the 50-sample AdvBench-Subset, with L∞-constrained perturbations and iterative visual-textual co-optimization.

## When NOT to use

- The task is inference/transfer evaluation with already-optimized components — use the inference phase (run_config.py --type inference) instead of re-optimizing.
- The goal is textual-only prompt revision without a visual component — the multi-agent revision system alone (multi_agent_group.py) covers that, not visual perturbation optimization.
- The target system is a text-only LLM with no image input channel — visual perturbation is inapplicable.

## Inputs

- 50-sample AdvBench-Subset optimization data (advbench_subset.csv from the TAP repository)
- YAML configuration files (model_*.yaml, dataset_*.yaml, common.yaml) with attack_config (attack_steps, learning_rate, perturbation_constraint, revision_iteration_num)
- Target multimodal LLM (e.g., Qwen2-VL-7B) on a CUDA GPU with at least 24GB memory
- Current steering prompt from the multi-agent revision system (optional, for co-optimization)

## Outputs

- Per-iteration adversarial images (iteration0 through iteration5 as *_iterationX_.png in results/qwen2vl_7b/)
- Paired steering prompt files (*_iterationX_.txt)
- Optimized adversarial image and steering prompt (iteration5_.png, iteration5_.txt) for use in the inference/transfer phase

## How to apply

Load the target model (Qwen2-VL-7B, InternVL2-8B, or MiniGPT4-13B) and the 50-sample AdvBench-Subset (advbench_subset.csv from the TAP repository) via the JPS dataset loader and model factory with YAML configs from config_batch_process/ (model_*.yaml, dataset_*.yaml, common.yaml). Configure attack_config with attack_steps (number of optimization steps), learning_rate (example configs use 1/255), perturbation_constraint as an L∞ bound (example configs use 32/255, encoded in full_con32|255_lr1|255_revision5), and revision_iteration_num (example: 5). Run the attack (optimization) phase via run_config.py --type attack, and in each revision round re-optimize the adversarial image conditioned on the revised steering prompt (Visual Update) using the same perturbation hyperparameters. Choose the perturbation strategy among Full (full-image), Patch (patch-based), or Grid (grid-based) conditions for ablation. Save per-iteration artifacts as *_iterationX_.png (iteration0 through iteration5) and the corresponding steering prompts in results/qwen2vl_7b/, then transfer the final iteration5 image and prompt in the inference phase.

## Related tools

- **Qwen2-VL-7B** (Target multimodal model whose safety is bypassed by the optimized adversarial perturbation; example artifacts are saved under results/qwen2vl_7b/)
- **Python 3.8+** (Runtime for the JPS attack phase (main.py, run_config.py, attack_factory.py) that executes the gradient-based perturbation optimization) — https://github.com/thu-coai/JPS
- **CUDA** (GPU backend required for the optimization loop; CUDA-compatible GPU with at least 24GB memory recommended)
- **AdvBench-Subset** (50-sample optimization subset (from the TAP repository) on which steering prompts and adversarial images are optimized) — https://github.com/RICommunity/TAP/blob/main/data/advbench_subset.csv
- **GPT-4o Mini Judge** (Judge used for quality assessment each revision round during the iterative co-optimization loop)

## Examples

```
python run_config.py --config_dir './config/advbench_subset' --type attack --gpus 4,5,6,7
```

## Evaluation signals

- Exactly one PNG and one TXT artifact exist per revision iteration (iteration0 through iteration5) in results/qwen2vl_7b/.
- The final iteration5 adversarial image and steering prompt load successfully and are applicable for inference on new questions.
- Perturbation respects the configured L∞ constraint (e.g., 32/255) with learning rate 1/255 over the configured attack_steps.
- Judge-computed ASR per perturbation-strategy condition (Full/Patch/Grid) is consistent with the response files; MIFR scores are reproducible via quality_evaluate.py.
- The co-optimized final image+prompt yields higher MIFR and ASR than baselines when transferred from the 50 training samples to AdvBench, MMSafetyBench, and HarmBench.

## Limitations

- The reported results depend on optimization on only a 50-sample AdvBench-Subset; generalization beyond it (transfer across models/benchmarks) is an empirical claim to verify, not guaranteed.
- Perturbation strategy choice (Full vs. Patch vs. Grid) affects both ASR and MIFR, so results are conditional on the strategy configured.
- Requires a CUDA-compatible GPU with at least 24GB memory (recommended); convergence speed depends on the multi-agent vs. single-agent revision configuration.

## Evidence

- [methods] Target-guided adversarial image perturbations are used for effective safety bypass: [evidence span withheld — anchor status: not_found]
- [abstract] These visual and textual components undergo iterative co-optimization for enhanced performance.: "These visual and textual components undergo iterative co-optimization for enhanced performance."
- [methods] 3. **Visual Update**: Re-optimizing adversarial images with new prompts: "3. **Visual Update**: Re-optimizing adversarial images with new prompts"
- [methods] Full: Full-image perturbation; Patch: Patch-based perturbation; Grid: Grid-based perturbation: "- **Full**: Full-image perturbation"
- [readme] python run_config.py --config_dir './config/advbench_subset' --type attack --gpus 4,5,6,7: "python run_config.py --config_dir './config/advbench_subset' --type attack --gpus 4,5,6,7"
