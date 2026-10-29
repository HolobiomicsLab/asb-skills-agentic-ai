---
name: pytorch-backpropagation-through-vision-transformer
description: Use when you need to explain a CLIP-based retrieval similarity score
  at the pixel level — e.g. after a composed image retrieval search (reference image
  φ(I_ref) + text prompt) — by producing a gradient-based heatmap of influential image
  regions.
when_to_use_negative:
- The retrieval model is not CLIP-based (e.g. no joint image-text embedding space
  with a differentiable image tower) — InfoCIR itself notes future work must extend
  model support
- 'You need a quantitative saliency criterion: the paper reports no quantitative saliency
  measure, so validation is expert visual review only'
- The goal is global dataset-level structure (e.g. style-biased UMAP clusters) rather
  than local per-query explanation — use the debiased UMAP projection pipeline instead
tools:
- name: PyTorch
  role: Autograd framework used to back-propagate the cosine similarity through the
    CLIP ViT image tower to the input pixels
  repo: https://github.com/giannhskp/InfoCIR
- name: CLIP
  role: Backbone (ViT-B/32 or ViT-L/14) whose image tower gradients and joint embedding
    space produce the saliency attribution
  repo: https://github.com/giannhskp/InfoCIR
- name: SEARLE
  role: Textual-inversion model that encodes the reference image into a pseudo-token
    φ(I_ref) used in the composed prompt for the similarity score
  repo: https://github.com/giannhskp/InfoCIR
- name: GradECLIP
  role: Gradient-based explanation technique adapted from scratch in InfoCIR to generate
    the saliency maps after each search
  repo: https://github.com/giannhskp/InfoCIR
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1109/pacificvis68791.2026.00041
    title: 'InfoCIR: Multimedia Analysis for Composed Image Retrieval'
  source_content_hash: sha256:c73fb9b725f4697114391fd62264dd9ce2a686054794e9715441850bf24fc740
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giannhskp/InfoCIR
  tools:
  - CLIP
  - GradECLIP
  - PyTorch
  - SEARLE
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# pytorch-backpropagation-through-vision-transformer

## Summary

Generates Grad-ECLIP-style saliency maps by differentiating the CLIP joint cosine similarity score with respect to input image pixels, back-propagating through the CLIP ViT image tower in PyTorch. InfoCIR uses this to overlay heatmaps that explain which image regions drive composed image retrieval results.

## When to use

Use when you need to explain a CLIP-based retrieval similarity score at the pixel level — e.g. after a composed image retrieval search (reference image φ(I_ref) + text prompt) — by producing a gradient-based heatmap of influential image regions.

## When NOT to use

- The retrieval model is not CLIP-based (e.g. no joint image-text embedding space with a differentiable image tower) — InfoCIR itself notes future work must extend model support
- You need a quantitative saliency criterion: the paper reports no quantitative saliency measure, so validation is expert visual review only
- The goal is global dataset-level structure (e.g. style-biased UMAP clusters) rather than local per-query explanation — use the debiased UMAP projection pipeline instead

## Inputs

- Reference image and text prompt forming a composed query (e.g. reference image + 'wearing sunglasses')
- Pretrained CLIP backbone (ViT-B/32 or ViT-L/14) with a SEARLE textual-inversion checkpoint loaded via PyTorch Hub
- Candidate/retrieved image(s) from the top-k similarity ranking (cosine similarity between query and pre-computed database embeddings)

## Outputs

- Saliency heatmap overlaid on the reference or retrieved image, highlighting regions driving the query-result similarity
- Gradient-derived token-attribution bars showing which prompt words are most influential
- Comparable saliency maps for multiple top-k candidates (navigable in the Explanation Panel)

## How to apply

1) Load a pretrained SEARLE textual-inversion model on a CLIP backbone (e.g. ViT-B/32 or ViT-L/14 via PyTorch Hub), encode the reference image into a pseudo-token, and form the prompt 'a photo of φ(I_ref) that C'. 2) Compute the joint cosine similarity s = cos(V, T) between the candidate image embedding (from the CLIP image tower) and the CLIP text embedding of the composed prompt. 3) Back-propagate s with respect to the input image pixels through the CLIP image tower, implementing the Grad-ECLIP adaptation from scratch in PyTorch. 4) Normalize the gradient-based attribution, upsample it to image resolution, and render it as a heatmap overlay highlighting regions crucial to the query-result similarity. In the InfoCIR implementation, saliency generation is controlled via config flags (SALIENCY_ENABLED, SALIENCY_GENERATE_REFERENCE, SALIENCY_GENERATE_CANDIDATES) and can be capped with SALIENCY_MAX_CANDIDATES (e.g. limit saliency to top-5 retrieved results); token-level text attribution is produced alongside the image saliency map.

## Related tools

- **PyTorch** (Autograd framework used to back-propagate the cosine similarity through the CLIP ViT image tower to the input pixels) — https://github.com/giannhskp/InfoCIR
- **CLIP** (Backbone (ViT-B/32 or ViT-L/14) whose image tower gradients and joint embedding space produce the saliency attribution) — https://github.com/giannhskp/InfoCIR
- **SEARLE** (Textual-inversion model that encodes the reference image into a pseudo-token φ(I_ref) used in the composed prompt for the similarity score) — https://github.com/giannhskp/InfoCIR
- **GradECLIP** (Gradient-based explanation technique adapted from scratch in InfoCIR to generate the saliency maps after each search) — https://github.com/giannhskp/InfoCIR

## Examples

```
python -c "import torch; torch.hub.load('miccunifi/SEARLE', 'searle', backbone='ViT-B/32')"
```

## Evaluation signals

- The saliency map file exists and is a valid image (e.g. saliency_map.png opens and renders as a heatmap overlay)
- Highlighted regions correspond to the areas the model attends to when matching the query (expert visual review, as the paper reports no quantitative saliency criterion)
- Attribution is normalized and upsampled to image resolution before rendering the overlay
- Token-attribution bars are produced for the same composed prompt and highlight the influential prompt tokens (SALIENCY_GENERATE_TEXT_ATTRIBUTION = True)
- Saliency for multiple candidates respects the configured cap (SALIENCY_MAX_CANDIDATES, e.g. top-5 results) and covers both reference and candidate images when SALIENCY_GENERATE_REFERENCE/SALIENCY_GENERATE_CANDIDATES are enabled

## Limitations

- Validation of saliency maps is expert visual review only — the paper reports no quantitative saliency criterion
- GradECLIP adaptation must be implemented from scratch; InfoCIR supports only CLIP/SEARLE-style backends, and extending model support is deferred to future work
- Saliency generation can be computationally expensive over all candidates; the README suggests capping it (SALIENCY_MAX_CANDIDATES) and recommends a CUDA-capable GPU for optimal performance

## Evidence

- [methods] GradECLIP saliency adaptation workflow: [evidence span withheld — anchor status: not_found]
