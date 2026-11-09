---
name: attention-map-upsampling-and-gaussian-smoothing
description: Use when you have an image-question pair (e.g., ScienceQA) and a pretrained
  multimodal LLM that exposes cross-attention maps, and you need to generate a negatively
  perturbed image whose masked regions correspond to the question-relevant high-attention
  areas.
when_to_use_negative:
- The input is text-only (no image tokens), so no cross-attention saliency over visual
  regions can be aggregated and upsampled.
- The goal is the SVD steering-vector extraction or layer-wise injection step of VSSD
  (governed by hyperparameters alpha/beta), not the construction of the perturbed
  image itself.
- The downstream model does not expose cross-attention maps over image tokens, since
  the mask cannot be derived without them.
tools:
- name: VSSD
  role: Source repository providing the pretrained multimodal LLM, environment (requirements.txt),
    and perturbed-image construction pipeline used to extract cross-attention maps
    and generate I_r
  repo: https://github.com/BGWH123/VSSD
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1109/icassp55912.2026.11463006
    title: Visual Saliency Steering Distillation for Multimodal Chain-of-Thought Reasoning
  source_content_hash: sha256:b28296b3b4a9efa6bbceb31c5d9609af71a791a8226c1e092574b13e18353a62
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/VSSD
  tools:
  - VSSD
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# attention-map-upsampling-and-gaussian-smoothing

## Summary

Constructs VSSD's negatively perturbed semantic image samples (I_r) by turning a multimodal LLM's cross-attention maps over image tokens into a denoised, top-k-thresholded binary mask that is applied to the original ScienceQA image. This captures the task-sensitive visual regions the teacher model attends to, feeding the saliency-guided distillation pipeline.

## When to use

Use when you have an image-question pair (e.g., ScienceQA) and a pretrained multimodal LLM that exposes cross-attention maps, and you need to generate a negatively perturbed image whose masked regions correspond to the question-relevant high-attention areas.

## When NOT to use

- The input is text-only (no image tokens), so no cross-attention saliency over visual regions can be aggregated and upsampled.
- The goal is the SVD steering-vector extraction or layer-wise injection step of VSSD (governed by hyperparameters alpha/beta), not the construction of the perturbed image itself.
- The downstream model does not expose cross-attention maps over image tokens, since the mask cannot be derived without them.

## Inputs

- Pretrained multimodal LLM checkpoint from the VSSD repository (github.com/BGWH123/VSSD) with cross-attention outputs exposed
- One ScienceQA image-question pair (image + question text)
- Environment installed via the VSSD requirements.txt

## Outputs

- 2D saliency map (aggregated, upsampled to image resolution, normalized, Gaussian-smoothed)
- Binary mask from top-k thresholding of the smoothed saliency map
- Negatively perturbed semantic image I_r saved as perturbed_image.png

## How to apply

Load a pretrained multimodal LLM from the VSSD repository (github.com/BGWH123/VSSD, environment installed via requirements.txt) and run a forward pass with one ScienceQA image-question pair to extract cross-attention maps over the image tokens. Aggregate the attention maps and upsample them to the image resolution to form a 2D saliency map, then normalize and apply Gaussian smoothing to suppress attention noise. Threshold the smoothed saliency map at top-k to produce a binary mask covering the image regions most relevant to the question. Apply the binary mask to the input image to generate the negatively perturbed semantic sample I_r, saved as perturbed_image.png. The rationale is that VSSD identifies and masks the image regions most relevant to the question so that downstream feature differences between original and masked images expose task-critical sensitive directions for distillation.

## Related tools

- **VSSD** (Source repository providing the pretrained multimodal LLM, environment (requirements.txt), and perturbed-image construction pipeline used to extract cross-attention maps and generate I_r) — https://github.com/BGWH123/VSSD

## Examples

```
pip install -r requirements.txt  # set up the VSSD environment (github.com/BGWH123/VSSD) for cross-attention extraction and perturbed-image generation
```

## Evaluation signals

- perturbed_image.png exists on disk and is a valid, loadable image file.
- The masked pixels of perturbed_image.png coincide with the highest cross-attention (top-k saliency) regions of the smoothed attention map.
- The saliency map has been upsampled to the full image resolution before smoothing and mask generation (2D map matching the input image dimensions).
- The binary mask is limited to the image regions most relevant to the question, i.e., high-attention regions rather than random or uniform masking.
- Comparing original vs. perturbed images yields a feature difference from which a dominant direction can be extracted, consistent with VSSD's premise that masking question-relevant regions captures task-sensitive feature directions.

## Limitations

- The top-k threshold and Gaussian smoothing parameters are not stated adaptively; VSSD's authors note that more adaptive steering-vector extraction is deferred to future work, and the mask construction similarly relies on fixed heuristic thresholding.
- Validated only on small multimodal chain-of-thought settings (ScienceQA and M3CoT); broader multimodal reasoning tasks are explicitly left to future work.
- Requires a teacher multimodal LLM whose cross-attention maps over image tokens are accessible; models without exposed attention cannot be used.

## Evidence

- [abstract] VSSD leverages the attention maps of multimodal large language models to generate perturbed images that capture task-sensitive feature directions: "VSSD leverages the attention maps of multimodal large language models to generate perturbed images that capture task-sensitive feature directions"
- [intro] VSSD leverages MLLMs to identify and mask the image regions most relevant to the question, generating negatively perturbed semantic samples.: "VSSD leverages MLLMs to identify and mask the image regions most relevant to the question, generating negatively perturbed semantic samples."
- [other] The code is available at https://github.com/BGWH123/VSSD.: "The code is available at https://github.com/BGWH123/VSSD."
