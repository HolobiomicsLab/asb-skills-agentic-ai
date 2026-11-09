---
name: supervised-fine-tuning-with-chat-templates
description: Use when a pretrained base LLM must be turned into an assistant that
  follows user questions — i.e., the instruction-tuning stage before reward model
  training or RL, using a chat-template-formatted SFT dataset.
when_to_use_negative:
- The task is a later post-training stage (reward model training, rejection sampling,
  RL, direct alignment) — SFT is only the first step of the pipeline.
- The input is a preference/reward dataset (e.g., UltraFeedback, HelpSteer2-Preference)
  rather than prompt-completion instruction data with chat-template turns.
- You need general next-token LM training without chat structure or completion masking
  (prompt tokens should not be excluded from the loss).
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: TRL
  role: SFT trainer used for the instruction-tuning run; the book names TRL as a post-training
    framework and the repo huggingface/trl is cited as companion code
  repo: https://github.com/huggingface/trl
- name: PyTorch
  role: Backend for the masked loss computation (masked_mean/masked_sum over completion-masked
    token tensors)
- name: Python
  role: Language used to run the instruction_tuning.train script and generation captures
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:28f1db9ce301dbf13523bf9a4a17e97a63c616bde5e58efeaf295cb0342f8c02
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# supervised-fine-tuning-with-chat-templates

## Summary

Step 1 of the RLHF pipeline: supervised fine-tuning (SFT) that converts a base model (e.g., allenai/OLMo-2-0425-1B) into an instruction-following assistant by training on an instruction dataset (HuggingFaceH4/no_robots) formatted with a chat template, with prompt masking so only completion tokens contribute to the loss.

## When to use

Use when a pretrained base LLM must be turned into an assistant that follows user questions — i.e., the instruction-tuning stage before reward model training or RL, using a chat-template-formatted SFT dataset.

## When NOT to use

- The task is a later post-training stage (reward model training, rejection sampling, RL, direct alignment) — SFT is only the first step of the pipeline.
- The input is a preference/reward dataset (e.g., UltraFeedback, HelpSteer2-Preference) rather than prompt-completion instruction data with chat-template turns.
- You need general next-token LM training without chat structure or completion masking (prompt tokens should not be excluded from the loss).

## Inputs

- Base language model checkpoint (allenai/OLMo-2-0425-1B)
- Instruction-following dataset (HuggingFaceH4/no_robots; comparable to Stanford Alpaca 52K style data)
- SFT YAML config (sft_olmo2_1b.yaml)
- Model tokenizer with chat template
- Fixed set of evaluation prompts for before/after generation capture

## Outputs

- Fine-tuned assistant model checkpoints
- Training log with monotonically decreasing instruction-tuning loss and metrics
- Before/after fixed-prompt generation comparison (base-model continuations vs. assistant-style responses)

## How to apply

Load an SFT config (e.g., sft_olmo2_1b.yaml) and the training script from code/instruction_tuning, then load the base model (allenai/OLMo-2-0425-1B) and dataset (HuggingFaceH4/no_robots), formatting message turns with the model's chat template. Apply prompt masking: completion_mask marks prompt tokens as 0 so the model does not learn to predict prompt tokens, mirroring the book's masked_mean/masked_sum loss utilities. Capture fixed-prompt generations at step 0 to record base-model behavior, train to convergence with a TRL-style SFT trainer while logging instruction-tuning loss, then regenerate on the same fixed prompts to produce a before/after comparison. The rationale is that this stage teaches the model to follow user questions and emit well-formed role markers terminated by <|endoftext|>, which is prerequisite to reward model training and RL optimization in the canonical RLHF recipe.

## Related tools

- **TRL** (SFT trainer used for the instruction-tuning run; the book names TRL as a post-training framework and the repo huggingface/trl is cited as companion code) — https://github.com/huggingface/trl
- **PyTorch** (Backend for the masked loss computation (masked_mean/masked_sum over completion-masked token tensors))
- **Python** (Language used to run the instruction_tuning.train script and generation captures)

## Examples

```
python -m instruction_tuning.train --config sft_olmo2_1b.yaml
```

## Evaluation signals

- Training log shows monotonically decreasing instruction-tuning loss over the run
- At step 0, fixed-prompt generations show raw base-model behavior (rambling, repeating the prompt, malformed role markers); post-training the same prompts yield concise answers
- Post-convergence generations terminate at the <|endoftext|> token instead of continuing indefinitely
- completion_mask correctly marks prompt tokens as 0 so only completion tokens contribute to the loss
- Post-training outputs are assistant-style, instruction-following responses consistent with the book's documented completions library

## Limitations

- SFT alone does not make the model safe or harmless — instruction-tuned models still require the later preference-tuning/RL stages (cf. Alpaca: 'We thus encourage users to be cautious when interacting with Alpaca').
- Base models before SFT emit malformed role markers and repeat prompts, so pre-training generation captures are expected to be low quality and are diagnostic, not evaluative.
- No clear training details for the instruction-tuning stage of rejection sampling have been released, so replication may require the book's own companion configs rather than upstream recipes.

## Evidence

- [methods] Masked loss utilities: [evidence span withheld — anchor status: not_found]
- [readme] SFT training code pattern: [evidence span withheld — anchor status: not_found]
- [readme] TRL library purpose: [evidence span withheld — anchor status: not_found]
- [readme] SFT safety caveat: [evidence span withheld — anchor status: not_found]
