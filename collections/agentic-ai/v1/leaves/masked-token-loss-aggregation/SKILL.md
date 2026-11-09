---
name: masked-token-loss-aggregation
description: Use when computing the training loss for chat/SFT or policy-gradient
  stages where the loss must apply only to completion tokens and prompt/padding tokens
  must be excluded via a completion_mask.", "how_to_apply
when_to_use_negative:
- Training objectives where every token should contribute to the loss (e.g., base-language-model
  pretraining without a prompt/completion split)
- Metrics that need unmasked statistics over full sequences (the mask would silently
  drop prompt positions from normalization)
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Tensor framework in which masked_mean and masked_sum helpers and the masked
    loss aggregation are implemented
- name: TRL
  role: Library used for the SFT/instruction-tuning and policy-gradient stages whose
    losses are aggregated with the completion mask
  repo: https://github.com/huggingface/trl
- name: stanford_alpaca
  role: Reference instruction-following fine-tuning codebase exemplifying the SFT
    stage where masked loss aggregation applies
  repo: https://github.com/tatsu-lab/stanford_alpaca
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:5366bc998379e7ef71dd68bbda220a379a498126e4893f1e211b95b6a6c96bde
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
  - Ray
  - TRL
  - stanford_alpaca
  - vLLM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# masked-token-loss-aggregation

## Summary

Aggregate per-token training losses (e.g., in SFT/instruction tuning or RL stages) by masking out irrelevant tokens (prompt tokens in completions, padding) and averaging the remaining per-token losses, using masked_mean/masked_sum helpers as implemented in the RLHF Book codebase.

## When to use

Use when computing the training loss for chat/SFT or policy-gradient stages where the loss must apply only to completion tokens and prompt/padding tokens must be excluded via a completion_mask.",
  "how_to_apply

## When NOT to use

- Training objectives where every token should contribute to the loss (e.g., base-language-model pretraining without a prompt/completion split)
- Metrics that need unmasked statistics over full sequences (the mask would silently drop prompt positions from normalization)

## Inputs

- per-token loss tensor (torch.Tensor), e.g., per_token_loss or per-token policy-gradient losses
- completion_mask tensor (torch.Tensor) marking 1 for completion tokens and 0 for prompt/padding tokens
- optional axis / constant_normalizer arguments for masked_sum

## Outputs

- scalar aggregated loss (per-sequence mean over unmasked tokens, then batch mean)
- masked_mean / masked_sum reduced tensors when applied along an axis

## How to apply

Build a completion_mask from the chat template so prompt tokens and padding are 0 and completion tokens are 1. Compute per-token losses (cross-entropy for SFT, or the PPO/GRPO per-token policy loss), then aggregate per sequence as ((per_token_loss * completion_mask).sum(dim=1) / completion_mask.sum(dim=1)).mean(), i.e., a length-normalized mean over unmasked tokens. Use the masked_mean(values, mask, axis=None) and masked_sum(values, mask, axis=None, constant_normalizer=1.0) helpers from the book's methods code (PyTorch) so masked positions contribute exactly zero to both the numerator and the normalization count. The rationale is that in the RLHF pipeline's instruction-tuning stage the model should learn from assistant completions only, not copy prompt content, and in RL stages the policy-gradient loss is aggregated per sequence with the same mask. Apply the same convention consistently across the SFT checkpoint and the downstream RL stage initialized from it.

## Related tools

- **PyTorch** (Tensor framework in which masked_mean and masked_sum helpers and the masked loss aggregation are implemented)
- **TRL** (Library used for the SFT/instruction-tuning and policy-gradient stages whose losses are aggregated with the completion mask) — https://github.com/huggingface/trl
- **stanford_alpaca** (Reference instruction-following fine-tuning codebase exemplifying the SFT stage where masked loss aggregation applies) — https://github.com/tatsu-lab/stanford_alpaca

## Examples

```
loss = ((per_token_loss * completion_mask).sum(dim=1) / completion_mask.sum(dim=1)).mean()
```

## Evaluation signals

- Masked positions contribute zero: numerator is per_token_loss * completion_mask, so setting all completion_mask entries to 1 must reproduce the unmasked mean
- Per-sequence normalization divides by completion_mask.sum(dim=1), so sequences with fewer completion tokens are not down-weighted relative to longer ones
- Prompt tokens must not appear in the gradient signal: verify with a probe batch that zeroing the mask yields loss 0.0
- Final scalar loss is finite and stable across batches (the RL stage logs finite loss and reward values)

## Limitations

- The mask must be constructed consistently with the chat template; a mismatched completion_mask silently excludes or includes wrong tokens
- Aggregation convention (per-sequence mean vs. per-token mean) changes effective learning-rate weighting on short vs. long completions; the book uses per-sequence normalization with a batch mean
- Software version numbers for the named tools (e.g., PyTorch, TRL) are not stated, so exact behavioral parity across library versions is not guaranteed

## Evidence

- [methods] masked_mean definition: [evidence span withheld — anchor status: not_found]
- [methods] masked_sum definition: [evidence span withheld — anchor status: not_found]
