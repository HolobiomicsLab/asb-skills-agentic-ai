---
name: cross-dataset-model-evaluation
description: Use when a vision-language meme-detection model has been trained (or
  ablated) and must be benchmarked on more than one dataset with dataset-specific
  metrics such as ACC/AUROC for Hateful Memes and ACC/F1 for MultiOFF.
when_to_use_negative:
- The task is single-dataset benchmarking against external baselines in Tables 1–2
  rather than internal ablation across component removals
- The model does not consume image-text meme pairs (e.g., text-only classification),
  since Long-CLIP/LLaVA/GPT-4-based multi-CoT evaluation assumes multimodal memes
- Input memes require processing beyond standard CLIP's 20-token limit but Long-CLIP
  (248-token context) is not available, making the ablation condition unreproducible
tools:
- name: Long-CLIP
  role: Visual and text encoder with 248-token context used for the full model; replacing
    it with standard CLIP (20-token limit) defines the ablation condition
  repo: https://github.com/BGWH123/EVL-MCoT
- name: LLaVA-1.5-7B
  role: MLLM generating the multi-CoT reasoning chains when evaluating on Hateful
    Memes (n=3, m=3 setting)
  repo: https://github.com/BGWH123/EVL-MCoT
- name: GPT-4
  role: MLLM generating the multi-CoT reasoning chains when evaluating on MultiOFF
    (n=3, m=3 setting)
- name: AdamW
  role: Optimizer for training each ablation configuration (lr 5e-5, weight decay
    0.01, cosine learning rate scheduler)
- name: CLIP
  role: Standard CLIP substitute for Long-CLIP in the ablation; limited to inputs
    under 20 tokens
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/978-981-95-3349-7_31
    title: 'EVL-MCoT: Enhanced Vision-Language Multi-CoT for Harmful Meme Detection'
  source_content_hash: sha256:c3c70b1ad5ce475dd927ea67b0b6300d9754d6e169bf91a6317c4d34e2b5989e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/EVL-MCoT
  tools:
  - AdamW
  - CLIP
  - GPT-4
  - LLaVA-1.5-7B
  - Long-CLIP
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-dataset-model-evaluation

## Summary

Evaluate a harmful-meme detection model (e.g., EVL-MCoT) on multiple benchmark datasets — Hateful Memes (Test Seen/Test Unseen, ACC and AUROC) and MultiOFF (ACC and F1) — including ablated variants, so performance is compared across datasets and model configurations in a single results table.

## When to use

Use when a vision-language meme-detection model has been trained (or ablated) and must be benchmarked on more than one dataset with dataset-specific metrics such as ACC/AUROC for Hateful Memes and ACC/F1 for MultiOFF.

## When NOT to use

- The task is single-dataset benchmarking against external baselines in Tables 1–2 rather than internal ablation across component removals
- The model does not consume image-text meme pairs (e.g., text-only classification), since Long-CLIP/LLaVA/GPT-4-based multi-CoT evaluation assumes multimodal memes
- Input memes require processing beyond standard CLIP's 20-token limit but Long-CLIP (248-token context) is not available, making the ablation condition unreproducible

## Inputs

- Trained model checkpoints for the full EVL-MCoT configuration (n=3, m=3) and each ablated variant (no Long-CLIP, no prototype-guided decoder, no context-guided decoder, reduced multi-CoT pairs)
- Hateful Memes dataset (over 10,000 multimodal examples, Test Seen and Test Unseen splits)
- MultiOFF dataset (743 memes labeled offensive or non-offensive)
- Training configuration: AdamW, lr 5e-5, weight decay 0.01, cosine scheduler, batch size 8, gradient accumulation 2

## Outputs

- Per-configuration metric rows: Accuracy and AUROC on Hateful Memes Test Seen
- Per-configuration metric rows: Accuracy and F1 on MultiOFF
- A single aggregated ablation results table with one row per ablation condition
- Comparison against reference full-model results (75.88/79.25 Hateful Memes Test Seen; 70.0/63.8 MultiOFF)

## How to apply

Train each model configuration (full EVL-MCoT with n=3, m=3 multi-CoT, and each ablated variant: standard CLIP replacing Long-CLIP, removal of the prototype-guided token fusion decoder, removal of the context-guided attention layer, or reduced multi-CoT pairs such as n=1,m=1) using AdamW with learning rate 5e-5, weight decay 0.01, a cosine learning rate scheduler, batch size 8 with gradient accumulation every 2 steps. Evaluate each trained model on Hateful Memes Test Seen (Accuracy and AUROC) and on MultiOFF (Accuracy and F1), using LLaVA-1.5-7B as the MLLM for Hateful Memes and GPT-4 for MultiOFF. Aggregate the per-condition metrics into a single ablation/comparison table with one row per configuration. Validate the full-model row against the reported references — 75.88 ACC / 79.25 AUROC on Hateful Memes Test Seen and 70.0 ACC / 63.8 F1 on MultiOFF — and confirm each ablated row shows the expected metric drop (e.g., removing Long-CLIP yields 68.68 ACC / 71.77 AUROC on Hateful Memes Test Seen and 62.60 ACC / 59.19 F1 on MultiOFF).

## Related tools

- **Long-CLIP** (Visual and text encoder with 248-token context used for the full model; replacing it with standard CLIP (20-token limit) defines the ablation condition) — https://github.com/BGWH123/EVL-MCoT
- **LLaVA-1.5-7B** (MLLM generating the multi-CoT reasoning chains when evaluating on Hateful Memes (n=3, m=3 setting)) — https://github.com/BGWH123/EVL-MCoT
- **GPT-4** (MLLM generating the multi-CoT reasoning chains when evaluating on MultiOFF (n=3, m=3 setting))
- **AdamW** (Optimizer for training each ablation configuration (lr 5e-5, weight decay 0.01, cosine learning rate scheduler))
- **CLIP** (Standard CLIP substitute for Long-CLIP in the ablation; limited to inputs under 20 tokens)

## Evaluation signals

- Full-model row (n=3, m=3) reproduces 75.88 accuracy / 79.25 AUROC on Hateful Memes Test Seen within tolerance
- Full-model row reproduces 70.0 accuracy / 63.8 F1 on MultiOFF within tolerance
- Removing Long-CLIP drops results to approximately 68.68 ACC / 71.77 AUROC on Hateful Memes Test Seen and 62.60 ACC / 59.19 F1 on MultiOFF
- Every ablated row (no context-guided attention layer, no prototype-guided token fusion decoder, reduced multi-CoT pairs) shows a metric drop relative to the full model, consistent with the paper's ablation analysis
- Output table has exactly one row per ablation condition with both dataset metrics filled (ACC/AUROC for Hateful Memes, ACC/F1 for MultiOFF)

## Limitations

- Standard CLIP cannot process the 150-word CoT sequences (20-token limit), so the Long-CLIP ablation inherently degrades input fidelity rather than only testing the encoder
- The repository README states model architecture definitions, configuration files, pretrained weights, and evaluation scripts are 'coming soon', so only the core training pipeline in train/ is currently available for reproduction
- Multi-CoT generation depends on the prior knowledge of the chosen MLLM (LLaVA-1.5-7B vs GPT-4), so results are not directly comparable across MLLM choices

## Evidence

- [other] removing Long-CLIP reduces performance to 68.68 ACC / 71.77 AUROC on Hateful Memes Test Seen and 62.60 ACC / 59.19 F1 on MultiOFF: [evidence span withheld — anchor status: not_found]
- [methods] EVL-MCoT achieves 75.88% accuracy and 79.25 AUROC on Hateful Memes Test Seen: "EVL-MCoT 75.88 79.25 75.57 79.40"
- [methods] The optimizer is set to AdamW, with a learning rate of 5e-5, a weight decay of 0.01, and a cosine learning rate scheduler: "The optimizer is set to AdamW [13], with a learning rate of 5e-5, a weight decay of 0.01, and a cosine learning rate scheduler"
- [methods] The MultiOFF dataset consists of 743 memes, annotated as offensive or non-offensive.: [evidence span withheld — anchor status: not_found]
- [readme] The core training code is available in the train/ directory.: "The core training code is available in the `train/` directory. Other components including model definitions, sample data, and configuration files will be added in upcoming updates."
