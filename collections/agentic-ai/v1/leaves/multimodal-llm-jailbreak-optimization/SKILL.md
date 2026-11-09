---
name: multimodal-llm-jailbreak-optimization
description: Use when probing whether adversarial visual perturbations plus optimized
  steering prompts can elicit high-quality (not merely successful) harmful responses
  from multimodal LLMs such as Qwen2-VL-7B, InternVL2-8B, or MiniGPT4-13B, and when
  comparing attack quality across benchmarks like AdvBench.
when_to_use_negative:
- The goal is to measure only bypass/refusal rates without assessing whether responses
  fulfill the attacker's malicious intent — the paper explicitly notes research 'often
  overlooking' response quality, which this skill is designed to address.
- Only black-box text-only LLM access is available — JPS requires white-box gradient-based
  visual perturbation of images fed to multimodal models on CUDA GPUs.
- Fewer than ~24GB GPU memory is available (the README recommends at least 24GB for
  the target models).
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Qwen2-VL-7B
  role: Primary target multimodal LLM for optimization and ablation runs (e.g., multi-agent
    vs. single-agent, perturbation strategy comparisons)
  repo: https://github.com/thu-coai/JPS
- name: GPT-4o Mini Judge
  role: Judge used to compute Attack Success Rate on generated responses
  repo: https://github.com/thu-coai/JPS
- name: LlamaGuard3 Judge
  role: Safety judge for ASR computation across benchmark responses
  repo: https://github.com/thu-coai/JPS
- name: HarmBench Judge
  role: Harm-evaluation judge for ASR computation, tied to the HarmBench benchmark
  repo: https://github.com/centerforaisafety/HarmBench
- name: Python 3.8+
  role: Runtime for the JPS codebase (run_config.py, main.py, quality_evaluate.py,
    multi_agent_group.py)
  repo: https://github.com/thu-coai/JPS
- name: CUDA
  role: CUDA-compatible GPU with at least 24GB memory (recommended) for gradient-based
    perturbation optimization and MLLM inference
  repo: https://github.com/thu-coai/JPS
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_004
  source_papers:
  - doi: 10.1145/3746027.3754561
    title: 'JPS: Jailbreak Multimodal Large Language Models with Collaborative Visual
      Perturbation and Textual Steering'
  source_content_hash: sha256:f264d5457f12eab4e5e79c6beead0c577e6f320a14171ccdf77d383e014062eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thu-coai/JPS
  tools:
  - AdvBench
  - CUDA
  - GPT-4o Mini Judge
  - HarmBench Judge
  - InternVL2-8B
  - LlamaGuard3 Judge
  - MiniGPT4-13B
  - Python
  - Python 3.8+
  - Qwen2-VL-7B
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# multimodal-llm-jailbreak-optimization

## Summary

Optimizes jailbreak attacks against multimodal LLMs by co-optimizing L-infinity-constrained adversarial image perturbations with multi-agent-revised textual steering prompts over 5 revision iterations, then evaluates outcomes with both ASR judges and the Malicious Intent Fulfillment Rate (MIFR) quality metric.

## When to use

Use when probing whether adversarial visual perturbations plus optimized steering prompts can elicit high-quality (not merely successful) harmful responses from multimodal LLMs such as Qwen2-VL-7B, InternVL2-8B, or MiniGPT4-13B, and when comparing attack quality across benchmarks like AdvBench, MMSafetyBench, and HarmBench.

## When NOT to use

- The goal is to measure only bypass/refusal rates without assessing whether responses fulfill the attacker's malicious intent — the paper explicitly notes research 'often overlooking' response quality, which this skill is designed to address.
- Only black-box text-only LLM access is available — JPS requires white-box gradient-based visual perturbation of images fed to multimodal models on CUDA GPUs.
- Fewer than ~24GB GPU memory is available (the README recommends at least 24GB for the target models).

## Inputs

- AdvBench-subset (advbench_subset.csv, 50 optimization samples from the TAP repository)
- Evaluation benchmark configs: config/advbench, config/mmsafetybench, config/harmbench directories
- Target MLLM checkpoints: Qwen2-VL-7B, InternVL2-8B, MiniGPT4-13B
- YAML attack_config settings (attack_steps, learning_rate, perturbation_constraint, revision_iteration_num)

## Outputs

- Per-iteration adversarial images (*_iteration0_.png ... *_iteration5_.png) and steering prompts (*_iteration1_.txt ... *_iteration5_.txt) under results/
- Generated responses under results/generate/responses
- Judge-computed ASR scores (GPT-4o Mini, LlamaGuard3, HarmBench judges) and MIFR quality scores from quality_evaluate.py

## How to apply

Run the attack (optimization) phase on the 50-sample AdvBench-subset from the TAP repository, co-optimizing full-image adversarial perturbations under an L-infinity constraint (example configs: 32/255 with learning rate 1/255) and steering prompts through the multi-agent revision system (multi_agent_group.py: prompt_feedback_agent, response_feedback_agent, summarize_agent) for revision_iteration_num=5 iterations, each iteration executing Response Analysis, Prompt Revision, Visual Update, and Quality Assessment. Then run the inference (transfer) phase via run_config.py --type inference, applying the final iteration5_.png adversarial image and iteration5_.txt steering prompt to AdvBench, MMSafetyBench, and HarmBench configs for each target model. Compute Attack Success Rate with the configured judges (GPT-4o Mini Judge, LlamaGuard3 Judge, HarmBench Judge) and response quality with quality_evaluate.py using the MIFR metric, since the paper's premise is that ASR alone overlooks whether generated responses actually fulfill the attacker's malicious intent.

## Related tools

- **Qwen2-VL-7B** (Primary target multimodal LLM for optimization and ablation runs (e.g., multi-agent vs. single-agent, perturbation strategy comparisons)) — https://github.com/thu-coai/JPS
- **GPT-4o Mini Judge** (Judge used to compute Attack Success Rate on generated responses) — https://github.com/thu-coai/JPS
- **LlamaGuard3 Judge** (Safety judge for ASR computation across benchmark responses) — https://github.com/thu-coai/JPS
- **HarmBench Judge** (Harm-evaluation judge for ASR computation, tied to the HarmBench benchmark) — https://github.com/centerforaisafety/HarmBench
- **Python 3.8+** (Runtime for the JPS codebase (run_config.py, main.py, quality_evaluate.py, multi_agent_group.py)) — https://github.com/thu-coai/JPS
- **CUDA** (CUDA-compatible GPU with at least 24GB memory (recommended) for gradient-based perturbation optimization and MLLM inference) — https://github.com/thu-coai/JPS

## Examples

```
python run_config.py --config_dir './config/advbench_subset' --type attack --gpus 4,5,6,7
```

## Evaluation signals

- Per-iteration artifacts exist for all 5 revision iterations: results/ contains *_iteration5_.png (final adversarial image) and *_iteration5_.txt (final steering prompt) per sample.
- The multi-agent configuration reaches its MIFR quality plateau in fewer revision iterations than the single-agent configuration (faster convergence) in agent_comparison.csv.
- Judge-computed ASR per condition is consistent with the response files, and MIFR scores are reproducible via quality_evaluate.py.
- Transfer from the 50 optimization samples yields consistent per-model, per-benchmark ASR and MIFR in transfer_results.csv across Qwen2-VL-7B, InternVL2-8B, and MiniGPT4-13B on AdvBench, MMSafetyBench, and HarmBench.
- Optimized perturbations respect the L-infinity perturbation_constraint (example: 32/255 with learning rate 1/255) during attack_steps optimization.

## Limitations

- Optimization is restricted to a 50-sample AdvBench-subset, so generalization to other datasets relies on transfer rather than direct optimization.
- Only the multi-agent (multi_agent_group.py) and single-agent (single_agent_group.py) revision configurations are provided; results depend on the chosen revision_iteration_num (example: 5).
- Requires substantial compute: CUDA-compatible GPU with at least 24GB memory recommended; results depend on external API judges (GPT-4o Mini) whose outputs may vary.
- The paper's discussion section content is unavailable ('No discussion section content available'), so caveats around misuse or ethical considerations cannot be traced from the provided context.

## Evidence

- [abstract] JPS utilizes target-guided adversarial image perturbations for effective safety bypass: "JPS utilizes target-guided adversarial image perturbations for effective safety bypass"
- [abstract] complemented by 'steering prompt' optimized via a multi-agent system to specifically guide LLM responses fulfilling the attackers' intent: "complemented by ''steering prompt'' optimized via a multi-agent system to specifically guide LLM responses fulfilling the attackers' intent"
- [methods] First, optimize steering prompts and adversarial images on AdvBench-subset:: "First, optimize steering prompts and adversarial images on AdvBench-subset:"
- [methods] 50-sample optimization subset (available at [TAP Repository](https://github.com/RICommunity/TAP/blob/main/data/advbench_subset.csv)): "50-sample optimization subset (available at [TAP Repository](https://github.com/RICommunity/TAP/blob/main/data/advbench_subset.csv))"
- [readme] python run_config.py --config_dir './config/advbench_subset' --type attack --gpus 4,5,6,7: "python run_config.py --config_dir './config/advbench_subset' --type attack --gpus 4,5,6,7"
- [readme] README recommends at least 24GB GPU memory: "- CUDA-compatible GPU
- At least 24GB GPU memory (recommended)"
