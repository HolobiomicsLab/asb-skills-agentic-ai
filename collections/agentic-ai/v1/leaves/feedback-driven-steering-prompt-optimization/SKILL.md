---
name: feedback-driven-steering-prompt-optimization
description: Use when a multimodal LLM jailbreak response passes safety filters but
  fails to fulfill the attacker's malicious intent, and the steering prompt must be
  improved through agent-driven feedback across revision iterations.
when_to_use_negative:
- The objective is only attack success rate (ASR) or safety-filter bypass measurement,
  not whether responses fulfill the attacker's malicious intent — the paper notes
  current research 'often overlooks' intent fulfillment, which is exactly the quality
  gap this loop targets.
- 'The pipeline is text-only with no multimodal target model: JPS co-optimizes the
  steering prompt with target-guided adversarial image perturbations on MLLMs such
  as Qwen2-VL-7B, InternVL2-8B, and MiniGPT4-13B.'
- No agent model or CUDA GPU with at least 24GB memory is available, since the revision
  agents require a separate LLM (agent_model_path) to run every iteration.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Qwen2-VL-7B
  role: Target MLLM whose response is analyzed each iteration and whose steering prompt
    is optimized; intermediate prompts are written under results/qwen2vl_7b/
- name: JPS revision agents (response_feedback_agent, prompt_feedback_agent, revision_agent,
    summarize_agent)
  role: 'Multi-agent loop: analyze response weaknesses, propose prompt improvements,
    merge feedback into the updated steering prompt, and consolidate history across
    iterations'
  repo: https://github.com/thu-coai/JPS
- name: HarmBench Judge
  role: Judge used in the evaluation pipeline to verify improvement of the optimized
    steering prompt via the Quality Assessment stage
  repo: https://github.com/centerforaisafety/HarmBench
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1145/3746027.3754561
    title: 'JPS: Jailbreak Multimodal Large Language Models with Collaborative Visual
      Perturbation and Textual Steering'
  source_content_hash: sha256:bf41cd4d1b26acdcd46bc16d8f1fc936a11609d3099d64a60ae019f4ab18f440
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thu-coai/JPS
  tools:
  - CUDA
  - HarmBench Judge
  - JPS revision agents (response_feedback_agent, prompt_feedback_agent, revision_agent,
    summarize_agent)
  - Qwen2-VL-7B
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# feedback-driven-steering-prompt-optimization

## Summary

The JPS multi-agent revision loop that iteratively refines a 'steering prompt' via response_feedback_agent, prompt_feedback_agent, revision_agent, and summarize_agent so that a target MLLM's jailbreak response fulfills the attacker's malicious intent rather than merely bypassing safety filters, with improvement verified by the MIFR metric.

## When to use

Use when a multimodal LLM jailbreak response passes safety filters but fails to fulfill the attacker's malicious intent, and the steering prompt must be improved through agent-driven feedback across revision iterations.

## When NOT to use

- The objective is only attack success rate (ASR) or safety-filter bypass measurement, not whether responses fulfill the attacker's malicious intent — the paper notes current research 'often overlooks' intent fulfillment, which is exactly the quality gap this loop targets.
- The pipeline is text-only with no multimodal target model: JPS co-optimizes the steering prompt with target-guided adversarial image perturbations on MLLMs such as Qwen2-VL-7B, InternVL2-8B, and MiniGPT4-13B.
- No agent model or CUDA GPU with at least 24GB memory is available, since the revision agents require a separate LLM (agent_model_path) to run every iteration.

## Inputs

- Query and the target MLLM's response (e.g., loaded from results/generate/responses)
- Current steering prompt text for the iteration
- YAML configuration specifying agent_model_path and revision_iteration_num
- Cross-iteration feedback history (input to summarize_agent)
- Target MLLM checkpoint (e.g., Qwen2-VL-7B) and its adversarial image for visual-textual co-optimization

## Outputs

- Per-iteration steering prompt files following the *_iterationX_.txt pattern under results/qwen2vl_7b/
- Final optimized steering prompt written to steering_prompt.txt
- Consolidated feedback history across iterations from summarize_agent
- MIFR-based Quality Assessment comparing the revised response against the initial response

## How to apply

Load the query, its target-model response (e.g., from results/generate/responses) and the revision-agent model path (agent_model_path) from the YAML configuration, then run the response_feedback_agent to identify weaknesses of the response relative to the query's intent. Pass those weaknesses to the prompt_feedback_agent, which proposes candidate improvements to the current steering prompt, and let the revision_agent merge both feedback streams into the updated steering prompt for the current iteration, while the summarize_agent consolidates the feedback history across iterations. Repeat this cycle for revision_iteration_num iterations (the released example uses 5, i.e. revision5), writing each intermediate steering prompt following the *_iterationX_.txt pattern under results/qwen2vl_7b/ and the final optimized prompt to steering_prompt.txt. Verify each cycle's gain through the Quality Assessment stage using the MIFR metric (a Reasoning-LLM-based evaluator) against the initial response; the rationale is that ASR-focused jailbreaks often yield low-quality, harmful-free outputs, so the loop drives responses toward actually fulfilling the attacker's intent in co-optimization with target-guided adversarial image perturbations.

## Related tools

- **Qwen2-VL-7B** (Target MLLM whose response is analyzed each iteration and whose steering prompt is optimized; intermediate prompts are written under results/qwen2vl_7b/)
- **JPS revision agents (response_feedback_agent, prompt_feedback_agent, revision_agent, summarize_agent)** (Multi-agent loop: analyze response weaknesses, propose prompt improvements, merge feedback into the updated steering prompt, and consolidate history across iterations) — https://github.com/thu-coai/JPS
- **HarmBench Judge** (Judge used in the evaluation pipeline to verify improvement of the optimized steering prompt via the Quality Assessment stage) — https://github.com/centerforaisafety/HarmBench

## Evaluation signals

- Intermediate steering prompts appear as *_iterationX_.txt files under results/qwen2vl_7b/ for each of the revision_iteration_num rounds (5 in the released revision5 example).
- A final optimized steering_prompt.txt is written after the last iteration.
- The Quality Assessment stage reports improved MIFR for the response generated with the final steering prompt versus the initial response.
- Each iteration's prompt is the revision_agent's merge of response_feedback_agent weaknesses and prompt_feedback_agent candidate improvements, with summarize_agent consolidating the history.
- Revised responses show higher malicious-intent fulfillment (MIFR) while the target-guided adversarial image perturbation still achieves the safety bypass.

## Limitations

- Quality is scored by a Reasoning-LLM-based MIFR evaluator, so revision decisions inherit that evaluator's judgment and potential bias.
- Generalization is demonstrated from a small 50-sample AdvBench-subset optimization set; transfer to other datasets relies on that subset.
- Recommended setup requires a CUDA-compatible GPU with at least 24GB of memory, and the loop runs a fixed revision_iteration_num (released example: 5) rather than a convergence-based stopping criterion.

## Evidence

- [abstract] complemented by ''steering prompt'' optimized via a multi-agent system to specifically guide LLM responses fulfilling the attackers' intent: "complemented by ''steering prompt'' optimized via a multi-agent system to specifically guide LLM responses fulfilling the attackers' intent"
- [methods] 1. **Response Analysis**: Analyzing model responses to identify weaknesses: "1. **Response Analysis**: Analyzing model responses to identify weaknesses"
- [methods] 2. **Prompt Revision**: Using specialized agents to improve steering prompts: "2. **Prompt Revision**: Using specialized agents to improve steering prompts"
- [methods] 4. **Quality Assessment**: Evaluating improvements in response quality: "4. **Quality Assessment**: Evaluating improvements in response quality"
- [readme] Multi-Agent Framework: Utilizes revision agents to iteratively improve steering prompts: "Multi-Agent Framework: Utilizes revision agents to iteratively improve steering prompts"
- [other] Repeat steps 2-5 for revision_iteration_num iterations (the released example uses 5, i.e. revision5): [evidence span withheld — anchor status: not_found]
- [other] Write the final optimized steering prompt to steering_prompt.txt and verify improvement via the Quality Assessment stage (MIFR metric) against the initial response.: [evidence span withheld — anchor status: not_found]
