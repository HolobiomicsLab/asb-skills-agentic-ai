---
name: rank-delta-computation
description: 'Use when LLM-generated prompt variants must be verified against user-selected
  ideal images: you have an initial top-k ranking, SEARLE pseudo-token composed queries
  per variant, and need per-variant rank changes of the ideal image I*.'
when_to_use_negative:
- The user has not marked any ideal target images I*, since the heatmap anchors rank
  changes specifically to user-selected ideal targets
- The goal is global retrieval quality (Recall@K, NDCG@K, mAP, MRR) rather than per-variant
  rank changes of a specific ideal image; use the enhancement metrics panel instead
- The retrieval engine or CLIP backbone is not one of the supported composed-image-retrieval
  setups (SEARLE / SEARLE-XL / Phi / OTI), since composed query vectors cannot be
  formed
edam_operation: http://edamontology.org/operation_0564
tools:
- name: SEARLE
  role: Performs textual inversion of the reference image into a pseudo-token that
    is fused with each prompt variant to form the composed query vector
  repo: https://github.com/giannhskp/InfoCIR
- name: CLIP
  role: Provides the joint text/image embedding space in which cosine similarities
    against pre-computed database embeddings produce the re-ranking used for Rank-Δ
  repo: https://github.com/giannhskp/InfoCIR
- name: Plotly-Dash
  role: Renders the Ideal-anchored Rank-Δ heatmap (green/red cells) as panel F3 of
    the dashboard and exports it as rank_delta_heatmap.png
  repo: https://github.com/giannhskp/InfoCIR
- name: InfoCIR
  role: 'Hosts the whole verification loop: Prompt Enhancement Panel (E) proposes
    variants conditioned on selected ideals; Explanation Panel F3 shows the Rank-Δ
    heatmap'
  repo: https://github.com/giannhskp/InfoCIR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1109/pacificvis68791.2026.00041
    title: 'InfoCIR: Multimedia Analysis for Composed Image Retrieval'
  source_content_hash: sha256:1dd94a3ea551c1cac77d80443d372edf7435befb272989022780d051c5099cbe
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# rank-delta-computation

## Summary

Computes the Rank-Δ of user-selected ideal target images under LLM-generated prompt variants in InfoCIR, and renders the Ideal-anchored Rank-Δ heatmap (green for upward moves, red for drops) to verify at a glance how each prompt variant shifts the target's rank relative to the fixed initial top-k.

## When to use

Use when LLM-generated prompt variants must be verified against user-selected ideal images: you have an initial top-k ranking, SEARLE pseudo-token composed queries per variant, and need per-variant rank changes of the ideal image I*.

## When NOT to use

- The user has not marked any ideal target images I*, since the heatmap anchors rank changes specifically to user-selected ideal targets
- The goal is global retrieval quality (Recall@K, NDCG@K, mAP, MRR) rather than per-variant rank changes of a specific ideal image; use the enhancement metrics panel instead
- The retrieval engine or CLIP backbone is not one of the supported composed-image-retrieval setups (SEARLE / SEARLE-XL / Phi / OTI), since composed query vectors cannot be formed

## Inputs

- LLM-generated prompt variants from the Prompt Enhancement stage (e.g., Mistral-7B-Instruct-v0.2 candidates)
- Initial top-k ranking used as the stable reference set
- SEARLE pseudo-token of the user's reference image in CLIP's text embedding space
- Pre-computed CLIP database image embeddings
- User-selected ideal target image(s) I* marked in the Query Results Panel

## Outputs

- Per-variant Rank-Δ of each ideal image I* relative to the fixed initial top-k
- Ideal-anchored Rank-Δ heatmap figure saved as rank_delta_heatmap.png (green cells for upward rank moves, red cells for drops)

## How to apply

Load the LLM-generated prompt variants from the Prompt Enhancement stage together with the initial top-k ranking, which serves as the stable reference set. For each prompt variant, fuse it with the SEARLE pseudo-token of the reference image into a joint query vector in CLIP's text embedding space. Compute cosine similarities between each composed query vector and the pre-computed database embeddings to re-rank all database images, then record the new rank position of each user-selected ideal image I* and compute its Rank-Δ relative to the fixed initial top-k. Render the Ideal-anchored Rank-Δ heatmap with Plotly, encoding upward rank moves in green cells and drops in red, and save the figure as rank_delta_heatmap.png. The rationale is InfoCIR's 'diagnose-compare-enhance' feedback loop: this visual verification shows each variant's effect on the target's position (e.g., moving from rank 12 to rank 2 in the usage scenario) so users can iteratively refine the composed prompt.

## Related tools

- **SEARLE** (Performs textual inversion of the reference image into a pseudo-token that is fused with each prompt variant to form the composed query vector) — https://github.com/giannhskp/InfoCIR
- **CLIP** (Provides the joint text/image embedding space in which cosine similarities against pre-computed database embeddings produce the re-ranking used for Rank-Δ) — https://github.com/giannhskp/InfoCIR
- **Plotly-Dash** (Renders the Ideal-anchored Rank-Δ heatmap (green/red cells) as panel F3 of the dashboard and exports it as rank_delta_heatmap.png) — https://github.com/giannhskp/InfoCIR
- **InfoCIR** (Hosts the whole verification loop: Prompt Enhancement Panel (E) proposes variants conditioned on selected ideals; Explanation Panel F3 shows the Rank-Δ heatmap) — https://github.com/giannhskp/InfoCIR

## Examples

```
import plotly.express as px; fig = px.imshow(rank_delta_matrix, color_continuous_scale=['red','green']); fig.write_image('rank_delta_heatmap.png')
```

## Evaluation signals

- The output file rank_delta_heatmap.png exists and its cells encode the re-computed rank changes of I* for every prompt variant
- Cell values are Rank-Δ relative to the fixed initial top-k reference set (new rank minus original rank), not raw similarity scores
- Upward rank moves appear as green cells and rank drops as red cells, matching the paper's Ideal-anchored heatmap convention
- A manually verifiable variant shows the expected position change (e.g., ideal image moving from rank 12 to rank 2 in the paper's usage scenario)
- Re-computed ranks are consistent with cosine similarities between each composed SEARLE+variant query vector and the pre-computed database embeddings

## Limitations

- Limited CIR model support is acknowledged as a limitation; extending model support is deferred to future work
- Scalability and personalization are not yet addressed and are deferred to future work
- Rank changes are anchored only to user-selected ideal images; no retrieval-specific signal for prompt evolution beyond the ideal anchors is provided by generic tools

## Evidence

- [intro] couples LLM-generated prompt variants with visual verification via saliency maps, token-level attribution, and a novel Ideal-anchored Rank-∆ heatmap to verify prompt impact: [evidence span withheld — anchor status: not_found]
- [intro] They lack retrieval-specific signals such as rank position, prompt evolution, and ideal-image anchors, all of which are integrated into our system: [evidence span withheld — anchor status: not_found]
