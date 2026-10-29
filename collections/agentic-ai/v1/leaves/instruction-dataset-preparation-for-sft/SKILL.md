---
name: instruction-dataset-preparation-for-sft
description: Use when an instruction–completion dataset (e.g. HuggingFaceH4/no_robots,
  Stanford Alpaca's 52K alpaca_data.) must be formatted with a chat template and prompt-masked
  before training a base model into an assistant.
when_to_use_negative:
- The downstream stage is reward model training or RL optimization, not the initial
  instruction-tuning stage
- The dataset is already chat-template-formatted with valid loss masks, so reformatting
  would double-apply the template
- The goal is evaluation of an existing model (use a lighteval/inspect_ai evaluation
  harness instead of SFT preparation)
tools:
- name: TRL
  role: TRL-style SFT trainer used to run supervised fine-tuning with prompt masking
    and chat-template formatting
  repo: https://github.com/huggingface/trl
- name: PyTorch
  role: Tensor backend for the training loop and mask-weighted loss helpers (masked_mean)
  repo: https://github.com/natolambert/rlhf-book
- name: Python
  role: Implementation language for the companion instruction_tuning.train SFT script
  repo: https://github.com/natolambert/rlhf-book
- name: Stanford Alpaca
  role: Reference instruction-following dataset and fine-tuning recipe (52K instruction/input/output
    examples, Hugging Face training code)
  repo: https://github.com/tatsu-lab/stanford_alpaca
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:778d63e8260ee0d2d2d25a60d881b84215f46837114e223da00d2a47e66779da
  source_papers:
  - doi: 10.48550/arxiv.2504.12501
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/huggingface/trl
  tools:
  - PyTorch
  - Python
  - Stanford Alpaca
  - TRL
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# instruction-dataset-preparation-for-sft

## Summary

Prepares instruction-following datasets for supervised fine-tuning (SFT), the first optimization stage of the canonical RLHF recipe, by formatting message turns with a model's chat template and applying prompt masking so only completion tokens contribute to the loss.

## When to use

Use when an instruction–completion dataset (e.g. HuggingFaceH4/no_robots, Stanford Alpaca's 52K alpaca_data.) must be formatted with a chat template and prompt-masked before training a base model into an assistant.

## When NOT to use

- The downstream stage is reward model training or RL optimization, not the initial instruction-tuning stage
- The dataset is already chat-template-formatted with valid loss masks, so reformatting would double-apply the template
- The goal is evaluation of an existing model (use a lighteval/inspect_ai evaluation harness instead of SFT preparation)

## Inputs

- Instruction-following dataset (e.g. HuggingFaceH4/no_robots; comparable to Stanford Alpaca's 52K alpaca_data. with instruction/input/output fields)
- Base language model checkpoint (allenai/OLMo-2-0425-1B) with its chat template
- SFT config YAML (e.g. sft_olmo2_1b.yaml) from the rlhf-book codebase (code/instruction_tuning)

## Outputs

- Tokenized, chat-template-formatted batch with completion_mask (prompt tokens = 0, completion tokens = 1)
- Trained assistant model and training log showing instruction-tuning loss over the run
- Before/after fixed-prompt generation comparison documenting the base-model-to-assistant transition

## How to apply

Start from the RLHF pipeline's step 1: a language model that can follow user questions must be trained via instruction tuning (Chapter 4 of the RLHF Book). Load the SFT config (e.g. sft_olmo2_1b.yaml) with the instruction_tuning.train companion script, then load the base model (allenai/OLMo-2-0425-1B) and the dataset, applying the model's chat template to format each turn of the message list. Apply prompt masking so a completion_mask marks prompt tokens as 0 — the model should not learn to predict prompt tokens, only completions — which is the same masking philosophy the book's masked_mean/masked_sum helpers encode for token-weighted losses. Train with a TRL-style SFT trainer in Python/PyTorch, logging instruction-tuning loss, and capture fixed-prompt generations at step 0 and after convergence. Rationale: instruction tuning is the entry point of post-training (SFT, preference fine-tuning, RLVR), and demonstrating the base-model-to-assistant transition requires identical prompts before and after training with only completion tokens supervised.

## Related tools

- **TRL** (TRL-style SFT trainer used to run supervised fine-tuning with prompt masking and chat-template formatting) — https://github.com/huggingface/trl
- **PyTorch** (Tensor backend for the training loop and mask-weighted loss helpers (masked_mean)) — https://github.com/natolambert/rlhf-book
- **Python** (Implementation language for the companion instruction_tuning.train SFT script) — https://github.com/natolambert/rlhf-book
- **Stanford Alpaca** (Reference instruction-following dataset and fine-tuning recipe (52K instruction/input/output examples, Hugging Face training code)) — https://github.com/tatsu-lab/stanford_alpaca

## Examples

```
python -m instruction_tuning.train --config sft_olmo2_1b.yaml  # trains allenai/OLMo-2-0425-1B on HuggingFaceH4/no_robots with prompt masking
```

## Evaluation signals

- Training log shows monotonically decreasing instruction-tuning loss over the run
- completion_mask marks every prompt token as 0 and every completion token as 1 in the collated batch
- Chat-template formatting preserves role structure without malformed role markers in the rendered prompt
- Post-training fixed-prompt generations terminate at the <|endoftext|> token with concise answers, unlike step-0 base-model continuations that ramble and repeat the prompt
- Before/after comparison shows the shift from raw base-model continuations to assistant-style instruction-following responses

## Limitations

- Step-0 base models ramble, repeat the prompt, and emit malformed role markers, so a short run is not yet evidence of failure — compare after several hundred steps
- The Stanford Alpaca dataset is CC BY-NC 4.0 (non-commercial research only), and models trained on it inherit that restriction
- Alpaca-style models are not fine-tuned to be safe and harmless; users should exercise caution

## Evidence

- [methods] def masked_mean(values: torch.Tensor, mask: torch.Tensor, axis: Optional[int] = None): [evidence span withheld — anchor status: not_found]
- [readme] contains 52K instruction-following data we used for fine-tuning the Alpaca model.: [evidence span withheld — anchor status: not_found]
- [readme] We fine-tune our models using standard Hugging Face training code.: [evidence span withheld — anchor status: not_found]
