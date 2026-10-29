---
name: ideal-image-anchor-selection
description: Use when a composed image + text retrieval run (SEARLE/CLIP) has produced
  a top-k result list and the user wants to designate specific result images as targets
  whose ranking changes under enhanced prompts must be verified.
when_to_use_negative:
- No top-k retrieval results exist yet, or the user has not inspected the Query Results
  Panel — anchors must be chosen from actually retrieved candidates
- The goal is a single-prompt retrieval without iterative refinement — anchor selection
  and Rank-Δ verification only pay off within the diagnose-compare-enhance loop
- The CIR back-end is not one of the supported models (SEARLE, SEARLE-XL, Phi, OTI);
  the paper acknowledges limited model support as a current limitation
edam_operation: http://edamontology.org/operation_3802
tools:
- name: InfoCIR
  role: Dashboard whose Query Results Panel hosts the click-to-select interaction
    for marking ideal target images and whose Explanation Panel renders the Rank-Δ
    heatmap
  repo: https://github.com/giannhskp/InfoCIR
- name: SEARLE
  role: CIR back-end whose pseudo-token fusion produces the composed query embeddings
    used to re-rank the database and compute each anchor's Rank-Δ
  repo: https://github.com/giannhskp/InfoCIR
- name: CLIP
  role: Joint text/image embedding space in which cosine similarities between composed
    queries and database embeddings are computed for ranking
  repo: https://github.com/giannhskp/InfoCIR
- name: Plotly-Dash
  role: Modular UI framework implementing the gallery selection widget and the Ideal-anchored
    Rank-Δ heatmap visualization
  repo: https://github.com/giannhskp/InfoCIR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1109/pacificvis68791.2026.00041
    title: 'InfoCIR: Multimedia Analysis for Composed Image Retrieval'
  source_content_hash: sha256:885d7dffd8043b9451458dd60c29ca49634c62372ef9b52e5fa5684b5b478c92
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giannhskp/InfoCIR
  tools:
  - CLIP
  - InfoCIR
  - Plotly-Dash
  - SEARLE
  edam_operation: http://edamontology.org/operation_3802
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Ideal Image Anchor Selection

## Summary

Enables a user to mark one or more top-k retrieved images as ideal targets (I*) inside the InfoCIR dashboard, anchoring the subsequent Rank-Δ verification loop that shows how each LLM-generated prompt variant moves I*'s ranking.

## When to use

Use when a composed image + text retrieval run (SEARLE/CLIP) has produced a top-k result list and the user wants to designate specific result images as targets whose ranking changes under enhanced prompts must be verified.

## When NOT to use

- No top-k retrieval results exist yet, or the user has not inspected the Query Results Panel — anchors must be chosen from actually retrieved candidates
- The goal is a single-prompt retrieval without iterative refinement — anchor selection and Rank-Δ verification only pay off within the diagnose-compare-enhance loop
- The CIR back-end is not one of the supported models (SEARLE, SEARLE-XL, Phi, OTI); the paper acknowledges limited model support as a current limitation

## Inputs

- Top-k ranked result images from the Composed Image Retrieval Panel (ranked by cosine similarity of the composed SEARLE+CLIP query vector against pre-computed database embeddings)
- User click selections on gallery images marking them as ideal targets (I*)
- Reference image and text prompt used for the initial composed query
- LLM-generated prompt variants from the Prompt Enhancement Panel (Mistral-7B-Instruct-v0.2, ENHANCEMENT_CANDIDATE_PROMPTS = 10 by default)

## Outputs

- Set of user-selected ideal anchor images I* passed to the prompt enhancer
- Rank-Δ value per anchor image for each candidate prompt variant, relative to the fixed initial top-k
- Ideal-anchored Rank-Δ heatmap figure saved as rank_delta_heatmap.png (green cells for upward rank moves, red cells for drops), shown in Explanation Panel F3

## How to apply

After the composed query (reference image + natural language prompt fused via SEARLE's pseudo-token in CLIP's text embedding space) yields a cosine-similarity-ranked top-k list in the Query Results Panel, let the user click one or more gallery images to mark them as ideal targets for prompt refinement. These anchors are passed to the Prompt Enhancement Panel, where Mistral-7B-Instruct-v0.2 proposes counterfactual prompt variants conditioned on the selected ideals. Each variant is re-evaluated by the retrieval engine: its text is fused with the SEARLE pseudo-token, cosine similarities against pre-computed database embeddings are recomputed, and the new rank of each anchor I* is recorded as a Rank-Δ relative to the fixed initial top-k. Anchors and Rank-Δ values are rendered in the Explanation Panel (F3) as an Ideal-anchored Rank-Δ heatmap (green for upward moves, red for drops) via Plotly-Dash. Rationale: this closes the diagnose-compare-enhance feedback loop by making each variant's effect on the target's position (e.g., rank 12 to rank 2) visible at a glance.

## Related tools

- **InfoCIR** (Dashboard whose Query Results Panel hosts the click-to-select interaction for marking ideal target images and whose Explanation Panel renders the Rank-Δ heatmap) — https://github.com/giannhskp/InfoCIR
- **SEARLE** (CIR back-end whose pseudo-token fusion produces the composed query embeddings used to re-rank the database and compute each anchor's Rank-Δ) — https://github.com/giannhskp/InfoCIR
- **CLIP** (Joint text/image embedding space in which cosine similarities between composed queries and database embeddings are computed for ranking) — https://github.com/giannhskp/InfoCIR
- **Plotly-Dash** (Modular UI framework implementing the gallery selection widget and the Ideal-anchored Rank-Δ heatmap visualization) — https://github.com/giannhskp/InfoCIR

## Examples

```
cd cir_app && python run.py
```

## Evaluation signals

- The rank_delta_heatmap.png file exists after the loop completes and its cells encode the re-computed rank changes of I* for every prompt variant
- Heatmap color semantics are correct: green cells for upward rank moves and red cells for drops, relative to the fixed initial top-k reference ranking
- A known usage scenario is visible at a glance, e.g. an anchor moving from rank 12 to rank 2 under an enhanced prompt
- Selection is multi-capable: multiple images can be marked as ideals simultaneously, consistent with the README's multi-selection gallery behavior
- Re-running the same anchors and prompt variants reproduces the same Rank-Δ ordering, consistent with deterministic cosine-similarity ranking against the same pre-computed embeddings

## Limitations

- Anchors are only meaningful relative to the initial top-k; the Rank-Δ is computed against this fixed reference set, so changing the initial query invalidates prior anchors' deltas
- Rank deltas reflect the CLIP/SEARLE embedding space, which can suffer from style bias where images cluster by artistic style rather than semantic categories
- Limited CIR model support, scalability, and personalization are acknowledged as future work in the paper's discussion

## Evidence

- [intro] couples LLM-generated prompt variants with visual verification via saliency maps, token-level attribution, and a novel Ideal-anchored Rank-∆ heatmap: [evidence span withheld — anchor status: not_found]
