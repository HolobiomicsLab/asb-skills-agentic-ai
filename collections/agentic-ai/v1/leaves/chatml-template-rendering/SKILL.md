---
name: chatml-template-rendering
description: Use when preparing multi-turn role/content message lists for instruction
  tuning or SFT and you need the exact flat token-level render (ChatML delimiters)
  that the tokenizer consumes, including the add_generation_prompt continuation form.
when_to_use_negative:
- The target model uses a non-ChatML prompt format — e.g. Stanford Alpaca fine-tunes
  with '### Instruction:' / '### Input:' / '### Response:' prompts rather than <|im_start|>/<|im_end|>
  special tokens
- The input is a single-turn instruction/input/output triple (like the Alpaca 52K
  JSON schema) rather than a multi-turn role-alternating conversation
- You need actual tokenizer IDs rather than the template string — this skill produces
  the rendered token-sequence string, not token encodings
tools:
- name: Python
  role: Implements the ChatML renderer and message-order validation as list-of-dict
    processing
- name: TRL
  role: Downstream library whose SFT training consumes chat-template-rendered token
    sequences during instruction tuning
  repo: https://github.com/huggingface/trl
- name: stanford_alpaca
  role: 'Contrast reference: an alternative single-turn prompt template (''### Instruction:''
    / ''### Response:'') used for fine-tuning, showing where ChatML rendering does
    not apply'
  repo: https://github.com/tatsu-lab/stanford_alpaca
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:b86810c29fd96eb4d59a80ab12ef32678f480f54e7a3d0b89e40af15155c3281
  source_papers:
  - doi: 10.48550/arxiv.2504.12501
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/huggingface/trl
  tools:
  - PyTorch
  - Python
  - TRL
  - stanford_alpaca
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# chatml-template-rendering

## Summary

Implements and validates the ChatML-style chat template rendering described in the RLHF Book, serializing each message of a multi-turn conversation into a flat token sequence wrapped in '<|im_start|>'/'<|im_end|>' special tokens, including the add_generation_prompt variant. This is the token-level structure the model sees before instruction tuning (SFT).

## When to use

Use when preparing multi-turn role/content message lists for instruction tuning or SFT and you need the exact flat token-level render (ChatML delimiters) that the tokenizer consumes, including the add_generation_prompt continuation form.

## When NOT to use

- The target model uses a non-ChatML prompt format — e.g. Stanford Alpaca fine-tunes with '### Instruction:' / '### Input:' / '### Response:' prompts rather than <|im_start|>/<|im_end|> special tokens
- The input is a single-turn instruction/input/output triple (like the Alpaca 52K JSON schema) rather than a multi-turn role-alternating conversation
- You need actual tokenizer IDs rather than the template string — this skill produces the rendered token-sequence string, not token encodings

## Inputs

- Python list of message dictionaries with 'role' ('system'/'user'/'assistant') and 'content' string fields
- Boolean add_generation_prompt flag controlling the trailing assistant opening tag

## Outputs

- Complete-conversation render: a single flat string with '<|im_start|>{role}\n{content}<|im_end|>\n' per message
- Generation-prompt render: the same sequence ending exactly at '<|im_start|>assistant\n' with no closing '<|im_end|>'
- rendered_chat_template.txt containing both renders, clearly separated

## How to apply

Define the conversation in Python as a list of role/content dictionaries following the book's post-training convention: a system message first, then user/assistant turns that strictly alternate — validate this ordering before rendering. Render each message with the ChatML wrapper '<|im_start|>' + role + '\n' + trimmed content + '<|im_end|>\n', concatenating messages into a single flat token sequence string. For the add_generation_prompt option, produce a second render that appends a trailing '<|im_start|>assistant\n' opening without a closing '<|im_end|>', so a model would continue generating the assistant turn. Write both renders to rendered_chat_template.txt, clearly separating the complete-conversation render from the generation-prompt render. This matches the book's framing that instruction tuning (Chapter 4) trains a language model that can follow user questions, with TRL/Hugging Face training code consuming such templated sequences.

## Related tools

- **Python** (Implements the ChatML renderer and message-order validation as list-of-dict processing)
- **TRL** (Downstream library whose SFT training consumes chat-template-rendered token sequences during instruction tuning) — https://github.com/huggingface/trl
- **stanford_alpaca** (Contrast reference: an alternative single-turn prompt template ('### Instruction:' / '### Response:') used for fine-tuning, showing where ChatML rendering does not apply) — https://github.com/tatsu-lab/stanford_alpaca

## Examples

```
msgs=[{'role':'system','content':'You are helpful.'},{'role':'user','content':'Hi'},{'role':'assistant','content':'Hello'}]; render=''.join(f"<|im_start|>{m['role']}\n{m['content']}<|im_end|>\n" for m in msgs)
```

## Evaluation signals

- Every message in the rendered string is wrapped in '<|im_start|>' and '<|im_end|>' delimiters
- Roles appear in the validated order: system message first, then strictly alternating user/assistant turns
- The complete-conversation render ends with '<|im_end|>\n' after the final message
- The add_generation_prompt render ends exactly at '<|im_start|>assistant\n' with no closing '<|im_end|>'
- rendered_chat_template.txt contains both the complete render and the generation-prompt render, clearly separated

## Limitations

- The delimiter scheme is ChatML-specific; models with different chat markup (e.g. Alpaca's '### Instruction:' template) will not tokenize or train correctly with these special tokens
- Content is trimmed before wrapping — untrimmed whitespace or embedded '<|im_start|>'/'<|im_end|>' strings in message content would corrupt the token-level structure
- Validation only guards ordering (system first, user/assistant alternating); other conversation structures must be rejected before rendering

## Evidence

- [methods] ChatML serialization of each message: [evidence span withheld — anchor status: not_found]
- [methods] add_generation_prompt behavior: [evidence span withheld — anchor status: not_found]
- [methods] Message-order convention validated before rendering: [evidence span withheld — anchor status: not_found]
- [intro] Role in instruction tuning pipeline that consumes templated sequences: "First, a language model that can follow user questions must be trained (see Chapter 4)."
- [readme] TRL as the training library for post-training with templated conversations: [evidence span withheld — anchor status: not_found]
