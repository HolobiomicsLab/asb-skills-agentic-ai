---
name: saliency-top-k-threshold-mask-generation
description: Use when you need to identify and mask the image regions most relevant
  to a question (e.g., a ScienceQA image-question pair) to create a perturbed sample
  that exposes task-critical visual-semantic directions for distillation.
when_to_use_negative:
- Input images have no associated question/task text, so cross-attention over image
  tokens cannot define question-relevant regions
- The goal is general data augmentation rather than generating negatively perturbed
  semantic samples for steering-vector distillation; VSSD's scope is small multimodal
  CoT settings
- Attention maps from the available MLLM are not accessible or not aligned to image
  token positions, so no reliable saliency map can be upsampled
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: VSSD
  role: Reference implementation providing the pretrained multimodal LLM and Algorithm
    1 (image perturbation construction) used to extract attention maps, build the
    top-k binary mask, and emit the perturbed image
  repo: https://github.com/BGWH123/VSSD
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1109/icassp55912.2026.11463006
    title: Visual Saliency Steering Distillation for Multimodal Chain-of-Thought Reasoning
  source_content_hash: sha256:34e894aa55a8916e7c37d6a82f192086d105b130224144f2c344b3c5cf1f0ef3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/VSSD
  tools:
  - VSSD
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# saliency-top-k-threshold-mask-generation

## Summary

Generates an attention-guided binary mask from a multimodal LLM's cross-attention maps over image tokens, then applies it to the original image to produce a negatively perturbed semantic sample (as in VSSD's Algorithm 1). The masked high-attention regions define the task-sensitive feature directions used for steering vector distillation.

## When to use

Use when you need to identify and mask the image regions most relevant to a question (e.g., a ScienceQA image-question pair) to create a perturbed sample that exposes task-critical visual-semantic directions for distillation.

## When NOT to use

- Input images have no associated question/task text, so cross-attention over image tokens cannot define question-relevant regions
- The goal is general data augmentation rather than generating negatively perturbed semantic samples for steering-vector distillation; VSSD's scope is small multimodal CoT settings
- Attention maps from the available MLLM are not accessible or not aligned to image token positions, so no reliable saliency map can be upsampled

## Inputs

- Pretrained multimodal LLM checkpoint from the VSSD repository (installed via requirements.txt)
- ScienceQA image-question pair (image plus question text)
- Cross-attention maps over image tokens from a forward pass
- Top-k threshold parameter for binarizing the smoothed saliency map

## Outputs

- Normalized, Gaussian-smoothed 2D saliency map at image resolution
- Binary mask covering the highest cross-attention (top-k saliency) regions
- Negatively perturbed semantic sample I_r saved as perturbed_image.png

## How to apply

Load a pretrained multimodal LLM from the VSSD repository (github.com/BGWH123/VSSD, environment installed via requirements.txt) and run a forward pass with the question text and image to extract cross-attention maps over the image tokens. Aggregate the attention maps and upsample them to the image resolution to form a 2D saliency map, then normalize and apply Gaussian smoothing to suppress noise. Threshold the smoothed saliency map at top-k to produce a binary mask covering the image regions most relevant to the question. Apply the binary mask to the input image to generate the negatively perturbed semantic sample I_r. VSSD then compares decoder feature responses to the original and masked-image features to capture the task-critical sensitive direction; the steering strength injected into intermediate layers is controlled by α, with α = 0.2 the optimal setting on ScienceQA.

## Related tools

- **VSSD** (Reference implementation providing the pretrained multimodal LLM and Algorithm 1 (image perturbation construction) used to extract attention maps, build the top-k binary mask, and emit the perturbed image) — https://github.com/BGWH123/VSSD

## Examples

```
git clone https://github.com/BGWH123/VSSD && cd VSSD && pip install -r requirements.txt; python construct_perturbation.py --task_id task_007 --image scienceqa_sample.png --question question.txt --topk 10 --output perturbed_image.png
```

## Evaluation signals

- perturbed_image.png exists and is a valid image file
- Masked pixels in perturbed_image.png coincide with the highest cross-attention (top-k saliency) regions of the smoothed attention map
- The saliency map is at full image resolution after upsampling and has been normalized and Gaussian-smoothed before top-k thresholding
- Perturbed image differs from the original exactly in the mask region (high-attention pixels masked), while low-attention pixels are unchanged
- Downstream feature difference between original and perturbed images yields a meaningful dominant steering vector, with steering strength α calibrated (α = 0.2 optimal on ScienceQA in the source study)

## Limitations

- Steering vector extraction is not yet adaptive; more adaptive extraction is deferred to future work
- Method is limited to small multimodal CoT settings; broader multimodal reasoning tasks are left to future work
- Hyperparameter α must be precisely calibrated: too small (e.g., 0.1) gives insufficient steering signal; too large (e.g., ≥0.3) distorts feature representations and degrades complex categories like SOC

## Evidence

- [intro] VSSD leverages MLLMs to identify and mask the image regions most relevant to the question, generating negatively perturbed semantic samples.: "VSSD leverages MLLMs to identify and mask the image regions most relevant to the question, generating negatively perturbed semantic samples."
- [discussion] VSSD generates attention-guided perturbed image to capture task-sensitive feature directions: "VSSD generates attention-guided perturbed image to capture task-sensitive feature directions"
- [methods] α = 0.2 yields the best overall accuracy on the ScienceQA dataset: "α = 0.2 yields the best overall accuracy on the ScienceQA dataset"
- [abstract] The code is available at https://github.com/BGWH123/VSSD.: "The code is available at https://github.com/BGWH123/VSSD."
- [readme] pip install -r requirements.txt: "pip install -r requirements.txt"
