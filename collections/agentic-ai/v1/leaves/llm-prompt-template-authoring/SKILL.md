---
name: llm-prompt-template-authoring
description: Use when an LLM agent must consume real-time simulation telemetry (positions/velocities
  in celestial-body and vessel reference frames) as text and output structured throttle
  actions for a pursuit-evasion loop like KSPDG.
when_to_use_negative:
- The agent is a fine-tuned model (GPT fine-tuned with system prompt or LLaMA on navball-bot
  datasets) rather than a prompted few-shot agent — the paper compares these as alternative
  strategies
- The environment is a trainable RL environment; KSPDG provides evaluation environments
  only ('NOT training environments for reinforcement learning algorithms')
- The action space is already a structured vector consumed directly by a control baseline
  such as Lambert-MPC or the Navball bot, with no natural-language interface needed
tools:
- name: GPT-3.5
  role: Target LLM that receives the system prompt and one-shot CoT user prompt and
    emits perform_action function calls
- name: Kerbal Space Program (KSP)
  role: Simulation engine whose real-time KSPDG observations are serialized into the
    textual user prompts
- name: KSPDG
  role: Differential-game (pursuit-evasion) environment suite whose observation format
    dictates the prompt template's throttle question
  repo: https://github.com/mit-ll/spacegym-kspdg/tree main
- name: tiktoken
  role: Tokeniser for counting prompt-template tokens against LLM context limits
  repo: https://github.com/openai/tiktoken
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:ea343fe6b0ef940b958c664fd78c399af09448e9d8126e366a021bba85d6ced9
schema_version: 0.2.0
metadata:
  license_tier: open
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
yes_rate: 0.8333333333333334
n_claims: 6
---

# LLM Prompt Template Authoring

## Summary

Author a system prompt and a one-shot Chain-of-Thought user prompt template that turn real-time KSPDG telemetry into natural-language instructions for an LLM spacecraft operator, eliciting step-by-step reasoning and perform_action function calls.

## When to use

Use when an LLM agent must consume real-time simulation telemetry (positions/velocities in celestial-body and vessel reference frames) as text and output structured throttle actions for a pursuit-evasion loop like KSPDG.

## When NOT to use

- The agent is a fine-tuned model (GPT fine-tuned with system prompt or LLaMA on navball-bot datasets) rather than a prompted few-shot agent — the paper compares these as alternative strategies
- The environment is a trainable RL environment; KSPDG provides evaluation environments only ('NOT training environments for reinforcement learning algorithms')
- The action space is already a structured vector consumed directly by a control baseline such as Lambert-MPC or the Navball bot, with no natural-language interface needed

## Inputs

- KSPDG environment observation state (vessel and target positions/velocities in celestial body reference frame plus prograde direction)
- Vessel reference frame axis convention (x right, y toward target, z up)
- Action schema perform_action(ft, rt, dt) with maximum throttle of 1

## Outputs

- system_prompt.txt (agent role, reference-frame conventions, throttle limit, 'Reason step-by-step' instruction)
- user_prompt_template.txt (one-shot HUMAN/ASSISTANT Chain-of-Thought example ending in a perform_action call)

## How to apply

Write system_prompt.txt establishing the autonomous pursuit-spacecraft agent role: capture the evader using throttles derived from positions/velocities in the celestial body reference frame plus prograde direction, the vessel reference frame convention (x axis right, y axis toward target, z axis up), a maximum throttle of 1, and the closing instruction 'Reason step-by-step'. Write user_prompt_template.txt as a one-shot template with HUMAN/ASSISTANT role keywords: the worked Chain-of-Thought example reasons over the sign of each prograde coordinate (x, y, z) and concludes with a perform_action(ft, rt, dt) call carrying discrete thrust labels ('forward', 'left', 'up'), after which the model is asked the same throttle question given current KSPDG environment observations. The rationale is the classic RL-loop interface: observations are serialized to textual user prompts and the model's action is plugged back into KSPDG; the paper reports that applying the Chain of Thought approach significantly improved generalization of spacecraft piloting techniques. Validate the artifacts with substring checks before use.

## Related tools

- **GPT-3.5** (Target LLM that receives the system prompt and one-shot CoT user prompt and emits perform_action function calls)
- **Kerbal Space Program (KSP)** (Simulation engine whose real-time KSPDG observations are serialized into the textual user prompts)
- **KSPDG** (Differential-game (pursuit-evasion) environment suite whose observation format dictates the prompt template's throttle question) — https://github.com/mit-ll/spacegym-kspdg/tree main
- **tiktoken** (Tokeniser for counting prompt-template tokens against LLM context limits) — https://github.com/openai/tiktoken

## Examples

```
assert 'Reason step-by-step' in open('system_prompt.txt').read() and 'perform_action' in open('user_prompt_template.txt').read()
```

## Evaluation signals

- contains_substring check that system_prompt.txt contains 'prograde'
- contains_substring check that system_prompt.txt contains the vessel reference frame axis convention (x right, y toward target, z up)
- contains_substring check that system_prompt.txt contains 'maximum throttle is 1' and 'Reason step-by-step'
- contains_substring check that user_prompt_template.txt contains HUMAN/ASSISTANT role markers
- contains_substring check that user_prompt_template.txt contains a perform_action call, with the worked example concluding in discrete thrust labels ('forward', 'left', 'up')

## Limitations

- KSPDG's non-blocking step function means game time does not pause while the agent decides, so long prompt templates and slow LLM inference carry an inherent performance penalty
- The paper's repository marks the few-shot agent example as '[TO BE DONE]', so the exact verbatim prompt figures must be reconstructed from the paper's Appendix C prompting section
- Prompt-engineering is compared against fine-tuning (GPT via OpenAI API with LRM adjustments; LLaMA on 10/25/50 navball-bot files); prompting may underperform fine-tuned models on this task

## Evidence

- [abstract] Our approach leverages prompt engineering, few-shot prompting, and fine-tuning techniques: "Our approach leverages prompt engineering, few-shot prompting, and fine-tuning techniques"
- [intro] RL loop interfacing KSPDG with LLM via textual prompts: "we design the classic RL loop by interfacing the simulation environment (KSDPG) with a LLM, transforming the real-time observations (or state) of the mission as textual user prompts"
- [results] CoT improves generalization of piloting: "The application of the Chain of Thought approach demonstrates a significant improvement in the generalization of spacecraft piloting techniques"
- [methods] One-shot CoT prompt ends in perform_action call: [evidence span withheld — anchor status: not_found]
- [readme] KSPDG is evaluation environments, not RL training: [evidence span withheld — anchor status: not_found]
