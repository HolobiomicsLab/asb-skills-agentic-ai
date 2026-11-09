---
name: multi-turn-message-formatting
description: Use when a multi-turn message list must be serialized into a token-level
  ChatML chat-template sequence for instruction tuning, or when a generation prompt
  must end exactly at an opened assistant turn.
when_to_use_negative:
- Input is a single-turn prompt-completion pair (e.g. Stanford Alpaca-style '### Instruction:/###
  Response:' templates) rather than a multi-turn role/content conversation
- The target model uses a non-ChatML chat template (different special-token delimiters);
  do not blindly apply <|im_start|>/<|im_end|>
- You need token IDs from a specific tokenizer, not the template string — this skill
  renders markup, it does not tokenize to vocabulary IDs
tools:
- name: Python
  role: Implements the ChatML renderer that wraps each message in <|im_start|>{role}\n{content}<|im_end|>\n
    markup and validates role ordering before rendering
  repo: https://github.com/natolambert/rlhf-book
- name: TRL
  role: Post-training framework whose SFT trainers consume chat-template-rendered
    multi-turn conversations for instruction tuning
  repo: https://github.com/huggingface/trl
- name: PyTorch
  role: Tensor backend for downstream fine-tuning of the rendered token sequences
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:cf5ac4217e798ffdfbb6569e0010a8a0ee275a926f14f7f656f98bde359b3911
  source_papers:
  - doi: 10.48550/arxiv.2504.12501
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/natolambert/rlhf-book
  tools:
  - PyTorch
  - Python
  - TRL
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# multi-turn-message-formatting

## Summary

Renders a multi-turn conversation (system + alternating user/assistant messages) into a single flat ChatML token sequence, '<|im_start|>' + role + '\n' + content + '<|im_end|>' + '\n', as specified in the RLHF Book's instruction-tuning (SFT) stage before preference tuning or RL.

## When to use

Use when a multi-turn message list must be serialized into a token-level ChatML chat-template sequence for instruction tuning, or when a generation prompt must end exactly at an opened assistant turn.

## When NOT to use

- Input is a single-turn prompt-completion pair (e.g. Stanford Alpaca-style '### Instruction:/### Response:' templates) rather than a multi-turn role/content conversation
- The target model uses a non-ChatML chat template (different special-token delimiters); do not blindly apply <|im_start|>/<|im_end|>
- You need token IDs from a specific tokenizer, not the template string — this skill renders markup, it does not tokenize to vocabulary IDs

## Inputs

- Multi-turn message list of role/content dictionaries (system message first, user/assistant turns alternating)
- Boolean flag for add_generation_prompt (append '<|im_start|>assistant\n' without closing token)
- Target output file path, e.g. rendered_chat_template.txt

## Outputs

- rendered_chat_template.txt containing the full-conversation ChatML token sequence and a separate generation-prompt render
- Exact flat token sequence strings with <|im_start|>/<|im_end|> delimiters around every message

## How to apply

Define the conversation as a Python list of role/content dictionaries following the book's post-training convention: a system message first, then strictly alternating user and assistant turns, validating that order before rendering. Wrap every message in ChatML special tokens — '<|im_start|>' + role + '\n' + trimmed content + '<|im_end|>' + '\n' — so the whole conversation becomes one flat token sequence string. For the add_generation_prompt variant, append a trailing '<|im_start|>assistant\n' opener with no closing '<|im_end|>' so the model continues generating the assistant turn. Write the complete-conversation render and the generation-prompt render as clearly separated outputs (e.g. to rendered_chat_template.txt). Rationale: this matches the book's SFT/instruction-tuning stage, the first step of the RLHF pipeline, where chat templates turn raw message lists into the exact token stream the model is trained on.

## Related tools

- **Python** (Implements the ChatML renderer that wraps each message in <|im_start|>{role}\n{content}<|im_end|>\n markup and validates role ordering before rendering) — https://github.com/natolambert/rlhf-book
- **TRL** (Post-training framework whose SFT trainers consume chat-template-rendered multi-turn conversations for instruction tuning) — https://github.com/huggingface/trl
- **PyTorch** (Tensor backend for downstream fine-tuning of the rendered token sequences)

## Examples

```
msgs=[{'role':'system','content':'You are helpful.'},{'role':'user','content':'Hi'}]; rendered=''.join(f'<|im_start|>{m["role"]}\n{m["content"]}<|im_end|>\n' for m in msgs)
```

## Evaluation signals

- Rendered sequence contains <|im_start|> and <|im_end|> delimiters around every message
- Roles appear in the validated order: system message first, then strictly alternating user and assistant turns
- Generation-prompt variant ends exactly at the assistant opening tag '<|im_start|>assistant\n' with no closing <|im_end|>
- Each message's content is trimmed before insertion into the template
- The output file separates the complete-conversation render from the generation-prompt render

## Limitations

- The ChatML convention is the book's specific rendering choice; other chat templates use different special tokens, so rendered strings are model-family-specific
- The skill validates role ordering but does not enforce semantic correctness of message content or handle tool/function-call message types
- The generation-prompt render intentionally leaves an unclosed '<|im_start|>assistant\n' opener — it is invalid as a completed conversation and only meaningful as a continuation prefix

## Evidence

- [methods] ChatML renderer wraps each message in special tokens: [evidence span withheld — anchor status: not_found]
- [methods] Render full conversation to exact token sequence: [evidence span withheld — anchor status: not_found]
- [methods] Generation prompt handling: [evidence span withheld — anchor status: not_found]
