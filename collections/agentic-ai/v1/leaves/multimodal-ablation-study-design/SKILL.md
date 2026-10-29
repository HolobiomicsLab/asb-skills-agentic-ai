---
name: multimodal-ablation-study-design
description: Use when a multimodal vision-language model (e.g., harmful meme detection
  with CoT reasoning) must be decomposed to identify which architectural components
  or reasoning-chain settings (n, m) drive reported metrics.
when_to_use_negative:
- The goal is benchmark comparison of the full model against baselines (Tables 1-2),
  not internal component attribution
- Single-chain or single-component pipelines where removing one part leaves no meaningful
  remaining system to evaluate
- Datasets without both a full-model reference result and matching evaluation metrics,
  since deltas cannot be validated
edam_operation: http://edamontology.org/operation_0362
tools:
- name: Long-CLIP
  role: Backbone encoder extended to 248-token context; ablated by replacing with
    standard CLIP limited to 20 tokens
  repo: https://github.com/BGWH123/EVL-MCoT
- name: LLaVA-1.5-7B
  role: MLLM generating multi-CoT chains for Hateful Memes experiments
- name: GPT-4
  role: MLLM generating multi-CoT chains for MultiOFF experiments
- name: AdamW
  role: Optimizer (lr 5e-5, weight decay 0.01, cosine scheduler) used identically
    across all ablation conditions
- name: CLIP
  role: Standard-CLIP substitute whose 20-token limit demonstrates the Long-CLIP ablation
    contrast
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1007/978-981-95-3349-7_31
    title: 'EVL-MCoT: Enhanced Vision-Language Multi-CoT for Harmful Meme Detection'
  source_content_hash: sha256:fcba7a90ed140d72f242fdbd1f94aa96c8be3d2ad8a2e41050b2bac1b6202b1b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/EVL-MCoT
  tools:
  - AdamW
  - CLIP
  - GPT-4
  - LLaVA
  - LLaVA-1.5-7B
  - Long-CLIP
  edam_operation: http://edamontology.org/operation_0362
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# multimodal-ablation-study-design

## Summary

Designs and executes component-removal ablation studies for multimodal detection models like EVL-MCoT, systematically removing Long-CLIP, the prototype-guided decoder, the context-guided decoder, or multi-CoT chains to quantify each component's contribution to ACC/AUROC on Hateful Memes and ACC/F1 on MultiOFF.

## When to use

Use when a multimodal vision-language model (e.g., harmful meme detection with CoT reasoning) must be decomposed to identify which architectural components or reasoning-chain settings (n, m) drive reported metrics.

## When NOT to use

- The goal is benchmark comparison of the full model against baselines (Tables 1-2), not internal component attribution
- Single-chain or single-component pipelines where removing one part leaves no meaningful remaining system to evaluate
- Datasets without both a full-model reference result and matching evaluation metrics, since deltas cannot be validated

## Inputs

- Hateful Memes dataset (10,000+ multimodal memes, hateful/non-hateful labels; Test Seen and Test Unseen splits)
- MultiOFF dataset (743 memes, offensive/non-offensive labels)
- EVL-MCoT training pipeline from github.com/BGWH123/EVL-MCoT (core code in train/)
- Full-model reference metrics (75.88 ACC / 79.25 AUROC on Test Seen; 70.0 ACC / 63.8 F1 on MultiOFF)

## Outputs

- Ablation results table with one row per condition (component removed or CoT setting), per-dataset metrics (ACC, AUROC for Hateful Memes; ACC, F1 for MultiOFF)
- Per-metric deltas (ablated minus full setting) quantifying each component's contribution, e.g., -6.68 ACC / -9.55 AUROC on Test Seen for w/o CoT (n=1,m=1)

## How to apply

Configure one model variant per ablation condition: (a) CoT ablations with multi-CoT pairs (n=1,m=1), (n=1,m=3), and (n=3,m=1) using LLaVA-1.5-7B as the MLLM for Hateful Memes and GPT-4 for MultiOFF; (b) remove the context-guided attention layer; (c) remove the prototype-guided token fusion decoder; (d) replace Long-CLIP (248-token context) with standard CLIP (20-token limit). Train each configuration with AdamW (learning rate 5e-5, weight decay 0.01, cosine scheduler), batch size 8 with gradient accumulation every 2 steps on NVIDIA RTX 3090/4090 GPUs. Evaluate each variant on Hateful Memes Test Seen (Accuracy and AUROC) and MultiOFF (Accuracy and F1), then aggregate one row per condition into a single ablation table. Anchor validation on the full model (n=3,m=3): 75.88 ACC / 79.25 AUROC on Test Seen and 70.0 ACC / 63.8 F1 on MultiOFF; every ablated row must show the expected metric drop relative to the full model.

## Related tools

- **Long-CLIP** (Backbone encoder extended to 248-token context; ablated by replacing with standard CLIP limited to 20 tokens) — https://github.com/BGWH123/EVL-MCoT
- **LLaVA-1.5-7B** (MLLM generating multi-CoT chains for Hateful Memes experiments)
- **GPT-4** (MLLM generating multi-CoT chains for MultiOFF experiments)
- **AdamW** (Optimizer (lr 5e-5, weight decay 0.01, cosine scheduler) used identically across all ablation conditions)
- **CLIP** (Standard-CLIP substitute whose 20-token limit demonstrates the Long-CLIP ablation contrast)

## Examples

```
full=(75.88,79.25); wot_cot=(69.20,69.70); deltas=tuple(a-b for a,b in zip(wot_cot,full))  # -> (-6.68, -9.55) on Test Seen
```

## Evaluation signals

- Full-model row (n=3,m=3) reproduces 75.88 ACC / 79.25 AUROC on Hateful Memes Test Seen and 70.0 ACC / 63.8 F1 on MultiOFF within tolerance
- w/o CoT (n=1,m=1) row lands at 69.20 ACC / 69.70 AUROC on Test Seen and 61.08 ACC / 55.70 F1 on MultiOFF
- Long-CLIP removal row lands at 68.68 ACC / 71.77 AUROC on Test Seen and 62.60 ACC / 59.19 F1 on MultiOFF
- Every ablated row shows a metric drop relative to the full model, with CoT removal causing the largest degradation
- All deltas recomputed from the source table values exactly match (ablated minus full setting)

## Limitations

- Standard CLIP struggles with inputs exceeding 20 tokens while the CoT approach requires processing 150-word sequences, so the Long-CLIP ablation conflates encoder swap with context-length truncation
- MLLM differs across datasets (LLaVA-1.5-7B for Hateful Memes, GPT-4 for MultiOFF), so ablation deltas are not directly comparable across datasets
- The repository README indicates only the train/ pipeline is available; models/, configs/, and scripts/ are 'coming soon', limiting out-of-the-box reproducibility
- Future work notes the need for strategies to shorten reasoning chains while maintaining performance, suggesting compute-heavy n=3,m=3 settings may not be optimal

## Evidence

- [results] In the ablation on Table 3, removing Long-CLIP reduces performance to 68.68 ACC / 71.77 AUROC on Hateful Memes Test Seen and 62.60 ACC / 59.19 F1 on MultiOFF: [evidence span withheld — anchor status: not_found]
- [results] The w/o CoT (n=1, m=1) ablation achieves 69.20 ACC and 69.70 AUROC on Test Seen, well below the full model's 75.88/79.25: [evidence span withheld — anchor status: not_found]
- [methods] The optimizer is AdamW with lr 5e-5, weight decay 0.01, and a cosine learning rate scheduler: "The optimizer is set to AdamW [13], with a learning rate of 5e-5, a weight decay of 0.01, and a cosine learning rate scheduler"
- [methods] Standard CLIP struggles with inputs exceeding 20 tokens while the CoT approach requires processing 150-word sequences: "The standard CLIP model struggles with inputs exceeding 20 tokens, while our CoT approach requires processing 150-word sequences."
- [readme] The core training code is available in the train/ directory of the repository: "The core training code is available in the `train/` directory. Other components including model definitions, sample data, and configuration files will be added in upcoming updates."
