---
name: clip-embedding-dimensionality-reduction
description: Use when you have CLIP joint image/text embeddings (e.g., ImageNet-R
  dataset, SEARLE composed query vectors) and need a 2D projection for spatial reasoning
  where naive UMAP would cluster images by artistic style instead of semantics.
when_to_use_negative:
- You need metric distances between projected points — the article treats the UMAP
  layout as qualitative/topological rather than metric
- 'The CLIP embeddings have not been debiased: raw UMAP on CLIP spaces clusters images
  by artistic style rather than semantic category'
- You need retrieval-specific quantitative signals (rank position, prompt evolution,
  Rank-Δ heatmaps) rather than spatial neighborhood exploration
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: UMAP
  role: Performs the parametric dimensionality reduction and out-of-sample transform
    onto a fixed 2D map for the debiased projection pipeline
  repo: https://github.com/giannhskp/InfoCIR
- name: CLIP
  role: Provides the joint image/text embedding space whose vectors are filtered and
    projected into 2D
- name: SEARLE
  role: Produces the composed query embedding (textual inversion of the reference
    image into a pseudo-token) that is projected into the shared 2D space
  repo: https://github.com/giannhskp/InfoCIR
- name: Plotly-Dash
  role: Renders the 2D Embedding View (Panel D) scatterplot synchronized with the
    Query Results Panel for coordinated cross-view exploration
  repo: https://github.com/giannhskp/InfoCIR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1109/pacificvis68791.2026.00041
    title: 'InfoCIR: Multimedia Analysis for Composed Image Retrieval'
  source_content_hash: sha256:74dbeaaca44c6d5f45ce0640bbe9fa73fdfc6c047634e71ff153b29ca3f5b018
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
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# clip-embedding-dimensionality-reduction

## Summary

Projects CLIP embeddings (dataset images, reference image, SEARLE composed query, and top-k retrieved results) into a shared 2D space using a debiased UMAP approach, so the Embedding View (Panel D of InfoCIR) yields semantically accurate neighborhoods rather than style-biased clusters.

## When to use

Use when you have CLIP joint image/text embeddings (e.g., ImageNet-R dataset, SEARLE composed query vectors) and need a 2D projection for spatial reasoning where naive UMAP would cluster images by artistic style instead of semantics.

## When NOT to use

- You need metric distances between projected points — the article treats the UMAP layout as qualitative/topological rather than metric
- The CLIP embeddings have not been debiased: raw UMAP on CLIP spaces clusters images by artistic style rather than semantic category
- You need retrieval-specific quantitative signals (rank position, prompt evolution, Rank-Δ heatmaps) rather than spatial neighborhood exploration

## Inputs

- ImageNet-R CLIP embeddings (high-dimensional joint image/text vectors)
- Style-debiased, contrastive-debiased, and ICA-filtered embeddings from the upstream three-stage pipeline
- SEARLE composed query embedding (reference image inverted into a pseudo-token within CLIP's text embedding space)
- Top-k retrieved image embeddings and reference image embedding

## Outputs

- 2D UMAP coordinates for dataset images, reference image, composed query, and top-k results on a fixed reference map
- Interactive scatterplot (InfoCIR Embedding View, Panel D) rendered in Plotly-Dash with zoom, pan, lasso/box selection, and class highlighting

## How to apply

Load ImageNet-R CLIP embeddings (e.g., ViT-B/32 backbone, up to DATASET_SAMPLE_SIZE = 30000 images) that have been passed through the three-stage filtering pipeline: (A) PCA-based Style Debiasing retaining dimensions with higher semantic discriminability (pca_components = 80), (B) Contrastive Debiasing aligning embeddings with class prototypes, and (C) Independent Component Analysis to separate semantic signals from stylistic noise. Fit UMAP on the filtered high-dimensional vectors exploiting UMAP's parametric formulation for stable out-of-sample transforms, using the tuned parameters from NEW_UMAP_CONFIG (n_neighbors = 25, min_dist = 0.1, spread = 55.0, target_weight = 0.3, local_connectivity = 3.0, n_epochs = 1200). At query time, apply UMAP's out-of-sample transform to embed the reference image, the SEARLE composed query pseudo-token embedding, and the top-k retrieved image embeddings into the same fixed 2D map in real time. Render the coordinates in the Plotly-Dash Embedding View (Panel D), synchronized with the Query Results Panel, treating the layout as qualitative/topological rather than metric.

## Related tools

- **UMAP** (Performs the parametric dimensionality reduction and out-of-sample transform onto a fixed 2D map for the debiased projection pipeline) — https://github.com/giannhskp/InfoCIR
- **CLIP** (Provides the joint image/text embedding space whose vectors are filtered and projected into 2D)
- **SEARLE** (Produces the composed query embedding (textual inversion of the reference image into a pseudo-token) that is projected into the shared 2D space) — https://github.com/giannhskp/InfoCIR
- **Plotly-Dash** (Renders the 2D Embedding View (Panel D) scatterplot synchronized with the Query Results Panel for coordinated cross-view exploration) — https://github.com/giannhskp/InfoCIR

## Examples

```
cd cir_app && python run.py  # first run auto-generates CLIP embeddings and debiased UMAP projections; serve at http://localhost:8051
```

## Evaluation signals

- The projection yields denser, well-separated semantic clusters (semantic category verification per DG1) instead of clusters grouped by artistic style
- The reference image, composed query embedding, and top-k results all appear on the same fixed 2D map via the out-of-sample transform, not a refitted map
- Scatterplot enforces MAX_IMAGES_ON_SCATTERPLOT = 100 and supports zoom, lasso/box selection, and class highlighting that stay consistent with histogram filtering
- UMAP parameters match the configured NEW_UMAP_CONFIG values (pca_components = 80, n_neighbors = 25, min_dist = 0.1, spread = 55.0, n_epochs = 1200)
- Selected gallery images highlight correctly on the scatterplot, confirming cross-widget synchronization

## Limitations

- Standard UMAP on CLIP-based spaces suffers from style bias, clustering images by artistic style rather than semantic categories
- The 2D layout is qualitative/topological and must not be read as metric
- Scalability beyond current dataset sizes is acknowledged as future work; MAX_IMAGES_ON_SCATTERPLOT = 100 caps points shown

## Evidence

- [intro] standard dimensionality reduction techniques like UMAP can suffer from style bias in CLIP-based spaces, where images are clustered by artistic style: "standard dimensionality reduction techniques like UMAP can suffer from style bias in CLIP-based spaces, where images are clustered by artistic style"
- [abstract] projects the top-k results into a low-dimensional space using Uniform Manifold Approximation and Projection (UMAP) for spatial reasoning: "projects the top-k results into a low-dimensional space using Uniform Manifold Approximation and Projection (UMAP) for spatial reasoning"
- [methods] (A) Style Debiasing via PCA-based filtering to retain dimensions with higher semantic discriminability: "(A) Style Debiasing via PCA-based filtering to retain dimensions with higher semantic discriminability"
- [readme] UMAP Projections: 2D visualization of high-dimensional image embeddings: "UMAP Projections: 2D visualization of high-dimensional image embeddings"
- [readme] 'pca_components': 80,                         # PCA preprocessing: "'pca_components': 80,                         # PCA preprocessing"
