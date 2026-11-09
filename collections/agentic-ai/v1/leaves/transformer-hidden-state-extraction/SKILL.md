---
name: transformer-hidden-state-extraction
description: Use when you have forward passes of sample pairs (e.g., same-text-close-image
  or same-image-close-text cases from ScienceQA/M3CoT) through a transformer and need
  stage-wise hidden representations for similarity analysis.
when_to_use_negative:
- The task requires end-task accuracy or rationale-quality metrics rather than internal
  representation-level comparison (e.g., main ScienceQA/M3CoT benchmark evaluation)
- Only a single (unpaired) input per condition is available, so cross-sample cosine
  similarity between pairs cannot be defined
- The model architecture lacks separable Interaction & Fusion and Decoder stages that
  can be tapped independently
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Loads the base and VSSD-distilled checkpoints, runs forward passes over the
    paired ScienceQA/M3CoT samples, and taps hidden-state activations at the Interaction
    & Fusion and Decoder stages
  repo: https://github.com/BGWH123/VSSD
- name: Matplotlib
  role: Plots the per-stage cosine-similarity reductions between the base model and
    VSSD, saved as fig3b_cosine_similarity.png
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1109/icassp55912.2026.11463006
    title: Visual Saliency Steering Distillation for Multimodal Chain-of-Thought Reasoning
  source_content_hash: sha256:8d829ede8437ba25a2acc1d828ba35f8fe700e73623cfbd8f52fb6f3fe11d354
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/VSSD
  tools:
  - Matplotlib
  - PyTorch
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Transformer hidden-state extraction

## Summary

Extracts hidden-state activations from a multimodal LLM at specified stages (here the Interaction & Fusion and Decoder stages) so that paired-input representations can be compared via cosine similarity to quantify how a model encodes fine-grained cross-modal differences.

## When to use

Use when you have forward passes of sample pairs (e.g., same-text-close-image or same-image-close-text cases from ScienceQA/M3CoT) through a transformer and need stage-wise hidden representations for similarity analysis.

## When NOT to use

- The task requires end-task accuracy or rationale-quality metrics rather than internal representation-level comparison (e.g., main ScienceQA/M3CoT benchmark evaluation)
- Only a single (unpaired) input per condition is available, so cross-sample cosine similarity between pairs cannot be defined
- The model architecture lacks separable Interaction & Fusion and Decoder stages that can be tapped independently

## Inputs

- Base multimodal model checkpoint (from the VSSD repository environment)
- VSSD-distilled model checkpoint
- ScienceQA / M3CoT evaluation samples
- 100 same-text-close-image pairs (attention-guided masked/perturbed images with fixed question text)
- 100 same-image-close-text pairs (perturbed text with fixed image)

## Outputs

- Per-sample hidden-state activations at the Interaction & Fusion stage
- Per-sample hidden-state activations at the Decoder stage
- Per-stage cosine-similarity values for base vs. VSSD on both pair types
- Aggregated per-stage similarity reductions used to plot fig3b_cosine_similarity.png

## How to apply

Load the base multimodal model and the VSSD-distilled checkpoint from the BGWH123/VSSD repository environment (pip install -r requirements.txt) together with ScienceQA/M3CoT evaluation samples. Construct paired inputs — for the paper's Fig. 3(b) analysis, 100 same-text-close-image pairs (attention-guided masking/perturbation of the image with fixed question text) and 100 same-image-close-text pairs (perturbed text, fixed image). Forward each pair through both models and register hooks (or tap the model's hidden-state outputs) at the Interaction & Fusion stage and the Decoder stage, keeping the two stages separated so similarity can be computed per stage. Then compute cosine similarity between the paired hidden representations at each stage for both the base model and VSSD, and aggregate per-stage values to quantify the reduction achieved by VSSD relative to the base model.

## Related tools

- **PyTorch** (Loads the base and VSSD-distilled checkpoints, runs forward passes over the paired ScienceQA/M3CoT samples, and taps hidden-state activations at the Interaction & Fusion and Decoder stages) — https://github.com/BGWH123/VSSD
- **Matplotlib** (Plots the per-stage cosine-similarity reductions between the base model and VSSD, saved as fig3b_cosine_similarity.png)

## Examples

```
pip install -r requirements.txt && python extract_hidden_states.py --pairs same_text_close_image. --stages interaction_fusion,decoder --out fig3b_cosine_similarity.png
```

## Evaluation signals

- The reproduced fig3b_cosine_similarity.png must show VSSD yields a lower cross-modal cosine similarity than the base model at BOTH the Interaction & Fusion and Decoder stages
- The reduction pattern must match the one reported in Fig. 3(b) of the paper for both same-text-close-image and same-image-close-image pair types
- Hidden states must be extracted separately per stage — collapsing Interaction & Fusion and Decoder activations into one tensor invalidates the per-stage comparison
- Similarity must be computed on the same 100 pairs per condition for base and VSSD checkpoints so the reduction is attributable to the distillation, not to different inputs
- All 100 pairs per condition must produce finite, non-degenerate cosine values (no NaNs or zero vectors from failed forward passes)

## Limitations

- Analysis depends on the specific model architecture exposing distinguishable Interaction & Fusion and Decoder stages; other architectures may need re-mapping of tap points
- The underlying VSSD method is limited to small multimodal chain-of-thought settings; broader multimodal reasoning tasks are left to future work
- The same-image-close-text and same-text-close-image pairs are constructed via attention-guided masking/perturbation, so conclusions are conditioned on that perturbation strategy

## Evidence

- [other] Forward each pair through both models and extract hidden-state activations at the Interaction & Fusion stage and the Decoder stage.: [evidence span withheld — anchor status: not_found]
- [other] Cosine similarity computation per stage: [evidence span withheld — anchor status: not_found]
- [discussion] Fig 3(b) reduction finding: [evidence span withheld — anchor status: not_found]
- [readme] Repository environment setup: "pip install -r requirements.txt"
