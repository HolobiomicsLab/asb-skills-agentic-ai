---
name: multimodal-evaluation-pair-construction
description: Use when you need to test whether a multimodal model (e.g., a VSSD-distilled
  checkpoint vs. a base model) distinguishes visually or textually similar yet semantically
  distinct sample pairs on ScienceQA/M3CoT-style data.
when_to_use_negative:
- When the goal is overall answer-accuracy benchmarking (e.g., tuning alpha or beta
  on ScienceQA subcategories) rather than fine-grained representational-sensitivity
  analysis of paired samples
- When inputs are unimodal (text-only) with no cross-modal interaction and fusion
  stages to probe
- When you need adaptive steering-vector extraction; the article notes this is deferred
  to future work
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0003
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Forward paired samples through base and VSSD checkpoints, extract hidden-state
    activations at the Interaction & Fusion and Decoder stages, and compute cosine
    similarity between paired representations
  repo: https://github.com/BGWH123/VSSD
- name: Matplotlib
  role: Plot the per-stage cosine-similarity reductions and save the figure as fig3b_cosine_similarity.png
  repo: https://github.com/BGWH123/VSSD
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1109/icassp55912.2026.11463006
    title: Visual Saliency Steering Distillation for Multimodal Chain-of-Thought Reasoning
  source_content_hash: sha256:0d1b33c194e8cd5321e591c4656c82cc32d88d17f7ea6405bbebe8f2a78adfb9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/VSSD
  tools:
  - Matplotlib
  - PyTorch
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0003
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# multimodal-evaluation-pair-construction

## Summary

Constructs controlled contrastive evaluation pairs (same-text-close-image and same-image-close-text) for a multimodal model and measures per-stage hidden-state cosine similarity to quantify whether VSSD suppresses superficial cross-modal similarity, reproducing Fig. 3(b) of the VSSD paper.

## When to use

Use when you need to test whether a multimodal model (e.g., a VSSD-distilled checkpoint vs. a base model) distinguishes visually or textually similar yet semantically distinct sample pairs on ScienceQA/M3CoT-style data.

## When NOT to use

- When the goal is overall answer-accuracy benchmarking (e.g., tuning alpha or beta on ScienceQA subcategories) rather than fine-grained representational-sensitivity analysis of paired samples
- When inputs are unimodal (text-only) with no cross-modal interaction and fusion stages to probe
- When you need adaptive steering-vector extraction; the article notes this is deferred to future work

## Inputs

- Base multimodal model checkpoint and VSSD-distilled checkpoint (from the BGWH123/VSSD repository)
- ScienceQA and M3CoT evaluation samples with question text and image
- MLLM attention maps used to select image regions for masking/perturbation

## Outputs

- Per-stage cosine-similarity values (Interaction & Fusion stage and Decoder stage) for 100 same-text-close-image and 100 same-image-close-text pairs, for base model and VSSD
- Quantified per-stage similarity reduction of VSSD vs. base model
- Figure fig3b_cosine_similarity.png (per-stage cosine-similarity reduction plot)

## How to apply

Load the base multimodal model and the VSSD-distilled checkpoint from the BGWH123/VSSD repository (pip install -r requirements.txt) together with ScienceQA/M3CoT evaluation samples. Build 100 same-text-close-image pairs by applying attention-guided masking/perturbation to the image while keeping the question text fixed (mirroring VSSD's use of MLLM attention maps to mask task-relevant regions), and 100 same-image-close-text pairs by perturbing the text while keeping the image fixed. Forward each pair through both models and extract hidden-state activations separately at the Interaction & Fusion stage and the Decoder stage. Compute cosine similarity between paired hidden representations per stage for each model, then aggregate per-stage values and quantify VSSD's reduction relative to the base model. Plot the per-stage cosine-similarity reductions and save the figure as fig3b_cosine_similarity.png.

## Related tools

- **PyTorch** (Forward paired samples through base and VSSD checkpoints, extract hidden-state activations at the Interaction & Fusion and Decoder stages, and compute cosine similarity between paired representations) — https://github.com/BGWH123/VSSD
- **Matplotlib** (Plot the per-stage cosine-similarity reductions and save the figure as fig3b_cosine_similarity.png) — https://github.com/BGWH123/VSSD

## Examples

```
sim = torch.nn.functional.cosine_similarity(h_pair_a, h_pair_b, dim=-1).mean().item(); plt.savefig('fig3b_cosine_similarity.png')
```

## Evaluation signals

- The reproduced plot shows VSSD yields lower cross-modal cosine similarity than the base model at both the Interaction & Fusion and Decoder stages, matching the reduction pattern of Fig. 3(b)
- The reduction is observed for both pair types: 100 same-text-close-image cases and 100 same-image-close-text cases
- Exactly 100 pairs are constructed per condition, with text fixed in the image-perturbed condition and image fixed in the text-perturbed condition
- Similarity values are computed per stage (Interaction & Fusion vs. Decoder) separately, not pooled across stages
- The environment installs successfully via pip install -r requirements.txt and both checkpoints forward correctly

## Limitations

- Method is limited to small multimodal CoT settings; broader multimodal reasoning tasks are left to future work
- Steering vector extraction is not yet adaptive; more adaptive extraction is deferred to future work
- No quantitative results, hyperparameters, or model versions are reported in the discussion section itself
- The hyperparameters alpha and beta should be kept at their optimal value of 0.2; deviations (alpha >= 0.3, or beta increased/decreased) degrade performance, e.g., on complex categories like SOC

## Evidence

- [intro] VSSD leverages MLLMs to identify and mask the image regions most relevant to the question, generating negatively perturbed semantic samples.: "VSSD leverages MLLMs to identify and mask the image regions most relevant to the question, generating negatively perturbed semantic samples."
- [intro] current models still lack sufficient sensitivity and understanding of fine-grained distinctions across different inputs: "current models still lack sufficient sensitivity and understanding of fine-grained distinctions across different inputs."
- [discussion] VSSD generates attention-guided perturbed image to capture task-sensitive feature directions: "VSSD generates attention-guided perturbed image to capture task-sensitive feature directions"
- [methods] alpha = 0.2 yields the best overall accuracy on the ScienceQA dataset: "α = 0.2 yields the best overall accuracy on the ScienceQA dataset"
- [readme] beta = 0.2 effectively balances the main task loss and the visually saliency-guided distillation loss within the current architecture: "β = 0.2 effectively balances the main task loss and the visually saliency-guided distillation loss within the current architecture"
