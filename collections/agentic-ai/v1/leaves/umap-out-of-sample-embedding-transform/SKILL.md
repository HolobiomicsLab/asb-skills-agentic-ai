---
name: umap-out-of-sample-embedding-transform
description: Use when you must visualize a composed image retrieval state on a shared
  2D map — i.e., you have CLIP-space embeddings for a dataset, a reference image,
  a composed query, and top-k results, and raw UMAP clusters by artistic style rather
  than semantics.
when_to_use_negative:
- The input embeddings are raw, unfiltered CLIP vectors — UMAP will cluster by artistic
  style rather than semantic categories unless the three-stage debiasing pipeline
  (Style Debiasing, Contrastive Debiasing, ICA) is applied first.
- The analysis requires faithful metric distances between projected points — the resulting
  2D layout is qualitative/topological, not metric.
- You need exact per-image attribution or rank explanations rather than a global semantic
  overview — use the saliency/token-attribution or Rank-Δ heatmap panels instead.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: UMAP
  role: Fitted on filtered embeddings; its parametric out-of-sample transform projects
    reference image, composed query, and top-k results onto the fixed 2D map
  repo: https://github.com/giannhskp/InfoCIR
- name: CLIP
  role: Provides the joint image-text embedding space whose (filtered) vectors are
    the input to the UMAP fit and transform
  repo: https://github.com/giannhskp/InfoCIR
- name: SEARLE
  role: Produces the composed query embedding (textual inversion of the reference
    image into a pseudo-token) that is projected via the out-of-sample transform
  repo: https://github.com/giannhskp/InfoCIR
- name: Plotly-Dash
  role: Renders the 2D coordinates as the central Embedding View (Panel D) scatterplot,
    synchronized with the Query Results Panel
  repo: https://github.com/giannhskp/InfoCIR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1109/pacificvis68791.2026.00041
    title: 'InfoCIR: Multimedia Analysis for Composed Image Retrieval'
  source_content_hash: sha256:d47eb32e7799dbaa9acd6fd646a6990adaed8549e916b0d49ab985e9ca343f9e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giannhskp/InfoCIR
  tools:
  - CLIP
  - Plotly-Dash
  - SEARLE
  - UMAP
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# umap-out-of-sample-embedding-transform

## Summary

A supervised metric projection skill that fits a debiased UMAP model on style-filtered CLIP embeddings and then applies UMAP's out-of-sample transform to place reference images, SEARLE composed query embeddings, and top-k retrieval results into one fixed 2D map for spatial reasoning in InfoCIR's Embedding View (Panel D).

## When to use

Use when you must visualize a composed image retrieval state on a shared 2D map — i.e., you have CLIP-space embeddings for a dataset, a reference image, a composed query, and top-k results, and raw UMAP clusters by artistic style rather than semantics.

## When NOT to use

- The input embeddings are raw, unfiltered CLIP vectors — UMAP will cluster by artistic style rather than semantic categories unless the three-stage debiasing pipeline (Style Debiasing, Contrastive Debiasing, ICA) is applied first.
- The analysis requires faithful metric distances between projected points — the resulting 2D layout is qualitative/topological, not metric.
- You need exact per-image attribution or rank explanations rather than a global semantic overview — use the saliency/token-attribution or Rank-Δ heatmap panels instead.

## Inputs

- Class Projection Preservation Pipeline-filtered ImageNet-R CLIP embeddings (after PCA Style Debiasing, Contrastive Debiasing, and ICA)
- Reference image CLIP embedding
- SEARLE composed query embedding (reference image inverted into a pseudo-token in CLIP's text embedding space)
- Top-k retrieved image embeddings ranked by cosine similarity
- NEW_UMAP_CONFIG parameters (pca_components, n_neighbors, min_dist, spread, target_weight, local_connectivity, n_epochs, alternative_projection)

## Outputs

- 2D coordinates for reference image, composed query, and top-k results on a fixed shared UMAP map
- Interactive Embedding View (Panel D) scatterplot rendered with Plotly-Dash, synchronized with the Query Results Panel

## How to apply

First pass the ImageNet-R CLIP embeddings through the three-stage filtering pipeline: (A) PCA-based Style Debiasing to retain dimensions with higher semantic discriminability, (B) Contrastive Debiasing to align embeddings with class prototypes, and (C) ICA to separate semantic signals from stylistic noise; this is required because vanilla UMAP suffers from style bias in CLIP-based spaces. Fit UMAP on these filtered high-dimensional vectors, exploiting UMAP's parametric formulation so the fitted map supports stable out-of-sample transforms; in the InfoCIR implementation the UMAP enhancement is configured via NEW_UMAP_CONFIG in src/config.py (e.g., pca_components=80, n_neighbors=25, min_dist=0.1, spread=55.0, target_weight=0.3, local_connectivity=3.0, n_epochs=1200, alternative_projection='ica'). At query time, apply the out-of-sample transform to the reference image embedding, the SEARLE composed query embedding, and the top-k retrieved image embeddings, projecting all three onto the same fixed 2D map in real time. Render the resulting coordinates as the central Embedding View (Panel D) in Plotly-Dash, synchronized with the Query Results Panel for coordinated cross-view exploration. Treat the layout as qualitative/topological rather than metric when interpreting distances.

## Related tools

- **UMAP** (Fitted on filtered embeddings; its parametric out-of-sample transform projects reference image, composed query, and top-k results onto the fixed 2D map) — https://github.com/giannhskp/InfoCIR
- **CLIP** (Provides the joint image-text embedding space whose (filtered) vectors are the input to the UMAP fit and transform) — https://github.com/giannhskp/InfoCIR
- **SEARLE** (Produces the composed query embedding (textual inversion of the reference image into a pseudo-token) that is projected via the out-of-sample transform) — https://github.com/giannhskp/InfoCIR
- **Plotly-Dash** (Renders the 2D coordinates as the central Embedding View (Panel D) scatterplot, synchronized with the Query Results Panel) — https://github.com/giannhskp/InfoCIR

## Examples

```
NEW_UMAP_CONFIG = {'pca_components': 80, 'n_neighbors': 25, 'min_dist': 0.1, 'alternative_projection': 'ica'}  # in cir_app/src/config.py; then python run.py
```

## Evaluation signals

- The projection yields denser, well-separated semantic clusters, confirming style bias was overcome.
- A user can rapidly and visually confirm whether the composed query embedding has landed within the intended semantic category (DG1).
- Out-of-sample points (reference, query, top-k) appear on the same fixed 2D map as the fitted dataset without refitting UMAP per query.
- The UMAP configuration matches the documented NEW_UMAP_CONFIG values (e.g., pca_components=80, n_neighbors=25, min_dist=0.1, alternative_projection='ica').
- Distances in the 2D view are interpreted only topologically, never as metric similarities.

## Limitations

- Standard dimensionality reduction like UMAP suffers from style bias in CLIP-based spaces, clustering images by artistic style instead of semantics, so the debiasing pipeline is mandatory.
- The layout is qualitative/topological rather than metric — Euclidean distances in the 2D map are not faithful similarity measures.
- Scalability is an acknowledged limitation deferred to future work in InfoCIR.

## Evidence
