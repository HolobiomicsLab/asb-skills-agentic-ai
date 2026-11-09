---
name: clip-joint-embedding-composition
description: Use when a query must combine a reference image plus a text modification
  (e.g., 'wearing a red collar') into one searchable vector, and pre-computed CLIP
  image embeddings of the target database are available for nearest-neighbor retrieval.
when_to_use_negative:
- When the task is standard unimodal text-to-image or image-to-image retrieval with
  no reference image + text modifier composition required
- When the database has no pre-computed CLIP-compatible embeddings or the images fall
  outside a CLIP-segmentable label space (the pipeline requires class_names.csv and
  class-labeled directories)
- When the goal is embedding visualization or debiasing (style debiasing, contrastive
  debiasing, ICA projection) rather than producing a ranked retrieval list
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: SEARLE
  role: 'Zero-shot textual inversion: transforms the reference image into a pseudo-token
    within CLIP''s text embedding space to form the composed query'
  repo: https://github.com/giannhskp/InfoCIR
- name: CLIP
  role: Vision-language backbone (ViT-B/32 or ViT-L/14) providing the joint image/text
    embedding space in which the pseudo-token and database embeddings are compared
  repo: https://github.com/giannhskp/InfoCIR
- name: InfoCIR
  role: Plotly-Dash dashboard that wraps the SEARLE-based CIR back-end, exposing query
    composition, adjustable top-k, and result exploration
  repo: https://github.com/giannhskp/InfoCIR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1109/pacificvis68791.2026.00041
    title: 'InfoCIR: Multimedia Analysis for Composed Image Retrieval'
  source_content_hash: sha256:c44e4953ae57c3e54237f561e93bb6c877e7d8ab2e733fe6285a0d763786d443
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giannhskp/InfoCIR
  tools:
  - CLIP
  - InfoCIR
  - SEARLE
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# clip-joint-embedding-composition

## Summary

Composes a reference image with a natural-language text modifier into a single query embedding by using SEARLE's zero-shot textual inversion to map the image into CLIP's text embedding space, then ranks the ImageNet-R database by cosine similarity. This is the retrieval operation the InfoCIR dashboard is built on.

## When to use

Use when a query must combine a reference image plus a text modification (e.g., 'wearing a red collar') into one searchable vector, and pre-computed CLIP image embeddings of the target database are available for nearest-neighbor retrieval.

## When NOT to use

- When the task is standard unimodal text-to-image or image-to-image retrieval with no reference image + text modifier composition required
- When the database has no pre-computed CLIP-compatible embeddings or the images fall outside a CLIP-segmentable label space (the pipeline requires class_names.csv and class-labeled directories)
- When the goal is embedding visualization or debiasing (style debiasing, contrastive debiasing, ICA projection) rather than producing a ranked retrieval list

## Inputs

- Reference image (user-provided, e.g. from an ImageNet-R class directory)
- Natural-language text modifier/prompt (e.g. 'wearing a red collar', 'make it red')
- Pre-computed CLIP image embeddings for the ImageNet-R database (ViT-B/32, up to 30000 images)
- class_names.csv mapping dir_name to class_name

## Outputs

- Composed query embedding in CLIP's joint text/image embedding space
- Ranked top-k result list (k=10 per evaluation protocol) with image identifiers, cosine similarity scores in [-1, 1], and class labels (retrieval_results.csv)

## How to apply

Encode the reference image with a pre-trained SEARLE model (loaded via PyTorch Hub, backbones ViT-B/32 or ViT-L/14) into a pseudo-token within CLIP's text embedding space — SEARLE performs zero-shot textual inversion, so no per-image training is required. Fuse this pseudo-token with the natural-language text prompt to form the composed query embedding, using 'targetpad' image preprocessing (CIR_PREPROCESS_TYPE) as configured in InfoCIR's config.py. Compute cosine similarities between the composed query vector and all pre-computed CLIP image embeddings of the ImageNet-R database (CLIP backbone ViT-B/32, DATASET_SAMPLE_SIZE=30000) and rank every database image. Select the top-k results (k=10 in the paper's evaluation protocol, adjustable in the dashboard's Composed Image Retrieval Panel) and persist the ranked set with rank index, image identifier, similarity score, and class label. The rationale is that CLIP embeds images and text into a joint feature space, so a SEARLE-inverted image pseudo-token can be composed with text and compared against image embeddings with cosine similarity.

## Related tools

- **SEARLE** (Zero-shot textual inversion: transforms the reference image into a pseudo-token within CLIP's text embedding space to form the composed query) — https://github.com/giannhskp/InfoCIR
- **CLIP** (Vision-language backbone (ViT-B/32 or ViT-L/14) providing the joint image/text embedding space in which the pseudo-token and database embeddings are compared) — https://github.com/giannhskp/InfoCIR
- **InfoCIR** (Plotly-Dash dashboard that wraps the SEARLE-based CIR back-end, exposing query composition, adjustable top-k, and result exploration) — https://github.com/giannhskp/InfoCIR

## Examples

```
python -c "import torch; torch.hub.load('miccunifi/SEARLE', 'searle', backbone='ViT-B/32')"
```

## Evaluation signals

- retrieval_results.csv exists and contains exactly k=10 ranked rows, each with a rank index, image identifier, cosine similarity score in [-1, 1], and a class label field
- All returned similarity scores are bounded within [-1, 1], as expected for cosine similarity between CLIP-space vectors
- SEARLE model loads successfully via torch.hub for the configured backbone ('ViT-B/32' default, 'ViT-L/14' for SEARLE-XL)
- Top-k rankings respond monotonically to prompt changes: LLM-enhanced prompts measurably alter retrieval metrics (NDCG@K, mAP@K, MRR, Mean Rank) recorded in the Rank-Δ heatmap
- Dataset indexing completes on first run: CLIP embeddings and the search index are built for all sampled images (up to DATASET_SAMPLE_SIZE=30000)

## Limitations

- InfoCIR's authors acknowledge limited CIR model support as a limitation; future work must extend model support, scalability, and personalization
- CLIP-based embedding spaces can suffer from style bias, where UMAP projects cluster images by artistic style rather than semantic categories, so raw similarity rankings may reflect style as well as semantics
- App runs on a local server (port 8051) and recommends a CUDA-capable GPU and 8–16GB RAM for large datasets; first run performs full feature extraction and index building

## Evidence

- [methods] Textual inversion of reference image into pseudo-token: [evidence span withheld — anchor status: not_found]
- [methods] Cosine similarity ranking of database images: "The system then calculates cosine similarities between this vector and pre-computed database embeddings to generate the final image rankings."
- [readme] SEARLE integration in the app: "Uses pre-trained SEARLE models for high-quality composed queries"
- [readme] Query construction pattern: "Reference Image + Text Modification → Composed Query"
