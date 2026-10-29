---
name: multi-agent-steering-prompt-revision
description: 'Use when reproducing or extending the JPS attack''s textual component:
  an initial response from an MLLM (e.g. Qwen2-VL-7B) fails to fulfill the query''s
  intent despite safety-filter bypass, and the steering prompt must be refined over
  revision iterations.'
when_to_use_negative:
- The attack goal is only safety-filter bypass with low-quality responses — the multi-agent
  revision exists specifically to raise intent fulfillment (MIFR), not just ASR
- Single-agent revision suffices for your setup — the repository also provides single_agent_group.py
  as an alternative, so the multi-agent coordination is not mandatory
- Working on a text-only LLM jailbreak without an image modality — this loop is paired
  with target-guided adversarial image perturbation and co-optimized with it
tools:
- name: Qwen2-VL-7B
  role: Target MLLM whose responses are analyzed and steered; example results are
    stored under results/qwen2vl_7b/
  repo: https://github.com/thu-coai/JPS
- name: GPT-4o Mini Judge
  role: Judge used in the Quality Assessment stage to evaluate response-quality improvements
    each revision round
  repo: https://github.com/thu-coai/JPS
- name: Python 3.8+
  role: Runtime for the revision/ module (multi_agent_group.py, revision_agent.py,
    prompt_feedback_agent.py, response_feedback_agent.py, summarize_agent.py)
  repo: https://github.com/thu-coai/JPS
- name: CUDA
  role: GPU backend required to run the target model and the multi-agent revision
    system
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_006
  - task_007
  source_papers:
  - doi: 10.1145/3746027.3754561
    title: 'JPS: Jailbreak Multimodal Large Language Models with Collaborative Visual
      Perturbation and Textual Steering'
  source_content_hash: sha256:974dac3450985eb90a64fd5949df3759147aa8b58dfd1321a267fa272837dff1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thu-coai/JPS
  tools:
  - CUDA
  - GPT-4o Mini Judge
  - InternVL2-8B
  - MiniGPT4-13B
  - Python
  - Python 3.8+
  - Qwen2-VL-7B
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# multi-agent-steering-prompt-revision

## Summary

Iteratively revises a 'steering prompt' for a JPS jailbreak attack using a coordinated group of agents (response_feedback_agent, prompt_feedback_agent, revision_agent, summarize_agent) that analyze target-model responses and improve the prompt until the attacker's malicious intent is fulfilled, as measured by MIFR.

## When to use

Use when reproducing or extending the JPS attack's textual component: an initial response from an MLLM (e.g. Qwen2-VL-7B) fails to fulfill the query's intent despite safety-filter bypass, and the steering prompt must be refined over revision iterations.

## When NOT to use

- The attack goal is only safety-filter bypass with low-quality responses — the multi-agent revision exists specifically to raise intent fulfillment (MIFR), not just ASR
- Single-agent revision suffices for your setup — the repository also provides single_agent_group.py as an alternative, so the multi-agent coordination is not mandatory
- Working on a text-only LLM jailbreak without an image modality — this loop is paired with target-guided adversarial image perturbation and co-optimized with it

## Inputs

- Query text and target model response (results/generate/responses.) from an MLLM such as Qwen2-VL-7B
- YAML configuration with agent_model_path and revision_iteration_num (e.g. 5) from config_batch_process/ templates
- Current steering prompt from the previous iteration (*_iterationX_.txt)

## Outputs

- Revised steering prompt per iteration (*_iterationX_.txt under results/qwen2vl_7b/)
- Final optimized steering prompt (steering_prompt.txt)
- Consolidated feedback history from summarize_agent across iterations

## How to apply

Load the query, the model response (e.g. from results/generate/responses.), and the revision-agent model path (agent_model_path) from the YAML configuration. Run the response_feedback_agent to identify weaknesses in the response relative to the query's intent, then the prompt_feedback_agent to propose candidate improvements, and the revision_agent to merge feedback into an updated steering prompt; the summarize_agent consolidates feedback history across iterations. Repeat this cycle for revision_iteration_num iterations (the released example uses 5, i.e. revision5), writing intermediate prompts following the *_iterationX_.txt pattern under results/qwen2vl_7b/ and the final prompt to steering_prompt.txt. The revision loop is co-optimized with visual perturbation updates (L∞ constraint, e.g. 32/255, learning rate 1/255), so each revised prompt should be paired with a re-optimized adversarial image. Validate improvement via the Quality Assessment stage using the MIFR metric against the initial response.

## Related tools

- **Qwen2-VL-7B** (Target MLLM whose responses are analyzed and steered; example results are stored under results/qwen2vl_7b/) — https://github.com/thu-coai/JPS
- **GPT-4o Mini Judge** (Judge used in the Quality Assessment stage to evaluate response-quality improvements each revision round) — https://github.com/thu-coai/JPS
- **Python 3.8+** (Runtime for the revision/ module (multi_agent_group.py, revision_agent.py, prompt_feedback_agent.py, response_feedback_agent.py, summarize_agent.py)) — https://github.com/thu-coai/JPS
- **CUDA** (GPU backend required to run the target model and the multi-agent revision system)

## Examples

```
python run_config.py --config_dir './config/advbench_subset' --type attack --gpus 4,5,6,7
```

## Evaluation signals

- Exactly one TXT artifact exists per revision iteration, following the *_iterationX_.txt pattern (iteration1 through iteration5) under results/qwen2vl_7b/
- The final steering_prompt.txt loads successfully and is applicable for inference on new questions together with iteration5_.png
- MIFR (Malicious Intent Fulfillment Rate) evaluated via quality_evaluate.py on results/generate/responses. shows improvement over the initial (pre-revision) response
- Faster convergence than single-agent baselines, consistent with the paper's 'Faster convergence with multi-agent revision' finding
- GPT-4o Mini Judge quality assessments improve monotonically (or near-monotonically) across revision rounds

## Limitations

- The released example uses revision_iteration_num = 5 (revision5); other budgets are configurable but unvalidated in the paper artifacts
- The revision system depends on a revision-agent LLM specified via agent_model_path; changes to this agent change prompt quality
- Missing signals note: no discussion section content was available, so failure modes beyond those documented are unknown

## Evidence

- [abstract] abstract: "complemented by ''steering prompt'' optimized via a multi-agent system to specifically guide LLM responses fulfilling the attackers' intent"
- [methods] methods: "1. **Response Analysis**: Analyzing model responses to identify weaknesses"
- [readme] readme: "revision_iteration_num`: Number of revision iterations"
