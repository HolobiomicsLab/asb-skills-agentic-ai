---
name: function-call-action-format-specification
description: Use when rebuilding or adapting LLM prompt artifacts that must convert
  KSPDG telemetry observations into discrete throttle actions via a perform_action
  function call with step-by-step (CoT) reasoning.
when_to_use_negative:
- The task is fine-tuning GPT-3.5 or LLaMA models on human-gameplay data rather than
  constructing prompt-engineered few-shot agents
- The target environment does not emit KSPDG-style observations or accept discrete
  x/y/z throttle actions
- Free-form natural-language action output is acceptable and no machine-parseable
  perform_action call is required
tools:
- name: Kerbal Space Program (KSP)
  role: Game engine hosting the KSPDG pursuit-evasion environment whose real-time
    observations are rendered as the user prompt and into which the perform_action
    throttles are fed
- name: KSPDG
  role: Gymnasium/PettingZoo environment suite (Space Gym) that defines the observation
    format the CoT prompt reasons over and the action interface perform_action targets
  repo: https://github.com/mit-ll/spacegym-kspdg
- name: GPT-3.5
  role: Target LLM prompted via the system/user prompt pair to produce step-by-step
    reasoning and perform_action function calls
- name: tiktoken
  role: Tokeniser used to measure prompt size when assembling the few-shot user prompt
    from telemetry observations
  repo: https://github.com/openai/tiktoken
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:ce454d2a6597100af6cb536d589ae9e8c0322ffc8a7186161ed80114324dd3db
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mit-ll/spacegym-kspdg
  tools:
  - GPT-3.5
  - KSPDG
  - Kerbal Space Program (KSP)
  - tiktoken
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# function-call-action-format-specification

## Summary

Reconstructs the Chain-of-Thought few-shot prompting artifacts (system prompt and one-shot user prompt) that elicit step-by-step reasoning and perform_action(ft, rt, dt) function calls from a GPT-3.5 spacecraft operator controlling a pursuit craft in the KSPDG differential-game environment.

## When to use

Use when rebuilding or adapting LLM prompt artifacts that must convert KSPDG telemetry observations into discrete throttle actions via a perform_action function call with step-by-step (CoT) reasoning.

## When NOT to use

- The task is fine-tuning GPT-3.5 or LLaMA models on human-gameplay data rather than constructing prompt-engineered few-shot agents
- The target environment does not emit KSPDG-style observations or accept discrete x/y/z throttle actions
- Free-form natural-language action output is acceptable and no machine-parseable perform_action call is required

## Inputs

- Appendix C prompting section of the paper (system prompt Figure C.8, user prompt Figure C.9)
- KSPDG environment observation tuples (pursuer/evader positions and velocities in celestial body and vessel reference frames)
- KSPDG action convention (throttles ft, rt, dt in x/y/z directions with maximum throttle of 1)

## Outputs

- system_prompt.txt (autonomous pursuit-agent system prompt ending in 'Reason step-by-step')
- user_prompt_template.txt (one-shot HUMAN/ASSISTANT Chain-of-Thought template ending in perform_action(ft, rt, dt))

## How to apply

Read the paper's Appendix C prompting section, which presents the system prompt (Figure C.8) and user prompt (Figure C.9) verbatim. Write system_prompt.txt reproducing the autonomous pursuit-spacecraft agent role: applying throttles to capture the evader using positions/velocities in the celestial body reference frame plus prograde direction, the vessel reference frame convention (x axis right, y axis toward target, z axis up), a maximum throttle of 1, and the closing instruction 'Reason step-by-step'. Write user_prompt_template.txt as a one-shot template with HUMAN/ASSISTANT role keywords whose worked Chain-of-Thought example reasons over the sign of each prograde coordinate (x, y, z) and concludes with the function call format perform_action(ft, rt, dt) using discrete thrust labels ('forward', 'left', 'up'). The template then instructs the model to answer the same throttle question given current KSPDG environment observations, so the model's final token stream is a parseable perform_action call that can be plugged back into the KSPDG environment in the RL loop.

## Related tools

- **Kerbal Space Program (KSP)** (Game engine hosting the KSPDG pursuit-evasion environment whose real-time observations are rendered as the user prompt and into which the perform_action throttles are fed)
- **KSPDG** (Gymnasium/PettingZoo environment suite (Space Gym) that defines the observation format the CoT prompt reasons over and the action interface perform_action targets) — https://github.com/mit-ll/spacegym-kspdg
- **GPT-3.5** (Target LLM prompted via the system/user prompt pair to produce step-by-step reasoning and perform_action function calls)
- **tiktoken** (Tokeniser used to measure prompt size when assembling the few-shot user prompt from telemetry observations) — https://github.com/openai/tiktoken

## Examples

```
assert all(s in open('user_prompt_template.txt').read() for s in ['HUMAN', 'ASSISTANT', 'perform_action(ft, rt, dt)'])
```

## Evaluation signals

- contains_substring check that system_prompt.txt contains 'prograde'
- contains_substring check that system_prompt.txt contains the vessel reference frame axis convention (x axis right, y axis toward target, z axis up)
- contains_substring check that system_prompt.txt contains 'maximum throttle is 1' and 'Reason step-by-step'
- contains_substring check that user_prompt_template.txt contains the HUMAN/ASSISTANT role markers
- contains_substring check that user_prompt_template.txt contains a perform_action call with discrete thrust labels ('forward', 'left', 'up')

## Limitations

- The few-shot agent example is marked as not yet fully documented in the project notes ('[TO BE DONE] under Example: Few-Shot Agent Example')
- The paper's few-shot prompting results are specific to GPT-3.5 evaluated via the Chain-of-Thought approach in KSPDG; transfer to other models or games is unverified
- KSPDG game time does not pause while agents decide, so long CoT prompts impose an inherent real-time penalty on decision latency

## Evidence

- [methods] CoT user prompt concludes with perform_action carrying discrete thrust labels: [evidence span withheld — anchor status: not_found]
- [results] Chain-of-Thought improves generalization of piloting techniques: "The application of the Chain of Thought approach demonstrates a significant improvement in the generalization of spacecraft piloting techniques"
- [abstract] Prompt engineering and few-shot prompting are core strategies: "Our approach leverages prompt engineering, few-shot prompting, and fine-tuning techniques"
