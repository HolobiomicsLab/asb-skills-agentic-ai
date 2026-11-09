---
name: spacecraft-agent-prompt-formatting
description: Use when building an LLM agent that must appear stateful across sequential
  KSPDG/RL steps via a stateless completion API, i.e. when each observation must be
  turned into a textual user prompt that carries prior state-action pairs.
when_to_use_negative:
- When using a locally hosted LLaMA model or an API that natively maintains conversation
  context — the padding hack emulates context the backend may already provide
- When the environment pauses during agent decisions (unlike KSPDG's non-blocking
  step function), where variable-length prompts carry no timing penalty
- When evaluating the standard GPT agent without a sliding window (win absent from
  the experiment row), since history prepending changes the prompt format the model
  was fine-tuned on
tools:
- name: GPT-3.5
  role: Target LLM whose stateless API motivates prepending padded conversation history
    to each user prompt
- name: LLaMA
  role: Fine-tuned model evaluated with the sliding window (win=3) variant on the
    10-file dataset
- name: 'Hugging Face: LLaMA 10 files sliding window'
  role: Hosted artifact of the sliding-window (win=3) fine-tuned LLaMA model associated
    with this mechanism
- name: KSPDG
  role: Environment whose real-time observations are serialized into the textual user
    prompts that the padded history is prepended to
  repo: https://github.com/mit-ll/spacegym-kspdg/tree main
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:c6a6c74aae4e314f45a771599609ab9a92b400ec47c9de53ab524d093bfcd8a2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GPT-3.5
  - 'Hugging Face: LLaMA 10 files sliding window'
  - KSPDG
  - LLaMA
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# spacecraft-agent-prompt-formatting

## Summary

Reconstructs the sliding-window conversation-history mechanism used by the LLM-based KSPDG spacecraft operator: the user prompt is prepended with the last n prior state-action conversations, with zero padding when fewer than n exist, emulating multi-turn context over the stateless OpenAI API.

## When to use

Use when building an LLM agent that must appear stateful across sequential KSPDG/RL steps via a stateless completion API, i.e. when each observation must be turned into a textual user prompt that carries prior state-action pairs.

## When NOT to use

- When using a locally hosted LLaMA model or an API that natively maintains conversation context — the padding hack emulates context the backend may already provide
- When the environment pauses during agent decisions (unlike KSPDG's non-blocking step function), where variable-length prompts carry no timing penalty
- When evaluating the standard GPT agent without a sliding window (win absent from the experiment row), since history prepending changes the prompt format the model was fine-tuned on

## Inputs

- Conversation history of prior state-action pairs appended by the agent
- Integer window size (SLIDING_WINDOW_SIZE, e.g. n=3)
- Current KSPDG telemetry observation serialized as a textual user prompt

## Outputs

- Zero-padded prompt string containing the last n conversations, prepended to the current user prompt and sent to the GPT/LLaMA model
- Model action parsed back into the KSPDG environment

## How to apply

Define a function that takes the conversation history (state-action pairs appended by the agent each step) and a window size parameter SLIDING_WINDOW_SIZE (e.g. n=3, as in the win=3 LLaMA experiment). Slice the history to retain only the last n prior conversations, mirroring how the agent adds conversation history when a sliding window is used. When fewer than n prior conversations exist, prepend zero padding so the prompt string always has the same structure regardless of history length; this matters because KSPDG's non-blocking step function penalizes slow policies, so prompt assembly must be deterministic and cheap. Return the assembled padded prompt string and prepend it to the current user prompt (built from real-time KSPDG telemetry observations) before the LLM API call. The win=3 variant of the fine-tuned 10-file LLaMA model (23.08/40.03 reward metrics) shows the window mechanism was exercised in evaluation.

## Related tools

- **GPT-3.5** (Target LLM whose stateless API motivates prepending padded conversation history to each user prompt)
- **LLaMA** (Fine-tuned model evaluated with the sliding window (win=3) variant on the 10-file dataset)
- **Hugging Face: LLaMA 10 files sliding window** (Hosted artifact of the sliding-window (win=3) fine-tuned LLaMA model associated with this mechanism)
- **KSPDG** (Environment whose real-time observations are serialized into the textual user prompts that the padded history is prepended to) — https://github.com/mit-ll/spacegym-kspdg/tree main

## Examples

```
def build_prompt(history, SLIDING_WINDOW_SIZE=3): past = history[-SLIDING_WINDOW_SIZE:]; return zero_pad(past, SLIDING_WINDOW_SIZE) + current_user_prompt
```

## Evaluation signals

- Padded prompt strings produced on sample histories match reference-constructed prompts (output_matches_reference)
- With fewer than n prior conversations, zero padding is present and the prompt string length/structure is identical to a full-history prompt
- With more than n prior conversations, the prompt contains exactly the last n state-action pairs, no older entries
- Script executes without errors across history lengths 0 through n and beyond
- End-to-end: the agent's action is accepted by the KSPDG environment step function

## Limitations

- Only the last n conversations are retained, so context older than the window is silently dropped and may degrade long-horizon behavior
- KSPDG game time does not pause while agents decide, so slow prompt assembly/LLM calls carry an inherent performance penalty
- Fine-tuning hyperparameters and hardware for local LLaMA inference are not reported, making exact reproduction of win=3 results hardware-dependent

## Evidence

- [methods] finding: sliding-window context emulation: [evidence span withheld — anchor status: not_found]
- [intro] RL loop prompt construction: "we design the classic RL loop by interfacing the simulation environment (KSDPG) with a LLM, transforming the real-time observations (or state) of the mission as textual user prompts"
- [methods] win=3 experiment row: "fine-tune 10 files win=3 23.08 40.03 0.00%"
- [readme] non-blocking step rationale: [evidence span withheld — anchor status: not_found]
