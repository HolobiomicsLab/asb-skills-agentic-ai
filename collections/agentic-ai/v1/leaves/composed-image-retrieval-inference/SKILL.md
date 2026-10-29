---
name: composed-image-retrieval-inference
description: Use when a task requires ranking a database of images against a query
  defined by a reference image plus a text modifier (e.g., 'make it red'), producing
  a top-k result list in a joint CLIP embedding space.
when_to_use_negative:
- The task needs explanation or diagnosis of why results ranked as they did (use saliency/token
  attribution instead — this skill only produces rankings)
- The query is text-only or image-only, not a composition of a reference image with
  a text modifier (plain CLIP zero-shot classification or image-image similarity applies
  instead)
- The database embeddings have not been pre-computed and cannot fit in memory for
  exhaustive cosine-similarity ranking
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: SEARLE
  role: State-of-the-art CIR back-end that performs zero-shot textual inversion of
    the reference image into a pseudo-token within CLIP's text embedding space
  repo: https://github.com/giannhskp/InfoCIR
- name: CLIP
  role: Vision-language model providing the joint image-text feature space and the
    image embeddings for the database against which cosine similarities are computed
- name: InfoCIR
  role: Plotly-Dash dashboard that wraps the retrieval back-end, exposes adjustable
    top-k search, and feeds results to UMAP projection, saliency explanation, and
    prompt enhancement
  repo: https://github.com/giannhskp/InfoCIR
provenance:
  source_task_ids:
  - task_001
  - task_008
  source_papers:
  - doi: 10.1109/pacificvis68791.2026.00041
    title: 'InfoCIR: Multimedia Analysis for Composed Image Retrieval'
  source_content_hash: sha256:d4fdda6bde9afdb2853b0d1bce245a9fef37791eab628eef53b31a3bcc45b778
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giannhskp/InfoCIR
  tools:
  - CLIP
  - InfoCIR
  - Plotly-Dash
  - SEARLE
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# composed-image-retrieval-inference

## Summary

Runs zero-shot composed image retrieval (CIR) by fusing a SEARLE-derived pseudo-token from a reference image with a natural-language prompt in CLIP's text embedding space, then ranking a pre-computed image database by cosine similarity. This is the core retrieval operation that InfoCIR's dashboard builds its diagnose-compare-enhance loop upon.

## When to use

Use when a task requires ranking a database of images against a query defined by a reference image plus a text modifier (e.g., 'make it red'), producing a top-k result list in a joint CLIP embedding space.

## When NOT to use

- The task needs explanation or diagnosis of why results ranked as they did (use saliency/token attribution instead — this skill only produces rankings)
- The query is text-only or image-only, not a composition of a reference image with a text modifier (plain CLIP zero-shot classification or image-image similarity applies instead)
- The database embeddings have not been pre-computed and cannot fit in memory for exhaustive cosine-similarity ranking

## Inputs

- Reference image (query image file)
- Natural-language text prompt describing the desired modification (e.g., 'wearing sunglasses')
- Pre-computed CLIP embeddings for the database (e.g., ImageNet-R, up to 30000 sampled images)
- class_names.csv mapping dir_name to class_name for labeling results
- Pre-trained SEARLE model weights (ViT-B/32 or ViT-L/14 backbone)

## Outputs

- Ranked top-k result list (rank index, image identifier, cosine similarity score in [-1, 1], class label)
- Composed query embedding vector in CLIP's joint space
- retrieval_results.csv with exactly k ranked rows

## How to apply

Load pre-computed CLIP image embeddings for the database (e.g., ImageNet-R, sampled up to DATASET_SAMPLE_SIZE=30000 in the InfoCIR app) and load a pre-trained SEARLE model via PyTorch Hub (backbones ViT-B/32 or ViT-L/14; CIR_EVAL_TYPE='searle', CIR_PREPROCESS_TYPE='targetpad' in InfoCIR's config). SEARLE performs zero-shot textual inversion, transforming the reference image into a pseudo-token within CLIP's text embedding space; fuse this pseudo-token with the natural-language prompt to form the composed query vector. Compute cosine similarities between the composed query vector and all database embeddings and rank every database image; select the top-k results (the paper's evaluation protocol uses k=10, while the dashboard exposes adjustable top-K, defaulting to a 24-image gallery). Write the ranked list with rank index, image identifier, cosine similarity score in [-1, 1], and class label (from class_names.csv mapping dir_name to class_name) to an output such as retrieval_results.csv. The rationale, per the article, is that CLIP embeds both modalities into a joint feature space, so a single query vector supports nearest-neighbor retrieval and feeds downstream UMAP projection, saliency explanation, and LLM prompt-enhancement stages.

## Related tools

- **SEARLE** (State-of-the-art CIR back-end that performs zero-shot textual inversion of the reference image into a pseudo-token within CLIP's text embedding space) — https://github.com/giannhskp/InfoCIR
- **CLIP** (Vision-language model providing the joint image-text feature space and the image embeddings for the database against which cosine similarities are computed)
- **InfoCIR** (Plotly-Dash dashboard that wraps the retrieval back-end, exposes adjustable top-k search, and feeds results to UMAP projection, saliency explanation, and prompt enhancement) — https://github.com/giannhskp/InfoCIR

## Examples

```
python -c "import torch; torch.hub.load('miccunifi/SEARLE', 'searle', backbone='ViT-B/32')"
```

## Evaluation signals

- retrieval_results.csv exists and contains exactly k ranked rows (k=10 in the paper's protocol), each with a rank index, image identifier, and cosine similarity score in [-1, 1]
- Every result row carries a class label resolvable through class_names.csv (dir_name -> class_name)
- The composed query is produced by a SEARLE pseudo-token fused with the text prompt, not by a raw CLIP text encoding of the prompt alone
- Top-k results are monotonically ordered by descending cosine similarity to the composed query vector
- In the dashboard, changing the text prompt (e.g., to 'wearing sunglasses') visibly changes the top-k ranking, confirming the composition is prompt-sensitive

## Limitations

- The reference implementation is validated on ImageNet-R with CLIP ViT-B/32; other datasets require restructuring the dataset folder with class_names.csv and updating DATASET_ROOT_PATH
- Scalability is limited: exhaustive cosine ranking over up to 30000 pre-computed embeddings, and the scatterplot caps display at 100 images
- Limited CIR model support is acknowledged as a limitation; extending model support, scalability, and personalization is deferred to future work
- Dense saliency analysis is capped (SALIENCY_MAX_CANDIDATES, default top-5) because attribution over all retrieved images is costly

## Evidence

- [methods] SEARLE textual inversion: [evidence span withheld — anchor status: not_found]
