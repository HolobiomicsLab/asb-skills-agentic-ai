---
name: multimodal-jailbreak-response-quality-assessment
description: Use when evaluating jailbreak attack outputs against MLLMs (e.g. Qwen2-VL-7B,
  InternVL2-8B, MiniGPT4-13B) and you need to score response quality/intent fulfillment
  (MIFR) rather than only attack success rate.
when_to_use_negative:
- When the goal is only to measure safety-filter bypass (plain ASR) with no interest
  in whether response content fulfills malicious intent
- When responses come from non-MLLM text-only attacks outside the JPS visual-perturbation
  + textual-steering setting
- When no Reasoning-LLM-based evaluator or judge model access is available, since
  MIFR cannot be computed without it
edam_operation: http://edamontology.org/operation_2428
tools:
- name: GPT-4o Mini Judge
  role: Judge model used to score jailbreak responses during ASR/MIFR evaluation
  repo: https://github.com/thu-coai/JPS
- name: LlamaGuard3 Judge
  role: Safety-classifier judge used to verify bypass and quality assessment outcomes
  repo: https://github.com/thu-coai/JPS
- name: HarmBench Judge
  role: Comprehensive harm evaluation judge applied to generated responses
  repo: https://github.com/centerforaisafety/HarmBench
- name: Qwen2-VL-7B
  role: Target multimodal LLM whose responses are being quality-assessed (e.g. outputs
    under results/qwen2vl_7b/)
  repo: https://github.com/thu-coai/JPS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1145/3746027.3754561
    title: 'JPS: Jailbreak Multimodal Large Language Models with Collaborative Visual
      Perturbation and Textual Steering'
  source_content_hash: sha256:d726c65129b2113d7254f582e1f6e6a79a3b1ed0bf40dd103fd562b8ee18e40f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thu-coai/JPS
  tools:
  - CUDA
  - GPT-4o Mini Judge
  - HarmBench Judge
  - LlamaGuard3 Judge
  - Qwen2-VL-7B
  edam_operation: http://edamontology.org/operation_2428
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# multimodal-jailbreak-response-quality-assessment

## Summary

Assesses whether jailbreak responses from multimodal LLMs actually fulfill the attacker's malicious intent, not just bypass safety filters, using the Malicious Intent Fulfillment Rate (MIFR) metric from the JPS paper plus ASR judges (GPT-4o Mini Judge, LlamaGuard3 Judge, HarmBench Judge).

## When to use

Use when evaluating jailbreak attack outputs against MLLMs (e.g. Qwen2-VL-7B, InternVL2-8B, MiniGPT4-13B) and you need to score response quality/intent fulfillment (MIFR) rather than only attack success rate.

## When NOT to use

- When the goal is only to measure safety-filter bypass (plain ASR) with no interest in whether response content fulfills malicious intent
- When responses come from non-MLLM text-only attacks outside the JPS visual-perturbation + textual-steering setting
- When no Reasoning-LLM-based evaluator or judge model access is available, since MIFR cannot be computed without it

## Inputs

- Generated jailbreak response text for each query (e.g. results/generate/responses outputs from Qwen2-VL-7B / InternVL2-8B / MiniGPT4-13B)
- The original malicious query / benign-looking prompt paired with each response
- Benchmark datasets: AdvBench, AdvBench-Subset (50-sample optimization subset), MMSafetyBench, HarmBench
- Judge/evaluator model configuration (GPT-4o Mini Judge, LlamaGuard3 Judge, HarmBench Judge, Reasoning-LLM-based MIFR evaluator)
- Initial (pre-revision) responses for before/after quality comparison

## Outputs

- Per-response ASR verdict from each judge (GPT-4o Mini Judge, LlamaGuard3 Judge, HarmBench Judge)
- Per-response and aggregate MIFR (Malicious Intent Fulfillment Rate) scores per MLLM and dataset
- Before/after MIFR comparison table showing quality improvement from multi-agent steering-prompt revisions

## How to apply

Load the query and the target MLLM's response (e.g. from results/generate/responses) together with the benchmark context (AdvBench, AdvBench-Subset, MMSafetyBench, HarmBench). Score bypass with ASR judges — GPT-4o Mini Judge, LlamaGuard3 Judge, HarmBench Judge — then compute MIFR using the Reasoning-LLM-based evaluator, which checks whether the response content actually fulfills the attacker's malicious intent rather than merely emitting a compliance prefix. Run the Quality Assessment stage after each steering-prompt revision iteration to verify MIFR improvement over the initial response. Aggregate MIFR per model/dataset and compare against baseline jailbreak methods; note that optimization should use only the 50-sample AdvBench-Subset and evaluation should be on held-out datasets. Report both ASR and MIFR since JPS's core claim is that ASR alone overstates attack quality.

## Related tools

- **GPT-4o Mini Judge** (Judge model used to score jailbreak responses during ASR/MIFR evaluation) — https://github.com/thu-coai/JPS
- **LlamaGuard3 Judge** (Safety-classifier judge used to verify bypass and quality assessment outcomes) — https://github.com/thu-coai/JPS
- **HarmBench Judge** (Comprehensive harm evaluation judge applied to generated responses) — https://github.com/centerforaisafety/HarmBench
- **Qwen2-VL-7B** (Target multimodal LLM whose responses are being quality-assessed (e.g. outputs under results/qwen2vl_7b/)) — https://github.com/thu-coai/JPS

## Evaluation signals

- MIFR scores on the target model should exceed those of existing bypass-only jailbreak methods ('Higher MIFR scores compared to existing methods')
- Quality Assessment stage should show monotonic MIFR improvement over the initial response across the 5 revision iterations (revision_iteration_num=5, revision5 outputs matching *_iterationX_.txt)
- ASR and MIFR should be consistent across judges (GPT-4o Mini Judge, LlamaGuard3 Judge, HarmBench Judge) rather than judge-specific artifacts
- Results should hold across multiple MLLMs (Qwen2-VL-7B, InternVL2-8B, MiniGPT4-13B) and multiple datasets (AdvBench, MMSafetyBench, HarmBench) — 'Consistent results across different models and datasets'
- Optimized components derived only from the 50-sample AdvBench-Subset should still transfer to held-out datasets ('Effective transfer from 50 training samples')

## Limitations

- MIFR depends on a Reasoning-LLM-based evaluator, so scores are judge-dependent and inherit that model's biases
- Optimization/steering results were tuned on only a 50-sample AdvBench subset, so quality assessments on very different query distributions may not transfer
- The paper provides no discussion-section content ('No discussion section content available'), so failure-mode analysis for the evaluator is undocumented

## Evidence

- [abstract] MIFR metric definition: "To evaluate the quality of attack outcomes, we propose the Malicious Intent Fulfillment Rate (MIFR) metric, assessed using a Reasoning-LLM-based evaluator"
- [abstract] Motivation: ASR alone is insufficient: "Current research predominantly focuses on maximizing attack success rate (ASR), often overlooking whether the generated responses actually fulfill the attacker's malicious intent"
- [methods] Quality Assessment workflow step: "4. **Quality Assessment**: Evaluating improvements in response quality"
- [methods] MIFR evaluation instruction: "Evaluate response quality using MIFR metrics:"
- [readme] Quality-focused framing from README: "**Quality-Focused Approach**: Emphasizes response quality and relevance rather than simple bypassing"
