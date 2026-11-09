---
name: style-debiased-embedding-projection
description: 'Use when visualizing CLIP-based composed image retrieval results in
  2D: if neighborhoods are grouped by artistic style instead of semantics, apply PCA-based
  style debiasing, contrastive debiasing, and ICA before fitting UMAP.'
when_to_use_negative:
- When the goal is metric distance reading (e.g. computing exact similarity values
  from the 2D layout) — the projection is explicitly qualitative/topological, not
  metric
- 'When embeddings have not been style-debiased: vanilla UMAP on raw CLIP space suffers
  from style bias, clustering images by artistic style rather than semantic categories'
- When a quick qualitative check of unimodal (text-only or image-only) embeddings
  suffices — the pipeline targets composed image retrieval, where reference, query,
  and retrieved images must share one space
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_3474
tools:
- name: UMAP
  role: Low-dimensional projection engine; fitted parametrically on debiased embeddings
    and applied as an out-of-sample transform to place reference, query, and top-k
    results on one fixed 2D map
  repo: https://github.com/giannhskp/InfoCIR
- name: CLIP
  role: Source of the joint image/text embedding space whose ViT-B/32 embeddings are
    debiased and projected
- name: SEARLE
  role: CIR back-end producing the composed query embedding (textual inversion pseudo-token)
    that is projected into the shared 2D space
- name: Plotly-Dash
  role: Renders the Embedding View (Panel D) scatterplot with zoom, lasso/box selection,
    and synchronization with the Query Results Panel
  repo: https://github.com/giannhskp/InfoCIR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1109/pacificvis68791.2026.00041
    title: 'InfoCIR: Multimedia Analysis for Composed Image Retrieval'
  source_content_hash: sha256:2eed6bd7c93af638b80dd879fc3c7a91564054a465caca8fd9b2cf13b103d5d0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giannhskp/InfoCIR
  tools:
  - CLIP
  - Plotly-Dash
  - SEARLE
  - UMAP
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# style-debiased-embedding-projection

## Summary

A supervised metric projection pipeline that uses a debiased UMAP approach to place reference images, SEARLE-composed query embeddings, and top-k retrieved images into one shared 2D space, specifically designed to overcome the style bias that makes vanilla UMAP cluster CLIP embeddings by artistic style rather than semantic category.

## When to use

Use when visualizing CLIP-based composed image retrieval results in 2D: if neighborhoods are grouped by artistic style instead of semantics, apply PCA-based style debiasing, contrastive debiasing, and ICA before fitting UMAP.

## When NOT to use

- When the goal is metric distance reading (e.g. computing exact similarity values from the 2D layout) — the projection is explicitly qualitative/topological, not metric
- When embeddings have not been style-debiased: vanilla UMAP on raw CLIP space suffers from style bias, clustering images by artistic style rather than semantic categories
- When a quick qualitative check of unimodal (text-only or image-only) embeddings suffices — the pipeline targets composed image retrieval, where reference, query, and retrieved images must share one space

## Inputs

- Class Projection Preservation Pipeline-filtered ImageNet-R CLIP embeddings (post PCA Style Debiasing, Contrastive Debiasing, and ICA)
- Reference image embedding from the composed query (image + text prompt)
- SEARLE composed query embedding (pseudo-token in CLIP text embedding space)
- Top-k retrieved image embeddings ranked by cosine similarity
- NEW_UMAP_CONFIG hyperparameters (pca_components=80, n_neighbors=25, min_dist=0.1, spread=55.0, target_weight=0.3, n_epochs=1200)

## Outputs

- Fixed 2D UMAP reference map over the filtered dataset embeddings
- Out-of-sample 2D coordinates for the reference image, composed query, and top-k results
- Interactive Embedding View (Panel D) rendered in Plotly-Dash, synchronized with the Query Results Panel for cross-view brushing and selection

## How to apply

Start from ImageNet-R CLIP embeddings (ViT-B/32 backbone) that have passed the three upstream filtering stages: (A) Style Debiasing via PCA-based filtering (pca_components=80) to retain dimensions with higher semantic discriminability, (B) Contrastive Debiasing, which encourages embeddings to align more closely with their class prototypes, and (C) Independent Component Analysis to separate semantic signals from stylistic noise. Fit UMAP on the filtered high-dimensional vectors using the README's parameters (n_neighbors=25, min_dist=0.1, spread=55.0, target_weight=0.3, local_connectivity=3.0, n_epochs=1200), exploiting UMAP's parametric formulation so a fixed reference map supports stable out-of-sample transforms. Apply the out-of-sample transform in real time to the reference image embedding, the SEARLE composed query embedding (textual-inversion pseudo-token in CLIP's text space), and the top-k retrieved image embeddings, then render the 2D coordinates in the central Embedding View (Panel D) with Plotly-Dash, synchronized with the Query Results Panel. Treat the layout as qualitative/topological rather than metric: the decision point is whether the composed query lands within the intended semantic cluster (DG1), not exact inter-point distances.

## Related tools

- **UMAP** (Low-dimensional projection engine; fitted parametrically on debiased embeddings and applied as an out-of-sample transform to place reference, query, and top-k results on one fixed 2D map) — https://github.com/giannhskp/InfoCIR
- **CLIP** (Source of the joint image/text embedding space whose ViT-B/32 embeddings are debiased and projected)
- **SEARLE** (CIR back-end producing the composed query embedding (textual inversion pseudo-token) that is projected into the shared 2D space)
- **Plotly-Dash** (Renders the Embedding View (Panel D) scatterplot with zoom, lasso/box selection, and synchronization with the Query Results Panel) — https://github.com/giannhskp/InfoCIR

## Examples

```
cd cir_app && python run.py  # first run generates CLIP embeddings and debiased UMAP projections on http://localhost:8051
```

## Evaluation signals

- Projected space shows denser, well-separated semantic clusters so a user can visually confirm whether the composed query has landed within the intended semantic category (DG1)
- Neighborhoods are grouped by semantic category rather than artistic style — the explicit failure mode of undebiased UMAP in CLIP-based spaces
- Reference image, composed query, and all top-k retrieved images appear on the same fixed 2D map via out-of-sample transform (no refitting per query)
- Panel D scatterplot stays coordinated with the Query Results Panel: selecting points/images highlights the corresponding items in both views
- Layout is interpreted qualitatively/topologically; verify no claim of metric distance accuracy is made from the 2D coordinates

## Limitations

- Standard UMAP suffers from style bias in CLIP-based spaces, clustering images by artistic style rather than semantic categories — hence debiasing is mandatory, not optional
- The 2D layout is qualitative/topological; inter-point distances should not be read as similarities
- Scatterplot is limited to MAX_IMAGES_ON_SCATTERPLOT = 100 images on the plot at once
- Scalability to very large datasets is acknowledged as future work; WizMap-style quadtree summaries are cited as prior art for million-point scaling

## Evidence

- [methods] three-stage filtering: [evidence span withheld — anchor status: not_found]
