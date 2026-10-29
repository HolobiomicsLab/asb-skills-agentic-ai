---
name: vessel-reference-frame-convention
description: 'Use when writing or validating prompts for an LLM spacecraft operator
  in KSPDG: raw positions/velocities in the celestial body reference frame must be
  re-expressed in the vessel frame (x right, y toward target, z up) before selecting
  thrust directions.'
when_to_use_negative:
- The agent is a fine-tuned LLaMA or navball-bot policy that consumes numeric observation
  vectors directly rather than natural-language prompts
- The environment is not the KSPDG pursuit-evasion setting, so the x-right/y-toward-target/z-up
  vessel frame and prograde-direction base frame do not apply
- Continuous thrust magnitudes are required; the convention here maps reasoning to
  discrete thrust labels ('forward', 'left', 'up') with maximum throttle of 1
edam_operation: http://edamontology.org/operation_0306
tools:
- name: Kerbal Space Program (KSP)
  role: Game engine whose KSPDG pursuit-evasion environments supply the observations
    that must be re-expressed in the vessel reference frame
  repo: https://github.com/mit-ll/spacegym-kspdg/tree main
- name: KSPDG
  role: Gymnasium/PettingZoo environment suite providing the observation and action
    interface the prompts are written against
  repo: https://github.com/mit-ll/spacegym-kspdg
- name: GPT-3.5
  role: LLM that receives the frame-convention system prompt and one-shot CoT user
    prompt and emits perform_action calls
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:f7b53dfc74dc136d410636978a0c7d0a4376862ee9a761bf824f83c4a98f7eda
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GPT-3.5
  - KSPDG
  - Kerbal Space Program (KSP)
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# vessel-reference-frame-convention

## Summary

Encodes the KSPDG pursuit-evasion vessel reference frame convention (x axis right, y axis toward target, z axis up, prograde-direction base frame, maximum throttle 1) into natural-language prompts so an LLM agent can reason over telemetry signs and emit discrete perform_action throttles.

## When to use

Use when writing or validating prompts for an LLM spacecraft operator in KSPDG: raw positions/velocities in the celestial body reference frame must be re-expressed in the vessel frame (x right, y toward target, z up) before selecting thrust directions.

## When NOT to use

- The agent is a fine-tuned LLaMA or navball-bot policy that consumes numeric observation vectors directly rather than natural-language prompts
- The environment is not the KSPDG pursuit-evasion setting, so the x-right/y-toward-target/z-up vessel frame and prograde-direction base frame do not apply
- Continuous thrust magnitudes are required; the convention here maps reasoning to discrete thrust labels ('forward', 'left', 'up') with maximum throttle of 1

## Inputs

- KSPDG environment observations (pursuer/evader positions and velocities in the celestial body reference frame plus prograde direction)
- System prompt text specifying the vessel reference frame axis convention and maximum throttle of 1
- One-shot Chain-of-Thought user prompt template with HUMAN/ASSISTANT role markers and a perform_action(ft, rt, dt) worked example

## Outputs

- system_prompt.txt reproducing the agent role, frame convention, throttle limit, and 'Reason step-by-step' instruction
- user_prompt_template.txt containing the worked CoT example concluding in a perform_action call with discrete thrust labels
- Natural-language step-by-step reasoning plus a perform_action function call (discrete throttle labels) returned by the GPT agent

## How to apply

Write a system prompt that casts the model as an autonomous pursuit-spacecraft agent whose goal is to apply throttles to capture the evader, defines the vessel reference frame convention (x axis right, y axis toward target, z axis up) relative to the prograde direction, states that the maximum throttle is 1, and closes with 'Reason step-by-step'. Build a one-shot Chain-of-Thought user prompt template with HUMAN/ASSISTANT markers whose worked example reasons over the sign of each prograde coordinate (x, y, z) and ends with a perform_action(ft, rt, dt) call using discrete thrust labels such as 'forward', 'left', 'up'; the template then presents current KSPDG environment observations and asks the same throttle question. The rationale is that the LLM is interfaced to KSPDG in a classic RL loop where real-time observations are rendered as textual user prompts, so an explicit frame convention is what makes sign-based step-by-step reasoning and action selection well-defined.

## Related tools

- **Kerbal Space Program (KSP)** (Game engine whose KSPDG pursuit-evasion environments supply the observations that must be re-expressed in the vessel reference frame) — https://github.com/mit-ll/spacegym-kspdg/tree main
- **KSPDG** (Gymnasium/PettingZoo environment suite providing the observation and action interface the prompts are written against) — https://github.com/mit-ll/spacegym-kspdg
- **GPT-3.5** (LLM that receives the frame-convention system prompt and one-shot CoT user prompt and emits perform_action calls)

## Examples

```
python -c "s=open('system_prompt.txt').read(); u=open('user_prompt_template.txt').read(); assert 'Reason step-by-step' in s and 'maximum throttle is 1' in s and 'perform_action' in u"
```

## Evaluation signals

- Substring check that system_prompt.txt contains 'prograde', the x/y/z axis convention text, 'maximum throttle is 1', and 'Reason step-by-step'
- Substring check that user_prompt_template.txt contains the HUMAN/ASSISTANT markers and at least one perform_action call
- The worked CoT example reasons over the sign of each prograde coordinate (x, y, z) rather than jumping straight to an action
- The model's emitted action is a perform_action call restricted to discrete thrust labels (e.g. 'forward', 'left', 'up') within the maximum throttle of 1
- Applying the prompted frame convention improves capture performance in the KSPDG pursuit-evasion evaluation, consistent with the reported Chain-of-Thought generalization gain

## Limitations

- The full prompting appendix example was marked '[TO BE DONE] under Example: Few-Shot Agent Example' in one section, so the exact verbatim prompt text must be recovered from Appendix C figures (system prompt Figure C.8, user prompt Figure C.9)
- KSPDG game time does not pause while agents decide, so long step-by-step reasoning is penalized by the non-blocking step function
- The convention is specific to the KSPDG pursuit-evasion observation format and does not transfer to the lady-bandit-guard or sun-blocking scenario groups without adaptation

## Evidence

- [intro] RL loop textual observation interface: "we design the classic RL loop by interfacing the simulation environment (KSDPG) with a LLM, transforming the real-time observations (or state) of the mission as textual user prompts"
- [methods] CoT worked example ends in perform_action: [evidence span withheld — anchor status: not_found]
- [results] Chain-of-Thought improves generalization: "The application of the Chain of Thought approach demonstrates a significant improvement in the generalization of spacecraft piloting techniques"
- [readme] Non-blocking step function penalizes slow policies: [evidence span withheld — anchor status: not_found]
