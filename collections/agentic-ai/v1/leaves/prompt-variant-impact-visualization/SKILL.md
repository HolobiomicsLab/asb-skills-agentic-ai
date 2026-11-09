---
name: prompt-variant-impact-visualization
description: Use when a composed image retrieval workflow has produced multiple LLM-enhanced
  prompt variants plus a stable initial top-k ranking, and you must verify which variant
  actually moves user-selected ideal images to better rank positions.
when_to_use_negative:
- You only need global retrieval quality of one fixed query — use Recall@K / NDCG
  evaluation instead of a per-ideal Rank-Δ heatmap
- The user has not marked any ideal target images, so there is no anchor I* whose
  rank change can be tracked
- The embedding space itself is under investigation (style bias, cluster structure)
  — the debiased UMAP projection workflow is the right tool there, not rank-delta
  analysis
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: SEARLE
  role: CIR back-end that textual-inverts the reference image into a pseudo-token
    and composes it with each prompt variant into a joint query vector
  repo: https://github.com/giannhskp/InfoCIR
- name: CLIP
  role: Provides the joint text-image embedding space in which cosine similarities
    against pre-computed database embeddings are computed for re-ranking
  repo: https://github.com/giannhskp/InfoCIR
- name: Plotly-Dash
  role: Renders the Ideal-anchored Rank-Δ heatmap (green/red cells) inside the Explanation
    Panel (F3) and exports it as rank_delta_heatmap.png
  repo: https://github.com/giannhskp/InfoCIR
- name: InfoCIR
  role: Visual analytics system whose Prompt Enhancement and Explanation panels host
    the variant re-evaluation and heatmap verification loop
  repo: https://github.com/giannhskp/InfoCIR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1109/pacificvis68791.2026.00041
    title: 'InfoCIR: Multimedia Analysis for Composed Image Retrieval'
  source_content_hash: sha256:b91cc2f612d08d14efa999e7dd7c7470253e4b63cb60d2d4963d7af0449fa2f0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giannhskp/InfoCIR
  tools:
  - CLIP
  - InfoCIR
  - Plotly-Dash
  - SEARLE
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# prompt-variant-impact-visualization

## Summary

Re-evaluates each LLM-generated prompt variant through the InfoCIR/SEARLE retrieval engine and renders an Ideal-anchored Rank-Δ heatmap (green = upward rank moves, red = drops) showing how each variant changes the rank position of user-selected ideal target images relative to the fixed initial top-k.

## When to use

Use when a composed image retrieval workflow has produced multiple LLM-enhanced prompt variants plus a stable initial top-k ranking, and you must verify which variant actually moves user-selected ideal images to better rank positions.

## When NOT to use

- You only need global retrieval quality of one fixed query — use Recall@K / NDCG evaluation instead of a per-ideal Rank-Δ heatmap
- The user has not marked any ideal target images, so there is no anchor I* whose rank change can be tracked
- The embedding space itself is under investigation (style bias, cluster structure) — the debiased UMAP projection workflow is the right tool there, not rank-delta analysis

## Inputs

- LLM-generated candidate prompt variants from the Prompt Enhancement stage (e.g. 10 candidates per ENHANCEMENT_CANDIDATE_PROMPTS)
- Initial top-k ranking from the SEARLE-based retrieval engine (stable reference set)
- SEARLE pseudo-token embedding of the user's reference image
- Pre-computed CLIP database embeddings (e.g. ImageNet-R, up to 30000 sampled images)

## Outputs

- Per-variant rank positions of each user-selected ideal image I* and their Rank-Δ values
- Ideal-anchored Rank-Δ heatmap figure (rank_delta_heatmap.png) with green/red encoded cells
- Supporting retrieval metrics per candidate (NDCG@K, mAP@K, MRR, Coverage, Mean Rank, Mean Similarity) shown in the Explanation Panel (F3)

## How to apply

Load the prompt variants from the Prompt Enhancement stage together with the initial top-k ranking that serves as the stable reference set. For each variant, fuse it with the SEARLE pseudo-token (textual inversion of the reference image) into a joint query vector in CLIP's text embedding space (ViT-B/32 backbone by default in InfoCIR). Compute cosine similarities between each composed query vector and the pre-computed database embeddings to re-rank all images. Record the new rank of each user-selected ideal image I* and compute its Rank-Δ versus the fixed initial top-k (e.g. rank 12 → rank 2 in the paper's usage scenario). Render the Rank-Δ heatmap with Plotly, using green cells for upward moves and red cells for drops, and save it as rank_delta_heatmap.png. Judge correctness by checking that the heatmap file exists and every cell encodes the recomputed rank change of I* for exactly one variant.

## Related tools

- **SEARLE** (CIR back-end that textual-inverts the reference image into a pseudo-token and composes it with each prompt variant into a joint query vector) — https://github.com/giannhskp/InfoCIR
- **CLIP** (Provides the joint text-image embedding space in which cosine similarities against pre-computed database embeddings are computed for re-ranking) — https://github.com/giannhskp/InfoCIR
- **Plotly-Dash** (Renders the Ideal-anchored Rank-Δ heatmap (green/red cells) inside the Explanation Panel (F3) and exports it as rank_delta_heatmap.png) — https://github.com/giannhskp/InfoCIR
- **InfoCIR** (Visual analytics system whose Prompt Enhancement and Explanation panels host the variant re-evaluation and heatmap verification loop) — https://github.com/giannhskp/InfoCIR

## Examples

```
import plotly.express as px; fig = px.imshow(rank_delta_matrix, color_continuous_scale=['red','green']); fig.write_image('rank_delta_heatmap.png')
```

## Evaluation signals

- rank_delta_heatmap.png exists and its cells encode the re-computed rank change of I* for every candidate variant
- Each cell's Rank-Δ equals (new rank of I* under the variant) minus (initial rank in the fixed top-k reference set)
- Rank changes match the paper's verification loop scenario, e.g. a variant moving I* from rank 12 to rank 2 is visible at a glance as a green cell
- Per-variant NDCG@K, mAP@K, and MRR metrics reported alongside the heatmap are consistent with the direction of the Rank-Δ cells
- Re-ranking is reproduced by cosine similarity against the same pre-computed database embeddings used for the initial top-k, ensuring the only difference is the prompt variant

## Limitations

- Verification is anchored to user-selected ideal images; without marked ideals the heatmap cannot be constructed
- Rank-Δ reflects the chosen CLIP/SEARLE model configuration (e.g. ViT-B/32 vs ViT-L/14) and may shift when switching CIR back-ends such as SEARLE-XL, Phi, or OTI
- Re-ranking is limited to the pre-computed database sample (DATASET_SAMPLE_SIZE, e.g. 30000 ImageNet-R images), so absolute ranks depend on the sampled database

## Evidence

- [intro] Ideal-anchored Rank-Delta heatmap workflow: [evidence span withheld — anchor status: not_found]
