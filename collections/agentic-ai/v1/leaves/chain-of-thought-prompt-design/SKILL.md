---
name: chain-of-thought-prompt-design
description: Use when an LLM agent must convert real-time telemetry-style observations
  (positions/velocities in the celestial body reference frame) into discrete throttles,
  and the paper's prompt artifacts must be reproduced verbatim for replication.
when_to_use_negative:
- The agent strategy is fine-tuning (e.g. GPT/LLaMA models fine-tuned on navball-bot
  gameplays) rather than prompt-engineering with few-shot examples
- The environment is not KSPDG pursuit-evasion and its observation/action vocabulary
  (throttle labels, vessel reference frame) differs, invalidating the worked CoT example
- Only the evaluation environments of KSPDG are needed (they provide environments,
  not agents, so prompt design is out of scope)
edam_operation: http://edamontology.org/operation_0306
tools:
- name: Kerbal Space Program (KSP)
  role: Game engine hosting the KSPDG pursuit-evasion environment whose observations
    the prompts must encode
  repo: https://www.kerbalspaceprogram.com/
- name: KSPDG
  role: Differential-game environment suite interfaced with the LLM in a classic RL
    loop, transforming real-time observations into textual user prompts
  repo: https://github.com/mit-ll/spacegym-kspdg
- name: GPT-3.5
  role: Target LLM that receives the CoT system/user prompts and emits step-by-step
    reasoning plus perform_action calls
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:601a803182f0a787734e2a6c175d147659d9b00c31e8076cfda8d0b8a8f0b1c1
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
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# chain-of-thought-prompt-design

## Summary

Reconstructs the one-shot Chain-of-Thought prompting artifacts (system prompt and user prompt template) used to elicit step-by-step reasoning and perform_action function calls from an LLM spacecraft operator in the KSPDG pursuit-evasion environment.

## When to use

Use when an LLM agent must convert real-time telemetry-style observations (positions/velocities in the celestial body reference frame) into discrete throttles, and the paper's prompt artifacts must be reproduced verbatim for replication.

## When NOT to use

- The agent strategy is fine-tuning (e.g. GPT/LLaMA models fine-tuned on navball-bot gameplays) rather than prompt-engineering with few-shot examples
- The environment is not KSPDG pursuit-evasion and its observation/action vocabulary (throttle labels, vessel reference frame) differs, invalidating the worked CoT example
- Only the evaluation environments of KSPDG are needed (they provide environments, not agents, so prompt design is out of scope)

## Inputs

- Appendix C prompting section of the paper (system prompt Figure C.8, user prompt Figure C.9)
- KSPDG environment observation vector (pursuer/evader positions and velocities in celestial body reference frame plus prograde direction)
- perform_action(ft, rt, dt) function-call signature and discrete thrust label vocabulary ('forward', 'left', 'up')

## Outputs

- system_prompt.txt (agent role, frame convention, maximum throttle of 1, 'Reason step-by-step' instruction)
- user_prompt_template.txt (one-shot HUMAN/ASSISTANT template with worked CoT example ending in a perform_action call)
- Substring validation report (contains_substring checks over both prompt files)

## How to apply

Read the Appendix C prompting section of the paper, which presents the system prompt (Figure C.8) and the user prompt (Figure C.9) verbatim. Write system_prompt.txt reproducing the system prompt structure: autonomous pursuit-spacecraft agent role, goal of applying throttles to capture the evader, the vessel reference frame convention (x axis right, y axis toward target, z axis up), maximum throttle of 1, and the closing instruction 'Reason step-by-step'. Write user_prompt_template.txt as a one-shot template with HUMAN/ASSISTANT role keywords, a worked Chain-of-Thought example that reasons over the sign of each prograde coordinate (x, y, z) and concludes with the function call format perform_action(ft, rt, dt) using discrete thrust labels ('forward', 'left', 'up'), followed by the same throttle question posed for the current KSPDG environment observations. Rationale: the classic RL loop feeds textual user prompts to the model and plugs the returned action back into KSPDG, so the prompt must fix the frame convention and action vocabulary exactly.

## Related tools

- **Kerbal Space Program (KSP)** (Game engine hosting the KSPDG pursuit-evasion environment whose observations the prompts must encode) — https://www.kerbalspaceprogram.com/
- **KSPDG** (Differential-game environment suite interfaced with the LLM in a classic RL loop, transforming real-time observations into textual user prompts) — https://github.com/mit-ll/spacegym-kspdg
- **GPT-3.5** (Target LLM that receives the CoT system/user prompts and emits step-by-step reasoning plus perform_action calls)

## Examples

```
python -c "s=open('system_prompt.txt').read(); assert 'Reason step-by-step' in s and 'prograde' in s and 'maximum throttle is 1' in s; u=open('user_prompt_template.txt').read(); assert 'perform_action' in u and 'HUMAN' in u and 'ASSISTANT' in u"
```

## Evaluation signals

- contains_substring check that system_prompt.txt contains 'prograde'
- contains_substring check that system_prompt.txt contains the vessel reference frame axis convention (x right, y toward target, z up)
- contains_substring check that system_prompt.txt contains 'maximum throttle is 1' and 'Reason step-by-step'
- contains_substring check that user_prompt_template.txt contains HUMAN/ASSISTANT role markers
- contains_substring check that user_prompt_template.txt contains a perform_action call with discrete thrust labels ('forward', 'left', 'up')

## Limitations

- The paper's repository marks the 'Example: Few-Shot Agent Example' section as '[TO BE DONE]', so full few-shot prompt details may be incomplete in the code
- The KSPDG loop is non-blocking — game time does not pause while the agent decides, so long Chain-of-Thought generations incur an inherent control penalty
- CoT prompting performance is evaluated only within the pursuit-evasion scenarios implemented in KSPDG (pe1, lbg1, sb1); generalization to other orbital domains is unverified

## Evidence

- [methods] Appendix C prompting section: [evidence span withheld — anchor status: not_found]
- [methods] perform_action thrust labels: [evidence span withheld — anchor status: not_found]
- [results] CoT improves generalization: "The application of the Chain of Thought approach demonstrates a significant improvement in the generalization of spacecraft piloting techniques"
- [intro] RL loop prompt construction: "we design the classic RL loop by interfacing the simulation environment (KSDPG) with a LLM, transforming the real-time observations (or state) of the mission as textual user prompts"
- [readme] non-blocking step function: [evidence span withheld — anchor status: not_found]
