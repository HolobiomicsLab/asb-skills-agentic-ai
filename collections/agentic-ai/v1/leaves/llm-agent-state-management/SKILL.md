---
name: llm-agent-state-management
description: Use when an agent must maintain conversational context across repeated
  stateless LLM API calls (e.g. GPT-3.5 or LLaMA acting as a spacecraft operator in
  the KSPDG RL loop) and history must be injected into a fixed-structure prompt.
when_to_use_negative:
- The API/model already maintains multi-turn context natively (the mechanism exists
  specifically to work around the OpenAI API's lack of context between calls)
- Fine-tuned models trained without the sliding-window dataset variant (e.g. the plain
  fine-tune 10/25/50-file LLaMA models) — prepending padded history would mismatch
  the training prompt structure
- Single-shot or few-shot prompting where conversation history is irrelevant to the
  action decision
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: GPT-3.5 (OpenAI API)
  role: Stateless LLM whose API lacks cross-call context, motivating the sliding-window
    padded prompt
- name: 'Hugging Face: LLaMA 10 files sliding window'
  role: Fine-tuned LLaMA model trained with the sliding-window (win=3) prompt structure
    that this mechanism must reproduce at inference
- name: openai/tiktoken
  role: Token counting for OpenAI models, used in methods to manage prompt sizing
  repo: https://github.com/openai/tiktoken
- name: kspdg
  role: Environment whose agent builds the padded prompt from its state-action history
    each RL step
  repo: https://github.com/ARCLab-MIT/kspdg
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:ec657876f1066f83af5dc86df8908adfc56cbc8482727ab9ea2e63c9fa4c3b23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/openai/tiktoken
  tools:
  - GPT-3.5 (OpenAI API)
  - 'Hugging Face: LLaMA 10 files sliding window'
  - kspdg
  - openai/tiktoken
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# llm-agent-state-management

## Summary

Reconstructs the sliding-window conversation-history mechanism used in the KSPDG LLM agent: to emulate multi-turn context over a stateless OpenAI API, the user prompt is prepended with the last n prior state-action conversations (SLIDING_WINDOW_SIZE, e.g. n=3), zero-padded when fewer than n exist.

## When to use

Use when an agent must maintain conversational context across repeated stateless LLM API calls (e.g. GPT-3.5 or LLaMA acting as a spacecraft operator in the KSPDG RL loop) and history must be injected into a fixed-structure prompt.

## When NOT to use

- The API/model already maintains multi-turn context natively (the mechanism exists specifically to work around the OpenAI API's lack of context between calls)
- Fine-tuned models trained without the sliding-window dataset variant (e.g. the plain fine-tune 10/25/50-file LLaMA models) — prepending padded history would mismatch the training prompt structure
- Single-shot or few-shot prompting where conversation history is irrelevant to the action decision

## Inputs

- Conversation history of state-action pairs appended by the agent (list of prior LLM exchanges)
- Sliding window size parameter (SLIDING_WINDOW_SIZE, e.g. n=3 as in the win=3 LLaMA experiment)
- Current user prompt encoding real-time KSPDG mission telemetry (position, velocity, relative state text)

## Outputs

- Assembled padded prompt string with the last n conversations (zero-padded) prepended, ready for the LLM API call

## How to apply

Define a function taking the conversation history (state-action pairs appended by the agent) and a window size parameter (SLIDING_WINDOW_SIZE, e.g. n=3 as in the win=3 LLaMA fine-tune). Slice the history to retain only the last n prior conversations, as done when the agent adds conversation history if the sliding window is used. When fewer than n prior conversations exist, prepend zero padding so the prompt string always has the same structure regardless of history length. Return the assembled padded prompt string and prepend it to the current user prompt (which encodes KSPDG telemetry as text) before the LLM API call. The rationale is that the OpenAI API does not maintain context between calls, so context must be explicitly re-sent each step; in KSPDG, game time does not pause while agents decide, so the window must stay small to keep inference fast.

## Related tools

- **GPT-3.5 (OpenAI API)** (Stateless LLM whose API lacks cross-call context, motivating the sliding-window padded prompt)
- **Hugging Face: LLaMA 10 files sliding window** (Fine-tuned LLaMA model trained with the sliding-window (win=3) prompt structure that this mechanism must reproduce at inference)
- **openai/tiktoken** (Token counting for OpenAI models, used in methods to manage prompt sizing) — https://github.com/openai/tiktoken
- **kspdg** (Environment whose agent builds the padded prompt from its state-action history each RL step) — https://github.com/ARCLab-MIT/kspdg

## Examples

```
history_window = state_action_history[-SLIDING_WINDOW_SIZE:]; prompt = zero_pad(history_window, SLIDING_WINDOW_SIZE) + user_prompt  # e.g. SLIDING_WINDOW_SIZE=3 as in the win=3 LLaMA experiment
```

## Evaluation signals

- Padded prompt strings produced from sample histories match reference constructed prompts (output_matches_reference)
- With fewer than n prior conversations, the prompt still has the same structure/length via zero padding
- With more than n prior conversations, only the last n state-action pairs appear in the prompt
- The script executes without errors on edge cases (empty history, exactly n, more than n exchanges)
- Prepending the padded prompt to the telemetry user prompt yields a valid API-call payload for GPT-3.5 or the win=3 LLaMA model

## Limitations

- Performance is hardware-dependent; larger history windows increase prompt length and inference latency in a non-blocking environment
- Prompt-structure mismatch: models fine-tuned without the sliding-window variant (plain fine-tune 10/25/50 files) may not benefit from padded history injection
- Effectiveness measured on KSPDG evaluation scenarios only (e.g. fine-tune 10 files win=3: 23.08 pursuit score), which are non-trainable test environments

## Evidence

- [methods] sliding-window zero padding: [evidence span withheld — anchor status: not_found]
- [intro] RL loop prompting: "we design the classic RL loop by interfacing the simulation environment (KSDPG) with a LLM, transforming the real-time observations (or state) of the mission as textual user prompts"
- [methods] win=3 filter step: "fine-tune 10 files win=3 23.08 40.03 0.00%"
- [readme] non-blocking decisions: [evidence span withheld — anchor status: not_found]
