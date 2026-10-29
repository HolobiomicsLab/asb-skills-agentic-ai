---
name: iterative-visual-textual-co-optimization
description: Use when a study needs to co-optimize paired visual and textual attack
  components across iterations — e.g. building or auditing a jailbreak pipeline with
  adversarial images plus steering prompts and per-iteration quality judging.
when_to_use_negative:
- The goal is single-pass safety bypass only, with no requirement that responses fulfill
  attacker intent (JPS's loop exists to optimize MIFR/response quality, not just ASR)
- The target model is a text-only LLM with no image input channel — the visual perturbation
  component of the loop is inapplicable
- Only black-box model access is available — the visual update step requires gradient-based
  adversarial image optimization
tools:
- name: Qwen2-VL-7B
  role: Target MLLM whose responses guide visual perturbation and prompt revision
  repo: https://github.com/thu-coai/JPS
- name: InternVL2-8B
  role: Alternative target MLLM with dedicated attack implementation (internvl2_8b_attack.py)
  repo: https://github.com/thu-coai/JPS
- name: MiniGPT4-13B
  role: Alternative compact target MLLM with dedicated attack implementation (minigpt4_13b_attack.py)
  repo: https://github.com/thu-coai/JPS
- name: GPT-4o Mini Judge
  role: Per-round judge evaluating response quality during Quality Assessment
  repo: https://github.com/thu-coai/JPS
- name: Python 3.8+
  role: Runtime for run_config.py batch attack/inference execution and the revision
    system
  repo: https://github.com/thu-coai/JPS
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1145/3746027.3754561
    title: 'JPS: Jailbreak Multimodal Large Language Models with Collaborative Visual
      Perturbation and Textual Steering'
  source_content_hash: sha256:ef1e17af9487fd23d2efaf92b3a6e83e213dc097c992ed0e6acaf606e878ca0a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thu-coai/JPS
  tools:
  - CUDA
  - GPT-4o Mini Judge
  - InternVL2-8B
  - MiniGPT4-13B
  - Python 3.8+
  - Qwen2-VL-7B
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# iterative-visual-textual-co-optimization

## Summary

Reconstructs JPS's iterative co-optimization loop in which target-guided adversarial image perturbations (L∞-constrained, e.g. 32/255) handle safety bypass while a multi-agent system iteratively revises a steering prompt, alternating visual and textual updates until revision_iteration_num (e.g. 5) rounds are exhausted.

## When to use

Use when a study needs to co-optimize paired visual and textual attack components across iterations — e.g. building or auditing a jailbreak pipeline with adversarial images plus steering prompts and per-iteration quality judging.

## When NOT to use

- The goal is single-pass safety bypass only, with no requirement that responses fulfill attacker intent (JPS's loop exists to optimize MIFR/response quality, not just ASR)
- The target model is a text-only LLM with no image input channel — the visual perturbation component of the loop is inapplicable
- Only black-box model access is available — the visual update step requires gradient-based adversarial image optimization

## Inputs

- YAML configuration templates from config_batch_process/ (model_*.yaml, dataset_*.yaml, common.yaml)
- 50-sample AdvBench-subset optimization set (advbench_subset.csv)
- Target multimodal model (Qwen2-VL-7B, InternVL2-8B, or MiniGPT4-13B)
- Initial steering prompt and target model responses

## Outputs

- Per-iteration adversarial images (*_iteration0_.png through *_iteration5_.png)
- Per-iteration steering prompts (*_iterationX_.txt)
- GPT-4o Mini Judge quality assessments per revision round
- Optimized image+prompt pair applicable to transfer inference on AdvBench, MMSafetyBench, and HarmBench

## How to apply

Load YAML templates from config_batch_process/ (model_*.yaml, dataset_*.yaml, common.yaml) and set attack_steps, learning_rate, perturbation_constraint (L∞, e.g. 32/255), and revision_iteration_num (e.g. 5). First optimize an initial adversarial image (iteration0) on the 50-sample AdvBench-subset via gradient-based visual perturbation against the target MLLM (e.g. Qwen2-VL-7B). Then run the multi-agent revision system (multi_agent_group.py coordinating revision_agent, prompt_feedback_agent, response_feedback_agent, summarize_agent) through four steps — Response Analysis, Prompt Revision, Visual Update, Quality Assessment — re-optimizing the adversarial image conditioned on each revised prompt. Save each round's artifacts as *_iterationX_.png and *_iterationX_.txt (e.g. under results/qwen2vl_7b/) and judge response quality with the GPT-4o Mini Judge each round. Repeat the analyze-revise-reoptimize cycle until revision_iteration_num is exhausted, then apply the optimized image+prompt pair in the Inference (Transfer) phase on AdvBench, MMSafetyBench, and HarmBench.

## Related tools

- **Qwen2-VL-7B** (Target MLLM whose responses guide visual perturbation and prompt revision) — https://github.com/thu-coai/JPS
- **InternVL2-8B** (Alternative target MLLM with dedicated attack implementation (internvl2_8b_attack.py)) — https://github.com/thu-coai/JPS
- **MiniGPT4-13B** (Alternative compact target MLLM with dedicated attack implementation (minigpt4_13b_attack.py)) — https://github.com/thu-coai/JPS
- **GPT-4o Mini Judge** (Per-round judge evaluating response quality during Quality Assessment) — https://github.com/thu-coai/JPS
- **Python 3.8+** (Runtime for run_config.py batch attack/inference execution and the revision system) — https://github.com/thu-coai/JPS

## Examples

```
python run_config.py --config_dir './config/advbench_subset' --type attack --gpus 4,5,6,7
```

## Evaluation signals

- Artifact check: exactly one PNG and one TXT per revision iteration exists (iteration0 through iteration5, e.g. in results/qwen2vl_7b/)
- The final iteration5 adversarial image and steering prompt load successfully and remain applicable for inference on new questions
- MIFR quality evaluation via quality_evaluate.py shows improved scores across iterations (JPS reports higher MIFR than existing methods)
- Each revision round runs all four multi-agent steps (Response Analysis, Prompt Revision, Visual Update, Quality Assessment) without dropping the visual update
- Perturbation hyperparameters (L∞ constraint, learning_rate, attack_steps) remain consistent across re-optimization rounds

## Limitations

- Optimization relies on only 50 AdvBench-subset training samples; transfer quality depends on generalization beyond this subset
- Requires CUDA GPU with at least 24GB memory recommended, so the loop is costly to run
- README lacks a discussion/changelog section ('No discussion section content available'), so long-term failure modes are not documented

## Evidence

- [methods] Visual and textual components undergo iterative co-optimization for enhanced performance: "These visual and textual components undergo iterative co-optimization for enhanced performance."
- [methods] Multi-agent steering prompt optimization: [evidence span withheld — anchor status: not_found]
- [readme] Visual Update step in the revision cycle: "3. **Visual Update**: Re-optimizing adversarial images with new prompts"
- [readme] Two-phase attack/inference execution: "python run_config.py --config_dir './config/advbench_subset' --type attack --gpus 4,5,6,7"
