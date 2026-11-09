---
name: base-model-to-assistant-generation-comparison
description: Use when you need empirical proof that SFT converts a base model into
  an instruction-following assistant — i.e., you have a base checkpoint, an SFT config,
  and want matched-prompt generation diffs before vs. after training.
when_to_use_negative:
- The model is already an instruction-tuned assistant — the before/after transition
  contrast cannot be demonstrated from a step-0 capture
- You need RL/post-training stage comparisons (reward-model, PPO, DPO outputs) rather
  than the SFT base-to-assistant transition described in Chapter 4
- The dataset lacks prompt/completion structure, so completion_mask-based prompt masking
  cannot be applied
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3382
tools:
- name: TRL
  role: TRL-style SFT trainer used to run the instruction-tuning loop with prompt
    masking on no_robots
  repo: https://github.com/huggingface/trl
- name: PyTorch
  role: Tensor/mask machinery (e.g., masked_mean over a completion mask) used to implement
    prompt masking in the SFT loss
- name: Python
  role: Language in which the instruction_tuning.train script and generation-capture
    routines are written
- name: rlhf-book codebase
  role: Source of code/instruction_tuning and sft_olmo2_1b.yaml; its completions library
    is the documented standard for comparing model completions across post-training
    stages
  repo: https://github.com/natolambert/rlhf-book
- name: Stanford Alpaca training recipe
  role: 'Prior-art reference: fine-tunes a base LLaMA model on 52K instruction data
    with standard Hugging Face training code, analogous to the SFT stage reproduced
    here'
  repo: https://github.com/tatsu-lab/stanford_alpaca
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:c63d8242ecb2a0b5775323fee09c39129fa4b21ecf3daf616beca242f1a6195a
  source_papers:
  - doi: 10.48550/arxiv.2504.12501
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/huggingface/trl
  tools:
  - PyTorch
  - Python
  - Stanford Alpaca training recipe
  - TRL
  - rlhf-book codebase
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# base-model-to-assistant-generation-comparison

## Summary

Demonstrates the base-model-to-assistant transition by capturing fixed-prompt generations from a base LLM (e.g., allenai/OLMo-2-0425-1B) at step 0 of SFT and again after fine-tuning on instruction data (e.g., HuggingFaceH4/no_robots), then writing a before/after comparison alongside the training log.

## When to use

Use when you need empirical proof that SFT converts a base model into an instruction-following assistant — i.e., you have a base checkpoint, an SFT config, and want matched-prompt generation diffs before vs. after training.

## When NOT to use

- The model is already an instruction-tuned assistant — the before/after transition contrast cannot be demonstrated from a step-0 capture
- You need RL/post-training stage comparisons (reward-model, PPO, DPO outputs) rather than the SFT base-to-assistant transition described in Chapter 4
- The dataset lacks prompt/completion structure, so completion_mask-based prompt masking cannot be applied

## Inputs

- Base model checkpoint (e.g., allenai/OLMo-2-0425-1B)
- Instruction dataset (e.g., HuggingFaceH4/no_robots) with message turns
- SFT training config (e.g., sft_olmo2_1b.yaml from code/instruction_tuning)
- Model chat template applied to prompt-completion pairs
- Fixed set of held prompts for before/after generation capture

## Outputs

- Fixed-prompt generations captured at step 0 (raw base-model continuations)
- Fixed-prompt generations captured post-convergence (assistant-style responses ending at <end_of_turn>)
- Before/after generation comparison file
- Training log with instruction-tuning loss over the run

## How to apply

Load the SFT config (e.g., sft_olmo2_1b.yaml) and the training script from code/instruction_tuning, then load the base model (allenai/OLMo-2-0425-1B) and dataset (HuggingFaceH4/no_robots), formatting message turns with the model's chat template. Apply prompt masking with a completion_mask so loss is computed only on completion tokens — the model should not learn to predict prompt tokens (the book implements this pattern via masked_mean over a torch.Tensor mask). Before any optimizer step, generate responses on a fixed set of prompts and save them: at step 0 the base model is expected to ramble, repeat the prompt, and emit malformed role markers. Train to convergence, logging loss to a training log file, then regenerate on the exact same prompts; the post-training completions should be concise, terminate at the <end_of_turn> token, and follow assistant-style formatting. Write the before/after generation comparison alongside the training log as the deliverable, consistent with the book's completions comparison library.

## Related tools

- **TRL** (TRL-style SFT trainer used to run the instruction-tuning loop with prompt masking on no_robots) — https://github.com/huggingface/trl
- **PyTorch** (Tensor/mask machinery (e.g., masked_mean over a completion mask) used to implement prompt masking in the SFT loss)
- **Python** (Language in which the instruction_tuning.train script and generation-capture routines are written)
- **rlhf-book codebase** (Source of code/instruction_tuning and sft_olmo2_1b.yaml; its completions library is the documented standard for comparing model completions across post-training stages) — https://github.com/natolambert/rlhf-book
- **Stanford Alpaca training recipe** (Prior-art reference: fine-tunes a base LLaMA model on 52K instruction data with standard Hugging Face training code, analogous to the SFT stage reproduced here) — https://github.com/tatsu-lab/stanford_alpaca

## Examples

```
python -m instruction_tuning.train --config sft_olmo2_1b.yaml
```

## Evaluation signals

- Training log shows monotonically decreasing instruction-tuning loss over the run
- Step-0 fixed-prompt generations exhibit base-model pathology: rambling, prompt repetition, malformed role markers
- Post-convergence fixed-prompt generations on the same prompts are concise and terminate at the <end_of_turn> token
- completion_mask correctly marks prompt tokens as 0 so only completion tokens contribute to the loss
- Before/after comparison shows a shift from raw continuations to instruction-following responses consistent with the book's completions library

## Limitations

- Software version numbers for named tools (e.g., PyTorch, TRL) are not stated in the book, so reproduction may vary across versions
- Alpaca-style SFT models are research-grade: safety fine-tuning is not part of this stage, so post-SFT behavior should be treated cautiously
- The comparison is qualitative (fixed-prompt before/after), not a benchmarked evaluation; it does not quantify assistant quality

## Evidence

- [abstract] companion resources including codebase and completions library: "The book is released with a variety of companion resources, including a codebase, a library to compare model completions from within post-training stages, and an educational course"
- [methods] prompt masking implementation in the book: [evidence span withheld — anchor status: not_found]
- [intro] SFT as first stage of the RLHF pipeline: "First, a language model that can follow user questions must be trained (see Chapter 4)."
- [readme] Alpaca precedent for base-model instruction fine-tuning: [evidence span withheld — anchor status: not_found]
