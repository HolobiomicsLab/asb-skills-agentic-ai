---
name: semantic-vs-style-signal-disentanglement
description: Use when UMAP or other dimensionality-reduction plots of CLIP embeddings
  cluster images by artistic style rather than semantic category, and debiased projections
  are needed for composed image retrieval diagnosis.
when_to_use_negative:
- The embedding space shows no style bias — UMAP already clusters images by semantic
  class, so the extra debiasing stages are unnecessary
- Input is not an image embedding matrix from a vision-language model like CLIP but
  already a low-dimensional or purely semantic feature table
- The goal is retrieval ranking itself rather than visualization/clustering diagnosis;
  the pipeline targets the debiased UMAP projection, not the cosine-similarity ranking
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0091
tools:
- name: CLIP
  role: Provides the joint vision-language embedding space of the images that the
    debiasing pipeline operates on
- name: scikit-learn
  role: Fits the PCA and Independent Component Analysis decompositions used for style
    debiasing and the alternative ICA projection
- name: UMAP
  role: Produces the final 2D low-dimensional projection from the debiased embeddings
    for spatial reasoning in the dashboard
- name: InfoCIR
  role: Hosts the three-stage filtering pipeline and the resulting debiased UMAP projection
    in the Plotly-Dash embedding view
  repo: https://github.com/giannhskp/InfoCIR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1109/pacificvis68791.2026.00041
    title: 'InfoCIR: Multimedia Analysis for Composed Image Retrieval'
  source_content_hash: sha256:120257c499f9ba25ad01251b162845f81cfc1ae557a8833af981da0d8c7f2873
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giannhskp/InfoCIR
  tools:
  - CLIP
  - InfoCIR
  - UMAP
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Semantic vs Style Signal Disentanglement

## Summary

Separates semantic class signal from stylistic noise in CLIP image embeddings via a three-stage filtering pipeline — PCA-based Style Debiasing, Contrastive Debiasing toward class prototypes, and Independent Component Analysis (ICA) as an alternative projection — before UMAP visualization in the InfoCIR system.

## When to use

Use when UMAP or other dimensionality-reduction plots of CLIP embeddings cluster images by artistic style rather than semantic category, and debiased projections are needed for composed image retrieval diagnosis.

## When NOT to use

- The embedding space shows no style bias — UMAP already clusters images by semantic class, so the extra debiasing stages are unnecessary
- Input is not an image embedding matrix from a vision-language model like CLIP but already a low-dimensional or purely semantic feature table
- The goal is retrieval ranking itself rather than visualization/clustering diagnosis; the pipeline targets the debiased UMAP projection, not the cosine-similarity ranking

## Inputs

- Pre-computed CLIP image embeddings of a dataset (e.g., ImageNet-R, CLIP ViT-B/32 backbone)
- Class labels file (class_names.csv with columns dir_name,class_name) for class prototypes and supervised debiasing
- UMAP enhancement configuration (NEW_UMAP_CONFIG) specifying style_debiasing, contrastive_debiasing, semantic_enhancement, and alternative_projection='ica'

## Outputs

- Debiased embedding matrix with stylistic components suppressed
- ICA unmixing matrix from the fitted scikit-learn decomposition
- ICA-space (source-transformed) embedding matrix saved as a .npy file with shape (n_samples, n_components)
- 2D debiased UMAP projection used for the InfoCIR Embedding View

## How to apply

Load pre-computed CLIP embeddings of the dataset (e.g., ImageNet-R, ViT-B/32 backbone, up to DATASET_SAMPLE_SIZE=30000 images) and pass them through three ordered stages configured in NEW_UMAP_CONFIG. Stage (A) Style Debiasing uses PCA-based filtering to retain dimensions with higher semantic discriminability (pca_components=80); stage (B) Contrastive Debiasing encourages embeddings to align more closely with their class prototypes; stage (C) applies Independent Component Analysis (ICA) as the alternative projection ('alternative_projection': 'ica') using scikit-learn to separate semantic signals from stylistic noise. Save the ICA unmixing matrix and the source-transformed embedding matrix as .npy files, then feed the debiased representation into UMAP (n_neighbors=25, min_dist=0.1, n_epochs=1200) for the 2D Embedding View. Rationale: CLIP spaces carry style bias, so raw UMAP projections group images by artistic style, undermining the diagnose-compare-enhance loop of CIR model development.

## Related tools

- **CLIP** (Provides the joint vision-language embedding space of the images that the debiasing pipeline operates on)
- **scikit-learn** (Fits the PCA and Independent Component Analysis decompositions used for style debiasing and the alternative ICA projection)
- **UMAP** (Produces the final 2D low-dimensional projection from the debiased embeddings for spatial reasoning in the dashboard)
- **InfoCIR** (Hosts the three-stage filtering pipeline and the resulting debiased UMAP projection in the Plotly-Dash embedding view) — https://github.com/giannhskp/InfoCIR

## Examples

```
NEW_UMAP_CONFIG = {'style_debiasing': True, 'contrastive_debiasing': True, 'alternative_projection': 'ica'}; ica = FastICA(n_components=80); S = ica.fit_transform(debiased_embeddings)
```

## Evaluation signals

- Both .npy artifacts exist after the ICA stage (unmixing matrix and transformed source matrix)
- The ICA-space source matrix has shape (n_samples, n_components) matching the input embedding count
- Transformed representation exhibits improved class-level separability — reduced stylistic grouping in the UMAP plot — consistent with the claim that ICA separates semantic signals from stylistic noise
- The UMAP projection now groups images by semantic category rather than artistic style compared to the raw-embedding baseline
- NEW_UMAP_CONFIG flags are set as intended: style_debiasing=True, contrastive_debiasing=True, alternative_projection='ica'

## Limitations

- Debiasing targets the UMAP visualization path; the retrieval ranking itself still uses cosine similarities between pre-computed database embeddings
- Upstream dataset processing caps the number of images (DATASET_SAMPLE_SIZE = 30000), so very large datasets are subsampled before debiasing
- Scalability and personalization remain acknowledged future work in the paper's discussion
- UMAP hyperparameters (pca_components=80, n_neighbors=25, min_dist=0.1, spread=55.0, n_epochs=1200) are tuned for the ImageNet-R / SEARLE setup and may need adjustment elsewhere

## Evidence

- [methods] Three-stage filtering pipeline order and roles: [evidence span withheld — anchor status: not_found]
