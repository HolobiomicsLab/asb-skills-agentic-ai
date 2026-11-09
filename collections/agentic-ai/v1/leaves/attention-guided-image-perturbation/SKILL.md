---
name: attention-guided-image-perturbation
description: 'Use when building contrastive (original vs. perturbed) visual inputs
  for a multimodal model: you have an image plus question text and need the question-relevant
  image regions masked to create a negatively perturbed sample for sensitivity or
  distillation analysis.'
when_to_use_negative:
- The analysis goal is the α/β hyperparameter sweep of the distillation loss itself
  (Table 4 / README analysis), not constructing perturbed inputs
- The task requires evaluating cosine-similarity reductions across model stages (Fig.
  3b) where the perturbed-image construction is only an upstream prerequisite, not
  the deliverable
- The model is not a multimodal LLM exposing cross-attention over image tokens, so
  no attention-derived saliency map can be extracted
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3399
- http://edamontology.org/topic_3474
tools:
- name: VSSD
  role: Source repository providing the pretrained multimodal LLM and environment
    (requirements.txt) used to extract cross-attention maps and construct the perturbed
    image
  repo: https://github.com/BGWH123/VSSD
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_004
  source_papers:
  - doi: 10.1109/icassp55912.2026.11463006
    title: Visual Saliency Steering Distillation for Multimodal Chain-of-Thought Reasoning
  source_content_hash: sha256:03e321d5358280ad471cdc3d94fc3fd68c828d05a3e5e0a97440e36a7541aebe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/VSSD
  tools:
  - Hugging Face Transformers
  - PyTorch
  - VSSD
  - pandas
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3399
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# attention-guided-image-perturbation

## Summary

Generates a negatively perturbed semantic sample I_r from a ScienceQA image-question pair by extracting cross-attention saliency from a multimodal LLM, smoothing it, thresholding at top-k into a binary mask, and masking the corresponding high-attention image regions. This perturbed image supplies the task-sensitive feature direction that VSSD later extracts via SVD.

## When to use

Use when building contrastive (original vs. perturbed) visual inputs for a multimodal model: you have an image plus question text and need the question-relevant image regions masked to create a negatively perturbed sample for sensitivity or distillation analysis.

## When NOT to use

- The analysis goal is the α/β hyperparameter sweep of the distillation loss itself (Table 4 / README analysis), not constructing perturbed inputs
- The task requires evaluating cosine-similarity reductions across model stages (Fig. 3b) where the perturbed-image construction is only an upstream prerequisite, not the deliverable
- The model is not a multimodal LLM exposing cross-attention over image tokens, so no attention-derived saliency map can be extracted

## Inputs

- One image-question pair from the ScienceQA dataset (or M3CoT)
- Pretrained multimodal LLM checkpoint from the VSSD repository (github.com/BGWH123/VSSD)
- Hyperparameters: top-k threshold for the binary mask and Gaussian-smoothing kernel for the saliency map

## Outputs

- perturbed_image.png — the negatively perturbed semantic sample I_r whose masked pixels coincide with the top-k saliency regions
- Smoothed, normalized 2D cross-attention saliency map upsampled to image resolution
- Binary mask over the question-relevant image regions

## How to apply

Load a pretrained multimodal LLM from the VSSD repository (environment installed via 'pip install -r requirements.txt') and run a forward pass with the question text and image to extract cross-attention maps over the image tokens. Aggregate the attention maps and upsample them to the image resolution to form a 2D saliency map, then normalize and apply Gaussian smoothing to suppress noise. Threshold the smoothed saliency map at top-k to produce a binary mask covering the image regions most relevant to the question, and apply that mask to the input image to generate the negatively perturbed semantic sample I_r. The rationale, per the article, is that masking the regions 'most relevant to the question' yields a perturbed image that captures task-sensitive feature directions, whose difference from the original features reveals the fine-grained visual-semantic direction VSSD distills.

## Related tools

- **VSSD** (Source repository providing the pretrained multimodal LLM and environment (requirements.txt) used to extract cross-attention maps and construct the perturbed image) — https://github.com/BGWH123/VSSD

## Examples

```
python scripts/build_perturbed_image.py --image scienceqa_sample.jpg --question q.txt --top_k 0.2 --smooth gaussian --out perturbed_image.png
```

## Evaluation signals

- perturbed_image.png exists and loads as a valid image file
- The masked pixels of perturbed_image.png coincide with the highest cross-attention (top-k saliency) regions of the smoothed attention map
- The binary mask covers only high-attention regions, not arbitrary or low-attention pixels (spot-check overlap against the normalized saliency map)
- The perturbed image differs from the original only in the masked regions — unmasked pixels remain pixel-identical
- Downstream, the original-vs-perturbed feature difference must yield a usable steering vector via SVD, as in the VSSD pipeline

## Limitations

- The article notes steering-vector extraction is not yet adaptive; more adaptive extraction is deferred to future work, so the current attention-guided masking is heuristic
- The method is scoped to small multimodal CoT settings; broader multimodal reasoning tasks are explicitly left to future work
- The binary mask depends on thresholding (top-k) and Gaussian smoothing choices; no principled threshold selection rule is given in the context

## Evidence

- [methods] After normalization and Gaussian smoothing of the attention-derived saliency map, a binary mask is generated using top-k selection and thresholding, and high-attention regions are masked from the original image to produce the perturbed version.: [evidence span withheld — anchor status: not_found]
- [abstract] VSSD leverages the attention maps of multimodal large language models to generate perturbed images that capture task-sensitive feature directions, and then applies singular value decomposition to: "VSSD leverages the attention maps of multimodal large language models to generate perturbed images that capture task-sensitive feature directions"
- [intro] VSSD leverages MLLMs to identify and mask the image regions most relevant to the question, generating negatively perturbed semantic samples.: "VSSD leverages MLLMs to identify and mask the image regions most relevant to the question, generating negatively perturbed semantic samples"
- [readme] pip install -r requirements.txt: "## Environment
bash
pip install -r requirements.txt
"
