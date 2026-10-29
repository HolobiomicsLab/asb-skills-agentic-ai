---
name: heatmap-rendering
description: Use when you have re-ranked retrieval results (top-k lists) for multiple
  prompt variants against a fixed reference ranking and need to verify, at a glance,
  whether user-selected ideal target images moved up or down in rank per variant.
when_to_use_negative:
- You need aggregate ranking quality metrics rather than per-target rank trajectories
  — use NDCG@K, mAP@K, MRR, Coverage, Mean Rank and Mean Similarity scores instead
- You are debugging model internals (which regions/words drive similarity) — use GradECLIP
  saliency maps or token-level attribution rather than the Rank-Δ heatmap
- You need spatial reasoning over the whole embedding space — use the debiased UMAP
  scatterplot view, not the rank heatmap
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Plotly-Dash
  role: Rendering the Rank-Δ heatmap inside the modular InfoCIR dashboard (Explanation
    Panel)
  repo: https://github.com/giannhskp/InfoCIR
- name: InfoCIR
  role: 'Host system: supplies the initial top-k reference ranking, prompt variants,
    ideal-image selection, and the panel where the heatmap is displayed'
  repo: https://github.com/giannhskp/InfoCIR
- name: SEARLE
  role: CIR back-end that textual-inverts the reference image into a pseudo-token
    so each prompt variant can be fused into a composed query vector for re-ranking
- name: CLIP
  role: Joint text/image embedding space in which cosine similarities are computed
    to derive the initial and per-variant rankings underlying the heatmap
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1109/pacificvis68791.2026.00041
    title: 'InfoCIR: Multimedia Analysis for Composed Image Retrieval'
  source_content_hash: sha256:d2cecc38ae53b292e9f4d72b9bb425679889215b4faaeae1a4deb73da707089b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giannhskp/InfoCIR
  tools:
  - CLIP
  - InfoCIR
  - Plotly-Dash
  - SEARLE
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# heatmap-rendering

## Summary

Render an Ideal-anchored Rank-Δ heatmap that visualizes how each LLM-generated prompt variant changes the rank position of user-selected ideal target images, using Plotly within the InfoCIR dashboard. This is the visual verification step of the diagnose-compare-enhance feedback loop for composed image retrieval.

## When to use

Use when you have re-ranked retrieval results (top-k lists) for multiple prompt variants against a fixed reference ranking and need to verify, at a glance, whether user-selected ideal target images moved up or down in rank per variant.

## When NOT to use

- You need aggregate ranking quality metrics rather than per-target rank trajectories — use NDCG@K, mAP@K, MRR, Coverage, Mean Rank and Mean Similarity scores instead
- You are debugging model internals (which regions/words drive similarity) — use GradECLIP saliency maps or token-level attribution rather than the Rank-Δ heatmap
- You need spatial reasoning over the whole embedding space — use the debiased UMAP scatterplot view, not the rank heatmap

## Inputs

- Initial top-k ranking of database images from the SEARLE/CLIP composed query (stable reference set)
- Per-variant re-ranked lists from re-evaluating each LLM-generated prompt variant against pre-computed database embeddings via cosine similarity
- User-selected ideal target images (I*) marked by clicking images in the Query Results Panel
- Rank-Δ values per (variant, ideal image) pair

## Outputs

- Ideal-anchored Rank-Δ heatmap figure with green cells for upward moves and red cells for drops (e.g., rank_delta_heatmap.png)
- Heatmap rendered in the InfoCIR Explanation Panel (panel F3)

## How to apply

Start from the initial top-k ranking produced by the CIR engine (SEARLE pseudo-token fused with the text prompt in CLIP's joint embedding space, ranked by cosine similarity against pre-computed database embeddings) — this serves as the stable reference set. For each LLM-generated prompt variant, recompose the query vector, re-rank all database images, and record the new rank position of each user-selected ideal image I* relative to the fixed initial top-k to obtain its Rank-Δ. Build a matrix with variants on one axis and ideal images on the other, encoding Rank-Δ values, and render it with Plotly using green cells for upward rank moves and red cells for drops. Save the figure (e.g., rank_delta_heatmap.png) so it can be surfaced in the Explanation Panel alongside the saliency map and token-attribution chart. The heatmap's purpose is verification of the prompt-enhancement loop: each variant's effect on the target's position (e.g., rank 12 to rank 2 in the paper's usage scenario) must be visible at a glance.

## Related tools

- **Plotly-Dash** (Rendering the Rank-Δ heatmap inside the modular InfoCIR dashboard (Explanation Panel)) — https://github.com/giannhskp/InfoCIR
- **InfoCIR** (Host system: supplies the initial top-k reference ranking, prompt variants, ideal-image selection, and the panel where the heatmap is displayed) — https://github.com/giannhskp/InfoCIR
- **SEARLE** (CIR back-end that textual-inverts the reference image into a pseudo-token so each prompt variant can be fused into a composed query vector for re-ranking)
- **CLIP** (Joint text/image embedding space in which cosine similarities are computed to derive the initial and per-variant rankings underlying the heatmap)

## Examples

```
import plotly.express as px; fig = px.imshow(rank_delta_matrix, color_continuous_scale=['red','green']); fig.write_image('rank_delta_heatmap.png')
```

## Evaluation signals

- The heatmap figure file (e.g., rank_delta_heatmap.png) exists and is non-empty
- Every prompt variant has a cell for each user-selected ideal image, encoding its re-computed Rank-Δ relative to the fixed initial top-k
- Cell colors correctly encode direction: green for upward rank moves, red for rank drops, consistent with the recorded per-variant re-rankings
- Reported rank transitions (e.g., rank 12 to rank 2 in the paper's usage scenario) are visible at a glance in the rendered heatmap
- The heatmap appears in the Explanation Panel (F3) alongside the saliency map (F1) and token attribution bar chart (F2)

## Limitations

- The heatmap anchors on user-selected ideal images, so it is only informative after the user has marked ideal targets in the Query Results Panel; it says nothing about global retrieval quality
- It depends on the initial top-k as a stable reference set; rank deltas are relative to that fixed reference, not an absolute quality measure
- InfoCIR's CIR model support is limited (SEARLE, SEARLE-XL, Phi, OTI on ImageNet-R with CLIP backbones), and scalability and personalization remain future work

## Evidence

- [intro] couples LLM-generated prompt variants with visual verification via saliency maps, token-level attribution, and a novel Ideal-anchored Rank-∆ heatmap to verify prompt impact: [evidence span withheld — anchor status: not_found]
