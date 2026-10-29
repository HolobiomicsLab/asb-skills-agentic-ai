---
name: pytorch-autograd-differentiation
description: Use when you need local, token-level explanations of a composed image
  retrieval (CIR) query — i.e., which words in the text prompt (after SEARLE textual
  inversion of the reference image) drive the similarity ranking of a user-selected
  ideal image.
when_to_use_negative:
- The goal is dataset-level structure discovery rather than local explanation of a
  single composed query — use the debiased UMAP embedding view or histogram/wordcloud
  panels instead
- The retrieval backend is not differentiable through PyTorch autograd (e.g., a frozen
  pipeline exposing only pre-computed embeddings), since gradients of the similarity
  score w.r.t. input tokens cannot be obtained
- The prompt has not been composed with SEARLE-style textual inversion, so the tokenization/pseudo-token
  pathway this attribution pipeline depends on does not exist
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Autograd engine used to back-propagate the gradient of the cosine similarity
    score through the CLIP text encoder to obtain per-token gradients
- name: CLIP
  role: Vision-language model whose text encoder (ViT-B/32 default) embeds the composed
    prompt and through which gradients are differentiated
- name: SEARLE
  role: Zero-shot textual inversion model that transforms the reference image into
    a pseudo-token within CLIP's text embedding space before token attribution
- name: InfoCIR
  role: Dashboard that couples the token attribution bar chart (F2) to a saliency
    map (F1) and an Ideal-anchored Rank-Δ heatmap (F3) in the Explanation Panel
  repo: https://github.com/giannhskp/InfoCIR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1109/pacificvis68791.2026.00041
    title: 'InfoCIR: Multimedia Analysis for Composed Image Retrieval'
  source_content_hash: sha256:55d2fa3c9922ba825ff995e2529e250d196a07c34c65fa1de9c1fefc2496de07
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giannhskp/InfoCIR
  tools:
  - CLIP
  - InfoCIR
  - PyTorch
  - SEARLE
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# pytorch-autograd-differentiation

## Summary

Uses PyTorch autograd to back-propagate the cosine similarity between a SEARLE-composed query embedding and an ideal target image through CLIP's text encoder, yielding per-token signed influence scores that InfoCIR renders as the Explanation Panel's token attribution bar chart (F2).

## When to use

Use when you need local, token-level explanations of a composed image retrieval (CIR) query — i.e., which words in the text prompt (after SEARLE textual inversion of the reference image) drive the similarity ranking of a user-selected ideal image.

## When NOT to use

- The goal is dataset-level structure discovery rather than local explanation of a single composed query — use the debiased UMAP embedding view or histogram/wordcloud panels instead
- The retrieval backend is not differentiable through PyTorch autograd (e.g., a frozen pipeline exposing only pre-computed embeddings), since gradients of the similarity score w.r.t. input tokens cannot be obtained
- The prompt has not been composed with SEARLE-style textual inversion, so the tokenization/pseudo-token pathway this attribution pipeline depends on does not exist

## Inputs

- Reference image (user upload)
- Natural language modification prompt (e.g., 'make it red', 'wearing a red collar')
- SEARLE pre-trained model (ViT-B/32 or ViT-L/14 backbone via PyTorch Hub)
- CLIP text encoder weights (ViT-B/32 default in config.py)
- Embedding of the user-selected ideal target image

## Outputs

- token_attribution.csv: one row per prompt token with (token, signed attribution score)
- Token attribution bar chart (panel F2) rendered in the Plotly-Dash Explanation Panel

## How to apply

Compose the query by encoding the reference image with SEARLE, which performs textual inversion into a pseudo-token within CLIP's text embedding space (ViT-B/32 backbone by default), then tokenize the full composed prompt with CLIP's text encoder. Compute the cosine similarity s = sim(f_c(I_ref, T'), f_i(I*)) between the composed text embedding and the embedding of the user-selected ideal target image. Back-propagate the gradient of s with respect to each input token using PyTorch autograd through the CLIP text encoder (Grad-ECLIP-style, adapted to CLIP by back-propagating through both image and text towers). Aggregate per-token gradient magnitudes into a single signed influence score per token, reflecting each word's contribution to the retrieval match. Export the rows as a (token, attribution score) table in token_attribution.csv; in the InfoCIR dashboard this is enabled via SALIENCY_GENERATE_TEXT_ATTRIBUTION = True and the bars are coupled to a Rank-Δ heatmap (F3).

## Related tools

- **PyTorch** (Autograd engine used to back-propagate the gradient of the cosine similarity score through the CLIP text encoder to obtain per-token gradients)
- **CLIP** (Vision-language model whose text encoder (ViT-B/32 default) embeds the composed prompt and through which gradients are differentiated)
- **SEARLE** (Zero-shot textual inversion model that transforms the reference image into a pseudo-token within CLIP's text embedding space before token attribution)
- **InfoCIR** (Dashboard that couples the token attribution bar chart (F2) to a saliency map (F1) and an Ideal-anchored Rank-Δ heatmap (F3) in the Explanation Panel) — https://github.com/giannhskp/InfoCIR

## Examples

```
SALIENCY_GENERATE_TEXT_ATTRIBUTION = True  # set in cir_app/src/config.py, then: python run.py
```

## Evaluation signals

- token_attribution.csv exists and contains exactly one row per prompt token
- All exported attribution magnitudes are non-negative
- Top-ranked tokens correspond to high-influence terms consistent with the scenario — e.g., stronger activation on 'red' after prompt refinement
- The token attribution bars (F2) agree with the saliency map (F1) and Rank-Δ heatmap (F3) for the same query/ideal pair
- Changing the composed prompt produces a corresponding change in the signed token ordering, confirming gradients flow through the text encoder

## Limitations

- Attribution is gradient-based and tied to the CLIP/SEARLE backends (ViT-B/32 or ViT-L/14); other CIR models are not yet supported, and extending model support is acknowledged as future work
- Saliency and text attribution computation can be limited to the top-k candidates via SALIENCY_MAX_CANDIDATES (e.g., 5) to bound cost; a CUDA-capable GPU is recommended
- Explanations are local to a single composed query and selected ideal image; the system only partially bridges the gap between opaque model internals and user-centered retrieval workflows

## Evidence

- [methods] Grad-ECLIP adaptation: [evidence span withheld — anchor status: not_found]
