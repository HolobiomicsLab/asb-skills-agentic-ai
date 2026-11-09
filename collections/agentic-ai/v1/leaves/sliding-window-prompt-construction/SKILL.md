---
name: sliding-window-prompt-construction
description: Use when calling a stateless LLM API (e.g. OpenAI API) inside a closed-loop
  agent (like the KSPDG RL loop) and prior state-action pairs must be carried into
  each prompt to emulate conversation memory with a fixed window size.
when_to_use_negative:
- When the LLM API natively maintains conversation context across calls (the workaround
  exists because the OpenAI API lacked such support)
- When building single-turn prompts with no need to include prior state-action history
- When training/fine-tuning data files (e.g. the LLaMA 10/25/50-file fine-tuning datasets)
  are being prepared rather than runtime prompts
tools:
- name: GPT-3.5
  role: Target LLM whose stateless API lacks conversation-context support, motivating
    the sliding-window prompt construction
- name: LLaMA
  role: Locally hosted LLM used with the same sliding-window prompt scheme (win=3
    experiment)
- name: KSPDG
  role: Simulation environment whose state-action pairs form the conversation history
    used to fill the sliding window
  repo: https://github.com/mit-ll/spacegym-kspdg/tree main
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:42403049ca9ea1c277e17d216e0d2984967e8f377fff36e79f7b3a3860bc51cb
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
yes_rate: 1.0
n_claims: 13
---

# sliding-window-prompt-construction

## Summary

Emulates multi-turn conversation context for stateless LLM API calls by prepending the last n prior conversations (state-action pairs) to the current user prompt, with zero padding when fewer than n prior conversations exist, as used in the GPT/LLaMA-based KSPDG spacecraft-control agent.

## When to use

Use when calling a stateless LLM API (e.g. OpenAI API) inside a closed-loop agent (like the KSPDG RL loop) and prior state-action pairs must be carried into each prompt to emulate conversation memory with a fixed window size.

## When NOT to use

- When the LLM API natively maintains conversation context across calls (the workaround exists because the OpenAI API lacked such support)
- When building single-turn prompts with no need to include prior state-action history
- When training/fine-tuning data files (e.g. the LLaMA 10/25/50-file fine-tuning datasets) are being prepared rather than runtime prompts

## Inputs

- conversation history: list of prior state-action pairs appended by the agent between LLM calls
- window size parameter (SLIDING_WINDOW_SIZE, e.g. n=3)
- current user prompt string encoding real-time KSPDG telemetry observations

## Outputs

- padded prompt string containing zero-padded last-n conversations, prepended to the current user prompt before the LLM API call

## How to apply

Define a function that takes the conversation history (state-action pairs appended by the agent after each KSPDG step) and a window size parameter (SLIDING_WINDOW_SIZE, e.g. n=3 as in the 'fine-tune 10 files win=3' LLaMA experiment). Slice the history to retain only the last n prior conversations, as done when the agent adds conversation history if sliding window is used. When fewer than n prior conversations exist, prepend zero padding so the assembled prompt string always has the same structure regardless of history length — this matters because KSPDG is a non-blocking environment where game time does not pause while agents make decisions, so prompt construction must be deterministic and fast. Return the padded prompt string, prepend it to the current user prompt (built from real-time telemetry observations), and send it in the LLM API call.

## Related tools

- **GPT-3.5** (Target LLM whose stateless API lacks conversation-context support, motivating the sliding-window prompt construction)
- **LLaMA** (Locally hosted LLM used with the same sliding-window prompt scheme (win=3 experiment))
- **KSPDG** (Simulation environment whose state-action pairs form the conversation history used to fill the sliding window) — https://github.com/mit-ll/spacegym-kspdg/tree main

## Examples

```
padded = build_window_prompt(agent_history, SLIDING_WINDOW_SIZE=3); llm_input = padded + current_user_prompt
```

## Evaluation signals

- Run the script on sample histories and check that produced padded prompt strings match reference constructed prompts (output_matches_reference)
- Verify the script executes without errors on empty, short (<n), and long (>n) conversation histories
- Confirm every assembled prompt string has the same structural length/format regardless of how many prior conversations exist (zero padding invariant)
- Check the window slices exactly the last n prior state-action pairs from the history (e.g. n=3)

## Limitations

- The fixed-size padded prompt truncates context to the last n conversations, so older state-action pairs are discarded
- KSPDG's non-blocking step function means game time does not pause during prompt construction/LLM calls, penalizing slow policies
- Outcome is hardware-dependent for locally hosted LLaMA models; the article notes results 'are highly hardware-dependent'

## Evidence

- [methods] finding: [evidence span withheld — anchor status: not_found]
- [results] filter_step win=3: "fine-tune 10 files win=3 23.08 40.03 0.00%"
- [readme] non-blocking rationale: [evidence span withheld — anchor status: not_found]
- [intro] RL loop interface: "we design the classic RL loop by interfacing the simulation environment (KSDPG) with a LLM, transforming the real-time observations (or state) of the mission as textual user prompts"
