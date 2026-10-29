---
name: cross-attention-map-extraction-from-mllm
description: Use when you need question-relevant image regions from a multimodal LLM
  — e.g., to build saliency-guided perturbations or masking for a ScienceQA/M3CoT
  image-question pair before feature-difference or distillation analysis.
when_to_use_negative:
- Input has no paired question/text prompt, since the mask is conditioned on cross-attention
  between the question tokens and image tokens
- You need an adaptive or learned steering-vector extraction rather than the attention-saliency-based
  masking used here; the paper defers adaptive extraction to future work
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3377
- http://edamontology.org/topic_3474
tools:
- name: VSSD
  role: Provides the pretrained multimodal LLM, environment (requirements.txt), and
    pipeline for extracting cross-attention maps, building the top-k saliency mask,
    and producing the perturbed image I_r
  repo: https://github.com/BGWH123/VSSD
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1109/icassp55912.2026.11463006
    title: Visual Saliency Steering Distillation for Multimodal Chain-of-Thought Reasoning
  source_content_hash: sha256:5f21466be58bb81d50b16ff3b9e2f37207a27f2a2ec32bce93b0673d9bb49b38
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/VSSD
  tools:
  - VSSD
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3377
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-attention-map-extraction-from-mllm

## Summary

Extracts and aggregates cross-attention maps from a pretrained multimodal LLM over image tokens to build a question-conditioned 2D saliency map, which VSSD uses to generate negatively perturbed semantic samples for steering distillation.

## When to use

Use when you need question-relevant image regions from a multimodal LLM — e.g., to build saliency-guided perturbations or masking for a ScienceQA/M3CoT image-question pair before feature-difference or distillation analysis.

## When NOT to use

- Input has no paired question/text prompt, since the mask is conditioned on cross-attention between the question tokens and image tokens
- You need an adaptive or learned steering-vector extraction rather than the attention-saliency-based masking used here; the paper defers adaptive extraction to future work

## Inputs

- Pretrained multimodal LLM checkpoint from the VSSD repository (environment via requirements.txt)
- One ScienceQA (or M3CoT) image-question pair (image plus question text)

## Outputs

- Smoothed, normalized 2D saliency map upsampled to image resolution
- Attention-guided binary mask (top-k thresholded)
- Negatively perturbed semantic sample I_r saved as perturbed_image.png

## How to apply

Load a pretrained multimodal LLM from the VSSD repository (github.com/BGWH123/VSSD) with its requirements.txt environment, and feed one ScienceQA image-question pair through a forward pass to extract cross-attention maps over the image tokens. Aggregate the attention maps and upsample them to the image resolution to form a 2D saliency map, then normalize and apply Gaussian smoothing to suppress noise. Threshold the smoothed saliency map at top-k to produce a binary mask covering the image regions most relevant to the question. Apply this mask to the original image to generate the negatively perturbed semantic sample I_r, which downstream SVD-based steering vector extraction compares against original-image features to capture the task-critical sensitive direction.

## Related tools

- **VSSD** (Provides the pretrained multimodal LLM, environment (requirements.txt), and pipeline for extracting cross-attention maps, building the top-k saliency mask, and producing the perturbed image I_r) — https://github.com/BGWH123/VSSD

## Examples

```
pip install -r requirements.txt  # then run a VSSD forward pass over a ScienceQA image-question pair to extract cross-attention maps and save perturbed_image.png
```

## Evaluation signals

- perturbed_image.png exists and is a valid image file
- Masked pixels in perturbed_image.png coincide with the highest cross-attention (top-k saliency) regions of the smoothed attention map
- The saliency map is upsampled to the original image resolution and normalized before Gaussian smoothing is applied
- The perturbation is negative with respect to the question: the most question-relevant regions are removed/masked, so the difference between original and masked-image features isolates task-sensitive directions

## Limitations

- Mask quality depends on the MLLM's cross-attention being well-calibrated to question relevance; smoothing and top-k thresholding only suppress noise, they do not add information
- Steering vector extraction downstream is not yet adaptive; the authors note more adaptive extraction is deferred to future work
- Validated on ScienceQA and M3CoT; the method targets small multimodal CoT settings, with broader multimodal reasoning left to future work

## Evidence

- [intro] VSSD leverages MLLMs to identify and mask the image regions most relevant to the question, generating negatively perturbed semantic samples.: "VSSD leverages MLLMs to identify and mask the image regions most relevant to the question, generating negatively perturbed semantic samples."
- [other] Aggregate the attention maps and upsample them to the image resolution to form a 2D saliency map, then normalize and apply Gaussian smoothing to suppress noise.: [evidence span withheld — anchor status: not_found]
- [abstract] VSSD leverages the attention maps of multimodal large language models to generate perturbed images that capture task-sensitive feature directions, and then applies singular value decomposition to: "VSSD leverages the attention maps of multimodal large language models to generate perturbed images that capture task-sensitive feature directions, and then applies singular value decomposition to"
- [readme] pip install -r requirements.txt: "pip install -r requirements.txt"
