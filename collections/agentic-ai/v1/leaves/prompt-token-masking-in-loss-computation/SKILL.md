---
name: prompt-token-masking-in-loss-computation
description: Use when fine-tuning a base model on instruction data with a chat template
  and you must prevent the loss from back-propagating through prompt tokens (e.g.
  OLMo-2-0425-1B SFT on HuggingFaceH4/no_robots).
when_to_use_negative:
- Training on full next-token prediction over unstructured corpora (pretraining),
  where predicting every position including 'prompt-like' text is intended
- Direct alignment or RL stages (DPO, PPO) where the objective is over full sampled
  completions rather than supervised teacher forcing
- Datasets without chat-template formatting where prompt/completion boundaries cannot
  be reliably located
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: TRL
  role: TRL-style SFT trainer used to run instruction_tuning.train with prompt masking
    applied to the loss
  repo: https://github.com/huggingface/trl
- name: PyTorch
  role: Implements the masked reduction (masked_mean over the completion mask) in
    the loss computation
- name: Python
  role: Language for the companion SFT script and config handling
- name: tatsu-lab/stanford_alpaca
  role: Reference Hugging Face fine-tuning recipe and prompt formatting precedent
    cited by the book's instruction-tuning chapter
  repo: https://github.com/tatsu-lab/stanford_alpaca
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:4ee7ac53b7de6c4b14395eed5dd9ad9359f89f012e2c404f429314b3fbd7e450
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
  - tatsu-lab/stanford_alpaca
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# Prompt-token masking in loss computation

## Summary

A supervised fine-tuning (SFT) technique in which tokens belonging to the formatted prompt (including chat-template role markers) are masked out of the cross-entropy loss via a completion_mask, so the model only learns to predict completion tokens. This is the core loss computation behind the base-model-to-assistant transition demonstrated by the RLHF Book's SFT recipe.

## When to use

Use when fine-tuning a base model on instruction data with a chat template and you must prevent the loss from back-propagating through prompt tokens (e.g. OLMo-2-0425-1B SFT on HuggingFaceH4/no_robots).

## When NOT to use

- Training on full next-token prediction over unstructured corpora (pretraining), where predicting every position including 'prompt-like' text is intended
- Direct alignment or RL stages (DPO, PPO) where the objective is over full sampled completions rather than supervised teacher forcing
- Datasets without chat-template formatting where prompt/completion boundaries cannot be reliably located

## Inputs

- Tokenized prompt-completion pairs rendered via the model's chat template (message turns from HuggingFaceH4/no_robots)
- Binary completion_mask tensor (1 on completion tokens, 0 on prompt tokens) aligned to input_ids
- Model logits from the SFT trainer forward pass (e.g. allenai/OLMo-2-0425-1B)

## Outputs

- Masked cross-entropy loss where only completion tokens contribute
- Training log with monotonically decreasing instruction-tuning loss
- Before/after fixed-prompt generation comparison showing base-model rambling vs. assistant-style answers ending at <|endoftext|>

## How to apply

Load the SFT config (e.g. code/instruction_tuning/sft_olmo2_1b.yaml) and format each example's message turns with the model's chat template so role markers and content are tokenized consistently. Build a completion_mask aligned to the tokenized sequence: set it to 0 for every prompt-side token (system/user turns, role markers, template delimiters) and 1 for completion tokens, since the model should not learn to predict prompt tokens. In the loss computation, use masked reduction — as in the book's masked_mean(values, mask, axis) pattern — so only positions with mask=1 contribute to the cross-entropy. Capture fixed-prompt generations at step 0 and after convergence to verify the behavioral shift; rationale: masking keeps the model's distribution over prompts untouched and concentrates the gradient on assistant-style responses terminating at <|endoftext|>.

## Related tools

- **TRL** (TRL-style SFT trainer used to run instruction_tuning.train with prompt masking applied to the loss) — https://github.com/huggingface/trl
- **PyTorch** (Implements the masked reduction (masked_mean over the completion mask) in the loss computation)
- **Python** (Language for the companion SFT script and config handling)
- **tatsu-lab/stanford_alpaca** (Reference Hugging Face fine-tuning recipe and prompt formatting precedent cited by the book's instruction-tuning chapter) — https://github.com/tatsu-lab/stanford_alpaca

## Examples

```
python -m instruction_tuning.train --config sft_olmo2_1b.yaml  # trains allenai/OLMo-2-0425-1B on HuggingFaceH4/no_robots with prompt masking
```

## Evaluation signals

- Training log shows monotonically decreasing instruction-tuning loss over the run
- Sanity check: with an all-ones mask (masking removed), loss magnitude changes — confirming the mask actually zeroes prompt-token contributions
- At step 0, fixed-prompt generations ramble, repeat the prompt, and emit malformed role markers; after a few hundred steps the same prompts yield concise answers terminating at <|endoftext|>
- completion_mask positions align exactly with chat-template role markers and prompt spans (no completion tokens masked, no prompt tokens unmasked)

## Limitations

- Prompt masking requires a reliable chat template; malformed role markers or template drift silently corrupt the mask boundary
- The book notes no clear released training details for some instruction-tuning stages, so exact hyperparameters may need empirical tuning per model/dataset

## Evidence

- [methods] masked_mean implements masked loss reduction: [evidence span withheld — anchor status: not_found]
- [readme] Alpaca README shows prompt-side template with empty Response region for generation: [evidence span withheld — anchor status: not_found]
- [readme] TRL is the post-training library underlying the SFT trainer: [evidence span withheld — anchor status: not_found]
