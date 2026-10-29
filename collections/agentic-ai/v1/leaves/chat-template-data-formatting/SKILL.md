---
name: chat-template-data-formatting
description: Use when preparing instruction-following datasets (e.g., Stanford Alpaca-style
  instruction/input/output records or UltraFeedback preferences) for SFT with TRL/HuggingFace
  tokenizers and you must render examples through a chat template and mask prompt
  tokens.
when_to_use_negative:
- The downstream step is reward-model training or the RL update itself — those consume
  the formatted batches/templated prompts, not raw instruction records, and are separate
  skills
- The data is already a tokenized feature table with masks applied — re-applying the
  template would double-format the text
- You need direct-alignment (DPA/ORA-style) training that skips explicit templated
  SFT sampling entirely
tools:
- name: TRL
  role: HuggingFace post-training library used for the instruction-tuning/SFT stage
    and its chat-template/data-collation utilities
  repo: https://github.com/huggingface/trl
- name: PyTorch
  role: Tensor backend implementing the completion_mask-based masked loss so prompt
    tokens are excluded from SFT updates
- name: Stanford Alpaca dataset
  role: Example instruction/input/output data format (52K records, ~40% with an input
    field) mapped into chat-template user/assistant slots
  repo: https://github.com/tatsu-lab/stanford_alpaca
- name: Qwen/Qwen3-1.7B
  role: Book-default base model whose chat template defines the exact rendering of
    prompts and completions
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:23fc9919706d9b6edca7e750b3b3db8de9570061ba567e26cc71e353a594d750
  source_papers:
  - doi: 10.48550/arxiv.2504.12501
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/huggingface/trl
  tools:
  - FSDP
  - PyTorch
  - Python
  - Qwen/Qwen3-1.7B
  - Ray
  - Stanford Alpaca dataset
  - TRL
  - vLLM
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# chat-template-data-formatting

## Summary

Format instruction/preference data into model-specific chat-template renders for the RLHF pipeline's instruction-tuning (SFT) stage, so the base model (e.g., Qwen/Qwen3-1.7B) trains on properly structured multi-turn strings with prompt tokens masked from the loss.

## When to use

Use when preparing instruction-following datasets (e.g., Stanford Alpaca-style instruction/input/output records or UltraFeedback preferences) for SFT with TRL/HuggingFace tokenizers and you must render examples through a chat template and mask prompt tokens.

## When NOT to use

- The downstream step is reward-model training or the RL update itself — those consume the formatted batches/templated prompts, not raw instruction records, and are separate skills
- The data is already a tokenized feature table with masks applied — re-applying the template would double-format the text
- You need direct-alignment (DPA/ORA-style) training that skips explicit templated SFT sampling entirely

## Inputs

- Instruction-following JSON records with 'instruction', optional 'input', and 'output' string fields (e.g., alpaca_data., 52K examples)
- Pairwise preference datasets such as UltraFeedback (chosen/rejected completions with optional annotator rating deltas for the Llama-2 style margin term)
- Tokenizer/chat-template definition for the target base model (e.g., Qwen/Qwen3-1.7B)

## Outputs

- Tokenized SFT batches with input_ids, labels, and a completion_mask that zeroes out prompt tokens
- Rendered chat-formatted prompt strings consumed by the RL stage (code/policy_gradients, e.g., PPO/GRPO) for completion sampling
- SFT checkpoint produced by code/instruction_tuning from the templated data

## How to apply

Apply the tokenizer's chat template to each conversation record so the rendered text matches what the target model (book default Qwen/Qwen3-1.7B) expects at inference. During tokenization, build a completion_mask that marks prompt/system tokens so they are excluded from the loss, aggregating per-sequence loss only over completion tokens. For datasets like Stanford Alpaca, map the instruction/input/output fields into the template's user/assistant slots — Alpaca distinguishes examples with a non-empty 'input' field (paired instruction+input prompt) from empty-input examples, and ~40% of its 52K examples carry an input. Keep the same formatting function consistent across the SFT checkpoint and the downstream RL stage (code/policy_gradients) so the policy sees identically templated prompts when sampling completions for reward-model scoring.

## Related tools

- **TRL** (HuggingFace post-training library used for the instruction-tuning/SFT stage and its chat-template/data-collation utilities) — https://github.com/huggingface/trl
- **PyTorch** (Tensor backend implementing the completion_mask-based masked loss so prompt tokens are excluded from SFT updates)
- **Stanford Alpaca dataset** (Example instruction/input/output data format (52K records, ~40% with an input field) mapped into chat-template user/assistant slots) — https://github.com/tatsu-lab/stanford_alpaca
- **Qwen/Qwen3-1.7B** (Book-default base model whose chat template defines the exact rendering of prompts and completions)

## Examples

```
tokenizer.apply_chat_template(messages, tokenize=True, add_generation_prompt=False)  # then build completion_mask so prompt token labels are masked from the SFT loss
```

## Evaluation signals

- completion_mask is 0 on all prompt/system tokens and 1 only on completion (and eos_token) positions, so masked per-sequence loss equals ((per_token_loss * completion_mask).sum(dim=1) / completion_mask.sum(dim=1)).mean()
- Rendered template strings decode back to the expected user/assistant structure for the target model (Qwen/Qwen3-1.7B) and match what the RL stage feeds at sampling time
- Empty-input records use the instruction-only prompt variant while records with a non-empty 'input' field use the paired instruction+input variant
- The SFT stage trains to a finite loss and the RL stage initialized from that checkpoint logs finite loss and reward values (end-to-end bundle validation)

## Limitations

- Formatting is model-specific: a chat template rendered for one base model will not transfer unchanged to another tokenizer/template
- The Stanford Alpaca prompt file contained a known formatting error; users were directed to incorporate the fix in github.com/tatsu-lab/stanford_alpaca/pull/24
- Alpaca's data generation discarded the distinction between classification and non-classification instructions, and the dataset is CC BY NC 4.0 (non-commercial use only)

## Evidence

- [methods] Chat-template rendering with completion-token-only loss masking: [evidence span withheld — anchor status: not_found]
- [readme] Alpaca paired instruction+input prompt format: [evidence span withheld — anchor status: not_found]
- [readme] Alpaca record schema with optional input field: [evidence span withheld — anchor status: not_found]
- [intro] SFT is the first stage of the canonical RLHF recipe: "First, a language model that can follow user questions must be trained (see Chapter 4)."
