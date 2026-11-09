---
name: llm-conversation-history-padding
description: Use when an LLM agent must emulate multi-turn context across stateless
  OpenAI API calls — e.g. an RL-loop agent where each KSPDG state-action exchange
  must be re-supplied as text in the user prompt.
when_to_use_negative:
- The deployed model was fine-tuned without the windowed variant (e.g. 'fine-tune
  10 files' rather than 'fine-tune 10 files win=3'), so padded history would mismatch
  its training prompt structure.
- The LLM API natively maintains conversation context (chat threads/sessions), making
  manual history re-embedding redundant and token-wasteful.
- Full episode history is required — the sliding window discards everything beyond
  the last n conversations, so long-horizon dependencies cannot be recovered.
edam_topics:
- http://edamontology.org/topic_3376
tools:
- name: GPT-3.5
  role: Target LLM invoked via the OpenAI API, which lacks cross-call context — the
    reason history must be embedded in each user prompt
- name: LLaMA
  role: Fine-tuned model evaluated with the sliding-window prompt variant (Hugging
    Face 'LLaMA 10 files sliding window'; 'fine-tune 10 files win=3')
- name: kspdg agents (ARCLab-MIT)
  role: Agent codebase where conversation history is appended and the sliding-window
    prompt is assembled before each LLM call
  repo: https://github.com/ARCLab-MIT/kspdg/tree/main/arclab_mit/agents
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:4c827dc739a7409456fdcc2795f03d40af7a2da02fd33fba74f8e3b9fc5a2237
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GPT-3.5
  - LLaMA
  - kspdg agents (ARCLab-MIT)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3376
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# llm-conversation-history-padding

## Summary

Reconstructs the sliding-window conversation-history mechanism the KSPDG authors used to emulate context with a stateless OpenAI API: the last n conversations (SLIDING_WINDOW_SIZE, e.g. n=3) are prepended to the user prompt, with zero padding when fewer than n exist, so every RL-step prompt has an identical structure.

## When to use

Use when an LLM agent must emulate multi-turn context across stateless OpenAI API calls — e.g. an RL-loop agent where each KSPDG state-action exchange must be re-supplied as text in the user prompt.

## When NOT to use

- The deployed model was fine-tuned without the windowed variant (e.g. 'fine-tune 10 files' rather than 'fine-tune 10 files win=3'), so padded history would mismatch its training prompt structure.
- The LLM API natively maintains conversation context (chat threads/sessions), making manual history re-embedding redundant and token-wasteful.
- Full episode history is required — the sliding window discards everything beyond the last n conversations, so long-horizon dependencies cannot be recovered.

## Inputs

- Conversation history: list of prior state-action exchanges appended by the KSPDG agent after each LLM call
- Window size parameter SLIDING_WINDOW_SIZE (e.g. n=3 as in the win=3 LLaMA experiment)
- Current user prompt string: real-time mission telemetry observations rendered as text

## Outputs

- Assembled padded prompt string: zero-padded, fixed-structure prefix of the last n conversations prepended to the current user prompt, ready for the LLM API call

## How to apply

Implement a prompt-assembly function that takes the agent's conversation history (the state-action pairs appended after every LLM call) and a SLIDING_WINDOW_SIZE parameter (n=3 in the paper's win=3 LLaMA experiment). Slice the history so only the last n prior conversations are retained, mirroring how the agent adds conversation history when the sliding window is used. When fewer than n prior conversations exist, prepend zero padding so the prompt string always has the same structure regardless of how early in the episode the call occurs. Return the assembled padded prompt string and prepend it to the current user prompt (the textual telemetry observation from the KSPDG environment) before issuing the LLM API call, because the OpenAI API does not maintain context between calls. Validate by running the script on sample histories and checking that the padded prompt strings match reference constructed prompts (output_matches_reference) and that the script executes without errors.

## Related tools

- **GPT-3.5** (Target LLM invoked via the OpenAI API, which lacks cross-call context — the reason history must be embedded in each user prompt)
- **LLaMA** (Fine-tuned model evaluated with the sliding-window prompt variant (Hugging Face 'LLaMA 10 files sliding window'; 'fine-tune 10 files win=3'))
- **kspdg agents (ARCLab-MIT)** (Agent codebase where conversation history is appended and the sliding-window prompt is assembled before each LLM call) — https://github.com/ARCLab-MIT/kspdg/tree/main/arclab_mit/agents

## Examples

```
hist = history[-3:] if len(history) >= 3 else [0]*(3-len(history)) + history; padded_prompt = str(hist) + user_prompt; llm.complete(padded_prompt)  # n=3, cf. 'fine-tune 10 files win=3'
```

## Evaluation signals

- Running the script on sample histories produces padded prompt strings that match reference constructed prompts (output_matches_reference).
- The script executes without errors both on histories shorter than n (zero-padding branch) and longer than n (slicing branch).
- With >= n prior conversations, the prompt contains exactly the last n state-action pairs and no older ones.
- Prompt string structure is invariant across calls made before n conversations accumulate — zero padding keeps the template constant regardless of history length.
- Downstream, the win=3 configuration reproduces the tracked 'fine-tune 10 files win=3' run (23.08 / 40.03 metrics) in wandb logs.

## Limitations

- Only the last n conversations are retained; context beyond the window is discarded, so early-episode information cannot influence later calls.
- Zero padding injects synthetic content early in episodes, shifting the prompt distribution relative to fully populated windows.
- KSPDG's non-blocking step function means game time does not pause while the agent assembles the padded prompt and awaits the LLM API response, penalising slow policies in real time.

## Evidence

- [intro] RL loop with textual user prompts: "we design the classic RL loop by interfacing the simulation environment (KSDPG) with a LLM, transforming the real-time observations (or state) of the mission as textual user prompts"
- [methods] Sliding-window emulation of conversation context: [evidence span withheld — anchor status: not_found]
- [methods] win=3 sliding-window experiment metrics: "fine-tune 10 files win=3 23.08 40.03 0.00%"
- [readme] Non-blocking step function penalises prompt/API latency: [evidence span withheld — anchor status: not_found]
