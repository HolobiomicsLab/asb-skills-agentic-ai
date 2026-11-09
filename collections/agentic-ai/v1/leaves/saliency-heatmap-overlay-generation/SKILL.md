---
name: saliency-heatmap-overlay-generation
description: Use when you need post-retrieval, model-local explanations for a CLIP-based
  composed image retrieval result — e.g. a user has run a SEARLE-based image+text
  query and must see which image regions and tokens drive the ranking.
when_to_use_negative:
- You need quantitative evaluation of retrieval quality (use NDCG@K, mAP@K, MRR or
  Recall@K instead — the paper reports no quantitative saliency criterion)
- The retrieval model is not CLIP-based (InfoCIR itself acknowledges limited CIR model
  support; gradients through a different architecture tower are required)
- You need global embedding-space analysis rather than per-query local explanation
  (use the debiased UMAP projection workflow instead)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Implements gradient backpropagation through the CLIP image tower and normalization/upsampling
    of the Grad-ECLIP attribution
- name: CLIP
  role: Provides the joint image-text embedding space whose cosine similarity is differentiated
    and whose ViT patches yield region-level attribution
- name: SEARLE
  role: Pretrained textual-inversion model encoding the reference image into a pseudo-token
    for the composed prompt, loaded via torch.hub
  repo: https://github.com/giannhskp/InfoCIR
- name: GradECLIP
  role: Gradient-based attribution technique adapted from scratch to produce saliency
    maps and token attribution for CIR
- name: InfoCIR
  role: Dashboard whose Explanation Panel renders the saliency heatmaps, token-attribution
    bars, and Rank-Δ heatmap after each search
  repo: https://github.com/giannhskp/InfoCIR
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1109/pacificvis68791.2026.00041
    title: 'InfoCIR: Multimedia Analysis for Composed Image Retrieval'
  source_content_hash: sha256:2efbdc3fedec300397f92739c6c8b460959c5aec7084c66446e4bd3b3220e5c7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giannhskp/InfoCIR
  tools:
  - CLIP
  - GradECLIP
  - InfoCIR
  - PyTorch
  - SEARLE
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# saliency-heatmap-overlay-generation

## Summary

Generates Grad-ECLIP saliency heatmaps that explain how a CLIP-based composed image retrieval (CIR) model aligns image regions with prompt tokens: gradients of the image-text cosine similarity are backpropagated through the CLIP image tower, normalized, and overlaid on the reference or retrieved images.

## When to use

Use when you need post-retrieval, model-local explanations for a CLIP-based composed image retrieval result — e.g. a user has run a SEARLE-based image+text query and must see which image regions and tokens drive the ranking.

## When NOT to use

- You need quantitative evaluation of retrieval quality (use NDCG@K, mAP@K, MRR or Recall@K instead — the paper reports no quantitative saliency criterion)
- The retrieval model is not CLIP-based (InfoCIR itself acknowledges limited CIR model support; gradients through a different architecture tower are required)
- You need global embedding-space analysis rather than per-query local explanation (use the debiased UMAP projection workflow instead)

## Inputs

- Pretrained SEARLE (or compatible) textual-inversion model on a CLIP backbone (ViT-B/32, ViT-L/14)
- Reference image and natural-language modification prompt forming a composed query
- Top-k retrieved candidate images ranked by CLIP cosine similarity
- config.py saliency settings (SALIENCY_ENABLED, SALIENCY_MAX_CANDIDATES, SALIENCY_GENERATE_TEXT_ATTRIBUTION)

## Outputs

- Saliency heatmap overlay image (e.g. saliency_map.png) per reference image and candidate
- Gradient-derived token-attribution bar chart of prompt-token influence
- Heatmaps visualized in the dashboard's Explanation Panel (F1 saliency map, F2 token attribution)

## How to apply

Load a pretrained SEARLE textual-inversion model on a CLIP backbone (e.g. ViT-B/32 or ViT-L/14 via torch.hub) and encode the reference image into a pseudo-token to form the composed prompt. Compute the joint cosine similarity s = cos(V, T) between a candidate database image's CLIP embedding V and the text embedding T of the composed prompt, then backpropagate ds/d(image pixels) through the CLIP image tower to obtain the Grad-ECLIP gradient attribution. Normalize the attribution, upsample it to image resolution, and render it as a heatmap overlay connecting salient regions with influential prompt tokens; optionally compute token-level text attribution bars alongside. In the InfoCIR implementation, saliency is configurable via config.py parameters (SALIENCY_ENABLED, SALIENCY_MAX_CANDIDATES, SALIENCY_GENERATE_REFERENCE, SALIENCY_GENERATE_CANDIDATES, SALIENCY_GENERATE_TEXT_ATTRIBUTION) so generation can be limited to the top-k results (e.g. SALIENCY_MAX_CANDIDATES = 5). Generate maps for both the reference image and top retrieved candidates so users can compare attention patterns between query and results in the Explanation Panel.

## Related tools

- **PyTorch** (Implements gradient backpropagation through the CLIP image tower and normalization/upsampling of the Grad-ECLIP attribution)
- **CLIP** (Provides the joint image-text embedding space whose cosine similarity is differentiated and whose ViT patches yield region-level attribution)
- **SEARLE** (Pretrained textual-inversion model encoding the reference image into a pseudo-token for the composed prompt, loaded via torch.hub) — https://github.com/giannhskp/InfoCIR
- **GradECLIP** (Gradient-based attribution technique adapted from scratch to produce saliency maps and token attribution for CIR)
- **InfoCIR** (Dashboard whose Explanation Panel renders the saliency heatmaps, token-attribution bars, and Rank-Δ heatmap after each search) — https://github.com/giannhskp/InfoCIR

## Examples

```
python -c "import torch; torch.hub.load('miccunifi/SEARLE', 'searle', backbone='ViT-B/32')"
```

## Evaluation signals

- saliency_map.png exists and is a valid image file for each generated map
- Highlighted regions visually correspond to the areas the model attends to when matching the composed query (expert visual review; the paper reports no quantitative saliency criterion)
- Token-attribution bars identify the prompt tokens with largest gradient magnitude, consistent with the composed prompt semantics
- Saliency generation respects SALIENCY_MAX_CANDIDATES (e.g. no more than 5 candidate maps when set to 5) and covers the reference image when SALIENCY_GENERATE_REFERENCE = True

## Limitations

- Validation is expert visual review only; the paper reports no quantitative saliency criterion
- Saliency computation on many candidates can be expensive; SALIENCY_MAX_CANDIDATES exists to limit cost, and a CUDA-capable GPU is recommended
- Generalizing the Grad-ECLIP adaptation beyond CLIP-backed CIR models is untested; InfoCIR lists extending model support as future work

## Evidence
