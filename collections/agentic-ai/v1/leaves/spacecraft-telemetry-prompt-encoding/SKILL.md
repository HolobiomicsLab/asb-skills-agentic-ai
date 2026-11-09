---
name: spacecraft-telemetry-prompt-encoding
description: Use when an LLM agent must control a KSPDG pursuit-evasion spacecraft
  from real-time telemetry, and raw numeric observations need conversion into natural-language
  user prompts plus a system prompt ending in 'Reason step-by-step'.
when_to_use_negative:
- The goal is to fine-tune a model (e.g. GPT via OpenAI API or LLaMA 10/25/50-file
  datasets) rather than elicit in-context reasoning — use the fine-tuning pipeline
  instead
- The environment is not KSPDG/Gymnasium-based (e.g. headless massively parallel RL
  training, which the KSPDG README explicitly does not support)
- Observations are already natural-language mission logs, so numeric telemetry-to-text
  serialization would add nothing
edam_operation: http://edamontology.org/operation_0306
tools:
- name: KSPDG
  role: Supplies the differential-game pursuit-evasion environments whose real-time
    observations are serialized into textual user prompts
  repo: https://github.com/mit-ll/spacegym-kspdg/tree main
- name: Kerbal Space Program (KSP)
  role: Game engine running the pursuit-evasion mission that the LLM agent observes
    and acts upon via the encoded prompts
  repo: https://www.kerbalspaceprogram.com/
- name: tiktoken
  role: BPE tokeniser used to measure/check prompt token counts for the GPT models
  repo: https://github.com/openai/tiktoken
- name: GPT-3.5
  role: Target LLM that receives the encoded telemetry prompts and emits perform_action
    throttle calls
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:4cfd99e73d53bcdfd6d5536edabd0c25398635048eebf5da2bb5867611e701f2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GPT-3.5
  - KSPDG
  - Kerbal Space Program (KSP)
  - tiktoken
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# spacecraft-telemetry-prompt-encoding

## Summary

Encodes real-time KSPDG spacecraft telemetry (pursuer/evader positions and velocities in the celestial-body reference frame plus prograde direction) into textual user prompts and a fixed system prompt so a GPT-3.5/LLaMA agent can output throttle actions as perform_action(ft, rt, dt) calls, closing an RL loop with the KSP simulation.

## When to use

Use when an LLM agent must control a KSPDG pursuit-evasion spacecraft from real-time telemetry, and raw numeric observations need conversion into natural-language user prompts plus a system prompt ending in 'Reason step-by-step'.

## When NOT to use

- The goal is to fine-tune a model (e.g. GPT via OpenAI API or LLaMA 10/25/50-file datasets) rather than elicit in-context reasoning — use the fine-tuning pipeline instead
- The environment is not KSPDG/Gymnasium-based (e.g. headless massively parallel RL training, which the KSPDG README explicitly does not support)
- Observations are already natural-language mission logs, so numeric telemetry-to-text serialization would add nothing

## Inputs

- KSPDG environment observation dict/vector (real-time telemetry: positions and velocities in the celestial body reference frame plus prograde direction)
- System prompt specification (agent role, vessel reference frame convention x/y/z, maximum throttle of 1)
- One-shot Chain-of-Thought worked example with HUMAN/ASSISTANT role keywords and perform_action(ft, rt, dt) call format

## Outputs

- system_prompt.txt (system prompt: agent role, frame conventions, throttle limit, 'Reason step-by-step')
- user_prompt_template.txt (one-shot CoT template with HUMAN/ASSISTANT markers and perform_action call)
- Per-step textual user prompt consumed by the LLM agent
- Discrete throttle action (perform_action(ft, rt, dt)) returned to the KSPDG environment

## How to apply

Author a system_prompt.txt defining the autonomous pursuit-spacecraft agent role, the goal of applying throttles to capture the evader from positions/velocities in the celestial body reference frame plus prograde direction, the vessel reference frame convention (x axis right, y axis toward target, z axis up), and a maximum throttle of 1, closing with 'Reason step-by-step'. Author a one-shot user_prompt_template.txt using HUMAN/ASSISTANT role markers and a worked Chain-of-Thought example that reasons over the sign of each prograde coordinate (x, y, z) and concludes with a perform_action(ft, rt, dt) function call carrying discrete thrust labels such as 'forward', 'left', 'up'. At each environment step, serialize the current KSPDG observations (relative positions and velocities) into the template as a user prompt, send it to the LLM, and plug the returned action back into KSPDG. Because KSPDG game time does not pause during agent decisions, prompt construction must be fast enough to avoid losing the pursuit; tokenization of long prompts can be checked with tiktoken. Validate the prompt artifacts with substring checks rather than eyeballing.

## Related tools

- **KSPDG** (Supplies the differential-game pursuit-evasion environments whose real-time observations are serialized into textual user prompts) — https://github.com/mit-ll/spacegym-kspdg/tree main
- **Kerbal Space Program (KSP)** (Game engine running the pursuit-evasion mission that the LLM agent observes and acts upon via the encoded prompts) — https://www.kerbalspaceprogram.com/
- **tiktoken** (BPE tokeniser used to measure/check prompt token counts for the GPT models) — https://github.com/openai/tiktoken
- **GPT-3.5** (Target LLM that receives the encoded telemetry prompts and emits perform_action throttle calls)

## Examples

```
assert 'maximum throttle is 1' in open('system_prompt.txt').read() and 'Reason step-by-step' in open('system_prompt.txt').read() and 'perform_action' in open('user_prompt_template.txt').read()
```

## Evaluation signals

- system_prompt.txt contains 'prograde'
- system_prompt.txt contains the vessel reference frame axis convention (x axis right, y axis toward target, z axis up)
- system_prompt.txt contains 'maximum throttle is 1' and 'Reason step-by-step'
- user_prompt_template.txt contains HUMAN/ASSISTANT role markers
- user_prompt_template.txt contains a perform_action(ft, rt, dt) call with discrete thrust labels such as 'forward', 'left', 'up'

## Limitations

- The few-shot agent example was marked as not yet documented in the paper's repo ('[TO BE DONE] under 'Example: Few-Shot Agent Example'')
- KSPDG game time does not pause while agents make decisions, so slow prompt construction/LLM inference incurs an inherent penalty
- The Chain-of-Thought prompt-engineering agent was evaluated only for the GPT model; results generalization to LLaMA variants via this prompt scheme is not established
- Fine-tuning hyperparameters such as learning rate, epochs, and LoRA rank are not reported, limiting reproduction of the comparison branch

## Evidence

- [intro] RL loop design: "we design the classic RL loop by interfacing the simulation environment (KSDPG) with a LLM, transforming the real-time observations (or state) of the mission as textual user prompts"
- [methods] system prompt structure: [evidence span withheld — anchor status: not_found]
- [methods] CoT template and function call: [evidence span withheld — anchor status: not_found]
- [supplementary] worked example conclusion: [evidence span withheld — anchor status: not_found]
- [readme] non-blocking step function: [evidence span withheld — anchor status: not_found]
