---
name: cosine-similarity-ranking
description: Use when a composed query embedding (reference image pseudo-token fused
  with a text prompt in CLIP's text embedding space) must be matched against a fixed
  image-embedding database to produce a ranked top-k list or to re-rank after prompt
  changes.
when_to_use_negative:
- You need a fine-grained similarity beyond what CLIP's joint embedding space captures,
  or a task where modality-specific distances (not a shared text/image space) are
  required — InfoCIR itself is scoped to CIR models that embed both modalities jointly
- You only need a low-dimensional visualization of the data without any ranking —
  the debiased UMAP projection pipeline (Style Debiasing, Contrastive Debiasing, ICA)
  is the appropriate skill instead
tools:
- name: SEARLE
  role: Zero-shot textual-inversion model that transforms the reference image into
    a pseudo-token fused with the text prompt to form the composed query embedding
    before cosine-similarity ranking
- name: CLIP
  role: Vision-language model embedding images and text into the joint feature space
    whose embeddings are compared via cosine similarity
- name: InfoCIR
  role: Dashboard implementing the cosine-similarity retrieval back-end over pre-computed
    ImageNet-R embeddings with adjustable top-k
  repo: https://github.com/giannhskp/InfoCIR
provenance:
  source_task_ids:
  - task_008
  - task_001
  source_papers:
  - doi: 10.1109/pacificvis68791.2026.00041
    title: 'InfoCIR: Multimedia Analysis for Composed Image Retrieval'
  source_content_hash: sha256:b456f8a50dea74fbfa76056306914d9ab1e29c2e7f2cf1d9e573f7d7b637bfeb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giannhskp/InfoCIR
  tools:
  - CLIP
  - InfoCIR
  - Plotly-Dash
  - SEARLE
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# cosine-similarity-ranking

## Summary

Ranks a database of pre-computed CLIP image embeddings against a composed image+text query vector using cosine similarity, producing the top-k ranked result list that InfoCIR's SEARLE-based CIR back-end is built on.

## When to use

Use when a composed query embedding (reference image pseudo-token fused with a text prompt in CLIP's text embedding space) must be matched against a fixed image-embedding database to produce a ranked top-k list or to re-rank after prompt changes.

## When NOT to use

- You need a fine-grained similarity beyond what CLIP's joint embedding space captures, or a task where modality-specific distances (not a shared text/image space) are required — InfoCIR itself is scoped to CIR models that embed both modalities jointly
- You only need a low-dimensional visualization of the data without any ranking — the debiased UMAP projection pipeline (Style Debiasing, Contrastive Debiasing, ICA) is the appropriate skill instead

## Inputs

- Pre-computed CLIP image embeddings for the ImageNet-R database (ViT-B/32 backbone)
- Composed query embedding: SEARLE pseudo-token fused with the natural-language text prompt in CLIP's text embedding space
- k value for top-k selection (k=10 in the paper's evaluation protocol)

## Outputs

- Ranked result set with image identifiers, cosine similarity scores in [-1, 1], and associated class labels/tags (e.g., retrieval_results.csv with exactly k=10 ranked rows)
- Per-prompt-variant rank positions of user-selected ideal images I* for Rank-Δ computation (feeding the Ideal-anchored Rank-Δ heatmap)

## How to apply

First encode the reference image into a SEARLE pseudo-token within CLIP's text embedding space and fuse it with the natural-language prompt to form the composed query embedding. Compute cosine similarities between this vector and every pre-computed database embedding (for ImageNet-R, embeddings are pre-computed with the ViT-B/32 CLIP backbone per the InfoCIR config), then sort all database images by similarity in descending order. Select the top-k images following the paper's evaluation protocol (k=10, configurable via the app's top-k selector) and record each result's rank index, cosine similarity score, image identifier, and class label. When a prompt variant is re-evaluated (Prompt Enhancement stage), recompute the same cosine-similarity ranking against the identical database embeddings and record the new rank of user-selected ideal images so Rank-Δ relative to the fixed initial top-k can be computed. The rationale for using cosine similarity is that CLIP embeds multiple modalities into a joint space where semantic closeness is angle-based rather than Euclidean, making nearest-neighbor retrieval a cosine-similarity nearest-neighbor search.

## Related tools

- **SEARLE** (Zero-shot textual-inversion model that transforms the reference image into a pseudo-token fused with the text prompt to form the composed query embedding before cosine-similarity ranking)
- **CLIP** (Vision-language model embedding images and text into the joint feature space whose embeddings are compared via cosine similarity)
- **InfoCIR** (Dashboard implementing the cosine-similarity retrieval back-end over pre-computed ImageNet-R embeddings with adjustable top-k) — https://github.com/giannhskp/InfoCIR

## Examples

```
python -c "import torch; torch.hub.load('miccunifi/SEARLE', 'searle', backbone='ViT-B/32')"
```

## Evaluation signals

- retrieval_results.csv exists and contains exactly k=10 ranked rows, each with a rank index, image identifier, a cosine similarity score in [-1, 1], and a class label field
- Every recorded similarity value lies within the mathematically valid cosine range [-1, 1]
- Re-ranking under a modified prompt preserves the database embedding set; only the query vector changes, so Rank-Δ values must be reproducible given the same query and database
- In the paper's verification loop, a known improvement (e.g., ideal image moving from rank 12 to rank 2 under an enhanced prompt variant) must be visible as a positive Rank-Δ in the heatmap
- Top-k results are the k database entries with strictly highest cosine similarity under descending sort (no ties silently dropped without a documented tie-break)

## Limitations

- Ranking quality depends on CLIP's joint space, which suffers style bias in CLIP-based embedding spaces (images may cluster by artistic style rather than semantic category) — mitigated downstream by debiased UMAP, not in the ranking itself
- Limited CIR model support is acknowledged as a limitation; model support, scalability, and personalization are deferred to future work
- The retrieval step is a nearest-neighbor search in a joint embedding space and does not by itself explain why an image ranks high — saliency maps and token attribution are needed for explanation

## Evidence
