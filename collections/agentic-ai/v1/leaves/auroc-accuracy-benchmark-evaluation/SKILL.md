---
name: auroc-accuracy-benchmark-evaluation
description: Use when a trained meme-classification or vision-language model must
  be scored on Hateful Memes Test Seen/Test Unseen or MultiOFF and compared against
  reported ACC/AUROC/F1 baselines.
when_to_use_negative:
- Evaluating on tasks or datasets outside harmful/offensive meme classification (e.g.
  general VQA or image captioning), where ACC/AUROC/F1 on Hateful Memes/MultiOFF are
  not the reported metrics
- Comparing models trained with different MLLM reasoning settings (e.g. LLaVA-1.5-7B
  vs GPT-4) without noting the n,m multi-CoT configuration, since reported numbers
  are tied to the n=3, m=3 setting
- The repository's models/, configs/, and scripts/ directories are still listed as
  'coming soon', so end-to-end evaluation scripts may not yet be runnable from the
  released code alone
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: Long-CLIP
  role: Frozen 248-token context encoder whose removal is quantified via the ACC/AUROC
    ablation metrics
  repo: https://github.com/BGWH123/EVL-MCoT
- name: LLaVA-1.5-7B
  role: MLLM used to generate multi-CoT reasoning for Hateful Memes evaluation runs
- name: GPT-4
  role: MLLM used to generate multi-CoT reasoning for MultiOFF evaluation runs
- name: AdamW
  role: Optimizer (lr 5e-5, weight decay 0.01, cosine scheduler) ensuring training
    conditions match reported benchmark numbers
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/978-981-95-3349-7_31
    title: 'EVL-MCoT: Enhanced Vision-Language Multi-CoT for Harmful Meme Detection'
  source_content_hash: sha256:7f3bb270b6682d30da03bb05f35a238329e3229615af0e0eaf40da5d6eed7651
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# auroc-accuracy-benchmark-evaluation

## Summary

Evaluates harmful meme detection models on Hateful Memes (Accuracy/AUROC) and MultiOFF (Accuracy/F1) using the metric pairs reported in EVL-MCoT, e.g. 75.88 ACC / 79.25 AUROC on Test Seen and 70.0 ACC / 63.8 F1 on MultiOFF, to benchmark model variants including ablations.

## When to use

Use when a trained meme-classification or vision-language model must be scored on Hateful Memes Test Seen/Test Unseen or MultiOFF and compared against reported ACC/AUROC/F1 baselines.

## When NOT to use

- Evaluating on tasks or datasets outside harmful/offensive meme classification (e.g. general VQA or image captioning), where ACC/AUROC/F1 on Hateful Memes/MultiOFF are not the reported metrics
- Comparing models trained with different MLLM reasoning settings (e.g. LLaVA-1.5-7B vs GPT-4) without noting the n,m multi-CoT configuration, since reported numbers are tied to the n=3, m=3 setting
- The repository's models/, configs/, and scripts/ directories are still listed as 'coming soon', so end-to-end evaluation scripts may not yet be runnable from the released code alone

## Inputs

- Trained EVL-MCoT model checkpoint (full model or ablated variant such as CoT n=1,m=3; no context-guided attention; no prototype-guided decoder; standard CLIP instead of Long-CLIP)
- Hateful Memes dataset (Test Seen and Test Unseen splits)
- MultiOFF dataset (743 offensive/non-offensive memes)

## Outputs

- Per-configuration Accuracy and AUROC on Hateful Memes Test Seen (and Test Unseen)
- Per-configuration Accuracy and F1 on MultiOFF
- Aggregated ablation results table with one row per ablation condition

## How to apply

After training a configuration (e.g. full EVL-MCoT with n=3, m=3, or an ablated variant), evaluate on Hateful Memes Test Seen reporting Accuracy and AUROC, and on MultiOFF reporting Accuracy and F1. Train with AdamW (learning rate 5e-5, weight decay 0.01, cosine learning rate scheduler), batch size 8 with gradient accumulation every 2 steps, so evaluation is comparable to the paper's setup. For ablation studies, aggregate per-condition metrics into a single table with one row per condition and verify the full model reproduces 75.88 ACC / 79.25 AUROC (Test Seen) and 70.0 ACC / 63.8 F1 (MultiOFF) within tolerance. Judge correctness by confirming ablated rows (e.g. removing Long-CLIP drops to 68.68 ACC / 71.77 AUROC) show the expected metric drop relative to the full model.

## Related tools

- **Long-CLIP** (Frozen 248-token context encoder whose removal is quantified via the ACC/AUROC ablation metrics) — https://github.com/BGWH123/EVL-MCoT
- **LLaVA-1.5-7B** (MLLM used to generate multi-CoT reasoning for Hateful Memes evaluation runs)
- **GPT-4** (MLLM used to generate multi-CoT reasoning for MultiOFF evaluation runs)
- **AdamW** (Optimizer (lr 5e-5, weight decay 0.01, cosine scheduler) ensuring training conditions match reported benchmark numbers)

## Examples

```
python train/evaluate.py --dataset hateful_memes --split test_seen --metrics acc,auroc --checkpoint evl-mcot_n3m3.pth
```

## Evaluation signals

- Full-model row reproduces 75.88 accuracy / 79.25 AUROC on Hateful Memes Test Seen within tolerance
- Full-model row reproduces 70.0 accuracy / 63.8 F1 on MultiOFF within tolerance
- Removing Long-CLIP yields 68.68 ACC / 71.77 AUROC on Test Seen and 62.60 ACC / 59.19 F1 on MultiOFF, a clear drop from the full model
- Every ablated condition (no context-guided attention, no prototype-guided decoder, reduced multi-CoT n,m) shows a metric drop relative to the full model as reported in the paper's ablation analysis
- Metrics are computed with the exact metric names per dataset: ACC+AUROC for Hateful Memes, ACC+F1 for MultiOFF

## Limitations

- Standard CLIP struggles with inputs exceeding 20 tokens, so evaluations involving long CoT sequences require Long-CLIP (248-token context) for comparable numbers
- Future work notes reasoning chains should be shortened while maintaining performance, implying current evaluation depends on long, costly multi-CoT chains
- Repository README indicates only the train/ directory is available; model weights, configs, and evaluation scripts are pending

## Evidence

- [methods] EVL-MCoT achieves 75.88% accuracy and 79.25 AUROC on Hateful Memes Test Seen: "EVL-MCoT 75.88 79.25 75.57 79.40"
- [results] MultiOFF metrics: [evidence span withheld — anchor status: not_found]
- [results] Long-CLIP ablation: [evidence span withheld — anchor status: not_found]
- [methods] Training hyperparameters: "The optimizer is set to AdamW [13], with a learning rate of 5e-5, a weight decay of 0.01, and a cosine learning rate scheduler"
- [methods] Evaluation setting: "The proposed EVL-MCoT is evaluated with the setting n=3, m=3 using LLaVA as the MLLM"
