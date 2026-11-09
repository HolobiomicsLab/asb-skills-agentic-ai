---
name: instruction-tuning-sft-stage
description: Use when building a post-training pipeline whose Stage 1 must convert
  a base model (e.g., Qwen/Qwen3-1.7B) into an instruction-following SFT checkpoint,
  using chat-formatted instruction data.
when_to_use_negative:
- The model already follows instructions — skip directly to reward-model training
  or preference optimization stages
- The task is optimizing the policy against a trained reward model (Stage 3 RL), not
  the initial supervised fine-tune
- Training a reward model on pairwise preference data (e.g., UltraFeedback with Bradley-Terry
  loss) — that is a separate stage
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: TRL
  role: Post-training library usable for the supervised fine-tuning / instruction-tuning
    stage
  repo: https://github.com/huggingface/trl
- name: PyTorch
  role: Implementing masked loss reductions (masked_mean) over completion masks during
    SFT
- name: tatsu-lab/stanford_alpaca
  role: Reference instruction-following dataset (52K examples) and Hugging Face-style
    fine-tuning recipe
  repo: https://github.com/tatsu-lab/stanford_alpaca
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:ece10cd96857ccb32e90a7adeec28020ac3715c30d2857fd130e8b0d4ef2cd2d
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
  - tatsu-lab/stanford_alpaca
  - vLLM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# instruction-tuning-sft-stage

## Summary

Implements the first stage of the canonical RLHF recipe: supervised fine-tuning (instruction tuning) of a base language model so it can follow user questions before reward-model training and RL optimization.

## When to use

Use when building a post-training pipeline whose Stage 1 must convert a base model (e.g., Qwen/Qwen3-1.7B) into an instruction-following SFT checkpoint, using chat-formatted instruction data.

## When NOT to use

- The model already follows instructions — skip directly to reward-model training or preference optimization stages
- The task is optimizing the policy against a trained reward model (Stage 3 RL), not the initial supervised fine-tune
- Training a reward model on pairwise preference data (e.g., UltraFeedback with Bradley-Terry loss) — that is a separate stage

## Inputs

- Base pretrained language model checkpoint (e.g., Qwen/Qwen3-1.7B)
- Instruction-following dataset with instruction/input/output fields (e.g., Stanford Alpaca 52K JSON; chat-template formatted)

## Outputs

- SFT checkpoint (instruction-tuned model weights) consumed by the reward-model and RL stages of the RLHF pipeline

## How to apply

Instantiate a base model such as Qwen/Qwen3-1.7B and fine-tune it on instruction-following data rendered through chat templates. Mask prompt tokens from the loss via the completion_mask so that only completion tokens contribute, aggregating per-sequence loss with masked reduction helpers like masked_mean over the completion mask. Produce an SFT checkpoint that downstream stages (reward-model training and policy-gradient RL, e.g., PPO/GRPO with KL-penalized rewards) initialize from. The reference recipe (natolambert/rlhf-book) uses standard Hugging Face-style training similar to the Stanford Alpaca fine-tuning code and can leverage TRL's post-training utilities.

## Related tools

- **TRL** (Post-training library usable for the supervised fine-tuning / instruction-tuning stage) — https://github.com/huggingface/trl
- **PyTorch** (Implementing masked loss reductions (masked_mean) over completion masks during SFT)
- **tatsu-lab/stanford_alpaca** (Reference instruction-following dataset (52K examples) and Hugging Face-style fine-tuning recipe) — https://github.com/tatsu-lab/stanford_alpaca

## Examples

```
python -m generate_instruction generate_instruction_following_data  # Alpaca-style 52K instruction data for the SFT stage
```

## Evaluation signals

- The SFT checkpoint file exists and loads after the instruction-tuning run
- Loss aggregates per sequence as ((per_token_loss * completion_mask).sum(dim=1) / completion_mask.sum(dim=1)).mean() with finite loss values logged
- Prompt tokens are excluded from the loss — verify the completion_mask correctly covers only completion positions
- Downstream RL stage initialized from the SFT checkpoint logs finite loss and reward values end-to-end

## Limitations

- No clear training details for the instruction tuning during rejection sampling have been released
- Alpaca-style SFT models are research-only and 'have not yet been fine-tuned ... to be safe and harmless'
- Software version numbers for the training stack are not stated in the source

## Evidence

- [methods] masked_mean loss helper over completion mask: [evidence span withheld — anchor status: not_found]
- [readme] Alpaca fine-tuning reference recipe and dataset: [evidence span withheld — anchor status: not_found]
