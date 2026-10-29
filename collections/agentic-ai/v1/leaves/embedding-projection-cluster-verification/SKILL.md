---
name: embedding-projection-cluster-verification
description: Use when high-dimensional joint image–text embeddings (e.g., CLIP ViT-B/32
  on ImageNet-R) must be visualized for retrieval diagnosis and raw UMAP clusters
  images by style instead of semantics.
when_to_use_negative:
- When the analysis requires precise metric distances between points — the UMAP layout
  is explicitly qualitative/topological rather than metric
- When raw (unfiltered) CLIP embeddings are all that is available and style bias has
  not been addressed, since plain UMAP clusters images by artistic style
- When the goal is model-agnostic unimodal explainability rather than retrieval-specific
  embedding inspection (rank position, prompt evolution, ideal-image anchors)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: UMAP
  role: Fits the parametric, debiased low-dimensional projection and applies out-of-sample
    transforms to a fixed 2D map
- name: CLIP
  role: Supplies the joint image–text embeddings that are debiased and projected
- name: SEARLE
  role: Produces the composed query embedding (textual inversion pseudo-token) that
    is embedded into the shared 2D space
- name: Plotly-Dash
  role: Renders the Embedding View (Panel D) synchronized with the Query Results Panel
  repo: https://github.com/giannhskp/InfoCIR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1109/pacificvis68791.2026.00041
    title: 'InfoCIR: Multimedia Analysis for Composed Image Retrieval'
  source_content_hash: sha256:9b6bfa9c857e4433ba62356937d8d84010eb51c83aafd22ab5ecb045d0100f92
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giannhskp/InfoCIR
  tools:
  - CLIP
  - Plotly-Dash
  - SEARLE
  - UMAP
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Embedding Projection Cluster Verification

## Summary

Projects CLIP embeddings (reference image, SEARLE composed query, and top-k retrieved images) into a shared 2D UMAP space after applying a three-stage debiasing pipeline (PCA-based Style Debiasing, Contrastive Debiasing, ICA) so that neighborhoods reflect semantic categories rather than artistic style.

## When to use

Use when high-dimensional joint image–text embeddings (e.g., CLIP ViT-B/32 on ImageNet-R) must be visualized for retrieval diagnosis and raw UMAP clusters images by style instead of semantics.

## When NOT to use

- When the analysis requires precise metric distances between points — the UMAP layout is explicitly qualitative/topological rather than metric
- When raw (unfiltered) CLIP embeddings are all that is available and style bias has not been addressed, since plain UMAP clusters images by artistic style
- When the goal is model-agnostic unimodal explainability rather than retrieval-specific embedding inspection (rank position, prompt evolution, ideal-image anchors)

## Inputs

- ImageNet-R CLIP embeddings filtered by the Class Projection Preservation Pipeline (Style Debiasing, Contrastive Debiasing, ICA)
- Reference image embedding
- SEARLE composed query embedding (pseudo-token within CLIP's text embedding space)
- Top-k retrieved image embeddings from cosine-similarity ranking

## Outputs

- Fixed 2D UMAP reference map supporting out-of-sample transforms
- 2D coordinates for reference image, composed query, and top-k results, rendered as the Embedding View (Panel D)

## How to apply

First pass the CLIP embeddings through the three upstream filtering stages: PCA-based Style Debiasing (retain dimensions with higher semantic discriminability), Contrastive Debiasing (align embeddings with class prototypes), and ICA to separate semantic signal from stylistic noise. Then fit UMAP on the filtered vectors using its parametric formulation so out-of-sample transforms are stable onto a fixed reference map, with parameters such as pca_components=80, n_neighbors=25, min_dist=0.1, spread=55.0, target_weight=0.3, local_connectivity=3.0, and n_epochs=1200. Apply the out-of-sample transform in real time to embed the reference image, the SEARLE composed query embedding, and the top-k retrieved embeddings into the same fixed 2D map, then render the coordinates in the Embedding View (Panel D) synchronized with the Query Results Panel via Plotly-Dash. Treat the layout as qualitative/topological, not metric.

## Related tools

- **UMAP** (Fits the parametric, debiased low-dimensional projection and applies out-of-sample transforms to a fixed 2D map)
- **CLIP** (Supplies the joint image–text embeddings that are debiased and projected)
- **SEARLE** (Produces the composed query embedding (textual inversion pseudo-token) that is embedded into the shared 2D space)
- **Plotly-Dash** (Renders the Embedding View (Panel D) synchronized with the Query Results Panel) — https://github.com/giannhskp/InfoCIR

## Examples

```
NEW_UMAP_CONFIG = {'pca_components': 80, 'n_neighbors': 25, 'min_dist': 0.1, 'spread': 55.0, 'target_weight': 0.3, 'local_connectivity': 3.0, 'n_epochs': 1200}
```

## Evaluation signals

- Projected space shows denser, well-separated semantic clusters (goal DG1) so a user can visually confirm the composed query landed within the intended semantic category
- Style bias is absent: images cluster by semantic category rather than artistic style in the 2D map
- Out-of-sample transform places the reference image, composed query, and top-k results into the same fixed 2D map consistently across queries (parametric/stable transform)
- Highlighted points in the Embedding View match the top-k ranking shown in the Query Results Panel (coordinated cross-view synchronization)

## Limitations

- The 2D layout is qualitative/topological and must not be read as metric distances
- Standard UMAP in CLIP-based spaces suffers from style bias, clustering images by artistic style rather than semantic categories, hence the required upstream debiasing stages
- Scalability of the projection pipeline is acknowledged as future work in the paper's discussion

## Evidence
