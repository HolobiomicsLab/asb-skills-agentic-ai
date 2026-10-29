---
name: cross-view-linked-visualization-rendering
description: Use when a visual analytics dashboard must present one shared retrieval
  state across multiple linked panels (2D embedding scatterplot, top-k image gallery,
  class histogram/wordcloud) so users can confirm via coordinated interaction where
  a composed query landed.
when_to_use_negative:
- You need metric/geodesic distances read off the 2D plot — InfoCIR's debiased UMAP
  layout is explicitly qualitative/topological, not metric
- You need to visualize a raw, unfiltered embedding space — standard UMAP on CLIP
  spaces clusters by artistic style rather than semantic categories, so apply the
  debiasing pipeline first
- The dataset exceeds the scatterplot limit (MAX_IMAGES_ON_SCATTERPLOT=100); use aggregated
  views (histogram/wordcloud) instead of densifying the scatterplot
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: UMAP
  role: Provides the debiased, out-of-sample 2D coordinates that position reference,
    query, and top-k points in the shared Embedding View
  repo: https://github.com/giannhskp/InfoCIR
- name: Plotly-Dash
  role: Modular dashboard framework rendering the Embedding View scatterplot, Query
    Results Panel, histogram, and wordcloud with cross-panel interaction callbacks
  repo: https://github.com/giannhskp/InfoCIR
- name: SEARLE
  role: Produces the composed query embedding that is projected into the shared 2D
    space alongside the reference and top-k images
  repo: https://github.com/giannhskp/InfoCIR
- name: CLIP
  role: Supplies the joint vision-language embeddings of the ImageNet-R dataset that
    are filtered and projected for the linked views
  repo: https://github.com/giannhskp/InfoCIR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1109/pacificvis68791.2026.00041
    title: 'InfoCIR: Multimedia Analysis for Composed Image Retrieval'
  source_content_hash: sha256:925275e14971c600f3f877e340c56d71d91a6655907d6b8534a95b10974dcada
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giannhskp/InfoCIR
  tools:
  - CLIP
  - Plotly-Dash
  - SEARLE
  - UMAP
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cross-view-linked-visualization-rendering

## Summary

Render a coordinated multi-panel dashboard in which the same retrieval state (reference image, composed query embedding, top-k results) is shown simultaneously as a 2D UMAP scatterplot, a ranked results gallery, and class-distribution views, with selection and highlighting synchronized across panels.

## When to use

Use when a visual analytics dashboard must present one shared retrieval state across multiple linked panels (2D embedding scatterplot, top-k image gallery, class histogram/wordcloud) so users can confirm via coordinated interaction where a composed query landed.

## When NOT to use

- You need metric/geodesic distances read off the 2D plot — InfoCIR's debiased UMAP layout is explicitly qualitative/topological, not metric
- You need to visualize a raw, unfiltered embedding space — standard UMAP on CLIP spaces clusters by artistic style rather than semantic categories, so apply the debiasing pipeline first
- The dataset exceeds the scatterplot limit (MAX_IMAGES_ON_SCATTERPLOT=100); use aggregated views (histogram/wordcloud) instead of densifying the scatterplot

## Inputs

- Class Projection Preservation-filtered ImageNet-R CLIP embeddings (after PCA-based Style Debiasing, Contrastive Debiasing, and ICA)
- UMAP 2D coordinates for the reference image, SEARLE composed query embedding, and top-k retrieved images
- Top-k ranked retrieval results (images + class labels) from cosine-similarity ranking
- Class labels from class_names.csv (dir_name,class_name)

## Outputs

- Interactive 2D Embedding View (Panel D) scatterplot in Plotly-Dash highlighting reference image, composed query, and top-k points
- Synchronized Query Results Panel gallery of top-k images with selectable ideal targets
- Linked class-frequency histogram (C1) and wordcloud (C2) panels that highlight classes on the scatterplot

## How to apply

First compute the shared 2D coordinates: apply UMAP's out-of-sample transform to the reference image embedding, the SEARLE composed query embedding, and the top-k retrieved image embeddings onto a fixed 2D map fitted on the Style-Debiasing/Contrastive-Debiasing/ICA-filtered ImageNet-R CLIP embeddings. Then position the points in the central Embedding View (Panel D) rendered with Plotly-Dash, using the resulting 2D coordinates, and keep the view synchronized with the Query Results Panel so that clicking a retrieved image marks it as an ideal target and highlights it in the scatterplot. Wire the Scatterplot Widget with zoom (+/- or area), pan (click-drag), and lasso/box multi-selection, and link the class-frequency histogram (C1) and wordcloud (C2) so that clicking a histogram bar highlights the corresponding class across views. Respect UI limits such as MAX_IMAGES_ON_SCATTERPLOT=100 and IMAGE_GALLERY_SIZE=24 from config.py. Treat the projected layout as qualitative/topological rather than metric: its purpose is rapid visual confirmation that the composed query landed within the intended semantic category, not precise distance reading.

## Related tools

- **UMAP** (Provides the debiased, out-of-sample 2D coordinates that position reference, query, and top-k points in the shared Embedding View) — https://github.com/giannhskp/InfoCIR
- **Plotly-Dash** (Modular dashboard framework rendering the Embedding View scatterplot, Query Results Panel, histogram, and wordcloud with cross-panel interaction callbacks) — https://github.com/giannhskp/InfoCIR
- **SEARLE** (Produces the composed query embedding that is projected into the shared 2D space alongside the reference and top-k images) — https://github.com/giannhskp/InfoCIR
- **CLIP** (Supplies the joint vision-language embeddings of the ImageNet-R dataset that are filtered and projected for the linked views) — https://github.com/giannhskp/InfoCIR

## Examples

```
cd cir_app; python run.py  # launch at http://localhost:8051; CLIP embeddings and UMAP projections are generated on first run
```

## Evaluation signals

- The Embedding View highlights exactly the reference image, the composed query point, and the k images returned by the Query Results Panel for the current search
- Clicking an image in the results gallery marks it as an ideal target and its scatterplot point is visually identified (and vice versa via lasso/box selection)
- Clicking a histogram bar (C1) highlights the corresponding class points on the scatterplot
- Scatterplot coordinates correspond to the UMAP out-of-sample transform of the filtered embeddings, and clusters are denser and well-separated so the composed query visibly lands in the intended semantic category (DG1)
- View sizes respect config.py limits: MAX_IMAGES_ON_SCATTERPLOT=100 points and IMAGE_GALLERY_SIZE=24 (4 per row) images in the gallery

## Limitations

- The 2D layout is qualitative/topological, not metric — distances in the Embedding View should not be interpreted as precise similarities
- Scalability of the linked interactive views is limited (MAX_IMAGES_ON_SCATTERPLOT=100) and acknowledged as future work
- Standard UMAP on CLIP spaces suffers from style bias, clustering images by artistic style; only the debiased pipeline yields semantically accurate neighborhoods

## Evidence
