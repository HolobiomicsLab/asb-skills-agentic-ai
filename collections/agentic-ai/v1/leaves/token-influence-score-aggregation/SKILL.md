---
name: token-influence-score-aggregation
description: Use when you need to explain which words of a composed text prompt (reference
  image + natural-language modification, e.g. 'make it red') drive the retrieval match,
  by quantifying each token's gradient-derived contribution to the CLIP similarity
  score.
when_to_use_negative:
- When the goal is image-side explanation of which visual regions drive retrieval
  — use GradECLIP saliency maps (F1) instead
- When the retrieval backend is not CLIP-embedding-based (e.g. non-SEARLE/OTI CIR
  systems without a differentiable text tower), since gradients through the CLIP text
  encoder are required
- When a global model-level explanation is needed rather than local, prompt-specific
  token influence
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: CLIP
  role: Vision-language model whose text encoder is back-propagated through to obtain
    per-token gradients
- name: SEARLE
  role: Zero-shot textual inversion model transforming the reference image into a
    pseudo-token within CLIP's text embedding space
- name: PyTorch
  role: Autograd engine used to compute gradients of the similarity score w.r.t. each
    input token
- name: InfoCIR
  role: Dashboard that visualizes the aggregated token attribution as the F2 bar chart
  repo: https://github.com/giannhskp/InfoCIR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1109/pacificvis68791.2026.00041
    title: 'InfoCIR: Multimedia Analysis for Composed Image Retrieval'
  source_content_hash: sha256:c50eb90241c7a821083f7804453357e2c36fa92a991fc665cc05a65d0126a163
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giannhskp/InfoCIR
  tools:
  - CLIP
  - InfoCIR
  - PyTorch
  - SEARLE
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# token-influence-score-aggregation

## Summary

Gradient-based per-token attribution for composed image retrieval: back-propagate the cosine similarity score through the CLIP text encoder to compute signed influence scores for each sub-word, which InfoCIR renders as the token attribution bar chart (Explanation Panel F2).

## When to use

Use when you need to explain which words of a composed text prompt (reference image + natural-language modification, e.g. 'make it red') drive the retrieval match, by quantifying each token's gradient-derived contribution to the CLIP similarity score.

## When NOT to use

- When the goal is image-side explanation of which visual regions drive retrieval — use GradECLIP saliency maps (F1) instead
- When the retrieval backend is not CLIP-embedding-based (e.g. non-SEARLE/OTI CIR systems without a differentiable text tower), since gradients through the CLIP text encoder are required
- When a global model-level explanation is needed rather than local, prompt-specific token influence

## Inputs

- Composed query: reference image + natural language prompt (e.g. 'wearing a red collar')
- SEARLE pseudo-token encoding of the reference image (ViT-B/32 or ViT-L/14 backbone)
- Embedding of a user-selected ideal target image
- CLIP-tokenized composed prompt

## Outputs

- token_attribution.csv: one row per prompt token with (token, attribution score)
- Token attribution bar chart in the Explanation Panel (F2)
- Coupled Ideal-anchored Rank-Δ heatmap (F3) showing ranking changes per prompt variant

## How to apply

Load the composed query and encode the reference image with SEARLE, which performs textual inversion into a pseudo-token within CLIP's text embedding space, then tokenize the full composed prompt with the CLIP text encoder (backbone e.g. ViT-B/32 or ViT-L/14). Compute the cosine similarity s = sim(f_c(I_ref, T'), f_i(I*)) between the composed text embedding and the embedding of a user-selected ideal target image. Back-propagate the gradient of s with respect to each input token using PyTorch autograd through the CLIP text encoder, following the Grad-ECLIP approach adapted to CLIP's image and text towers. Aggregate the per-token gradients into a single signed influence score reflecting each word's contribution to the retrieval match, and export rows as a (token, attribution score) table. Enable this in the InfoCIR dashboard via SALIENCY_GENERATE_TEXT_ATTRIBUTION = True in config.py, coupling the resulting bar chart to the Ideal-anchored Rank-Δ heatmap (F3) for prompt-impact verification.

## Related tools

- **CLIP** (Vision-language model whose text encoder is back-propagated through to obtain per-token gradients)
- **SEARLE** (Zero-shot textual inversion model transforming the reference image into a pseudo-token within CLIP's text embedding space)
- **PyTorch** (Autograd engine used to compute gradients of the similarity score w.r.t. each input token)
- **InfoCIR** (Dashboard that visualizes the aggregated token attribution as the F2 bar chart) — https://github.com/giannhskp/InfoCIR

## Examples

```
SALIENCY_GENERATE_TEXT_ATTRIBUTION = True  # set in cir_app/src/config.py, then: cd cir_app && python run.py
```

## Evaluation signals

- token_attribution.csv exists and contains exactly one row per prompt token
- Attribution magnitudes are non-negative
- Top-ranked tokens correspond to high-influence terms consistent with the paper's scenario, e.g. stronger activation on 'red' after prompt refinement
- SALIENCY_GENERATE_TEXT_ATTRIBUTION is enabled in config.py and attribution bars render in the Explanation Panel (F2)
- Attribution scores are signed, reflecting each word's contribution to the retrieval match

## Limitations

- Limited CIR model support: gradients require a differentiable CLIP-compatible text tower; future work must extend model support
- Saliency and text attribution computation can be restricted to top candidates (SALIENCY_MAX_CANDIDATES) and benefits from a CUDA-capable GPU (8GB+ RAM, 16GB+ for large datasets)
- Scalability and personalization are acknowledged limitations deferred to future work

## Evidence

- [methods] back-propagating gradient of similarity through CLIP text encoder: [evidence span withheld — anchor status: not_found]
