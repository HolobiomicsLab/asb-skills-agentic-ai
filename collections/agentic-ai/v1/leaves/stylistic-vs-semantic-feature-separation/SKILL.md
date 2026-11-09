---
name: stylistic-vs-semantic-feature-separation
description: Use when CLIP-based embedding projections (e.g., UMAP) cluster images
  by artistic style rather than semantic categories, and dimensions dominated by stylistic
  or background variation must be filtered out before visualization.
when_to_use_negative:
- The input is not a CLIP-style joint image embedding space — the documented failure
  mode (UMAP clustering images by artistic style) is specific to CLIP-based spaces,
  not arbitrary feature tables
- Label supervision is available and supervised alignment is desired — stage (B) Contrastive
  Debiasing, which aligns embeddings with class prototypes, is the appropriate mechanism
  instead of heuristic PCA filtering
- Full original dimensionality is required downstream (e.g., cosine similarity ranking
  against pre-computed database embeddings) — this stage keeps only the 80 retained
  principal components
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3474
tools:
- name: CLIP
  role: Supplies the raw ViT-B/32 image embeddings whose style-dominated dimensions
    the PCA filter removes
- name: InfoCIR
  role: 'Defines the three-stage filtering pipeline and the NEW_UMAP_CONFIG values
    (pca_components=80, ''style_debiasing'': True) that parameterize this stage'
  repo: https://github.com/giannhskp/InfoCIR
- name: scikit-learn
  role: Implements the PCA fit/transform applied to the raw CLIP image embeddings
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1109/pacificvis68791.2026.00041
    title: 'InfoCIR: Multimedia Analysis for Composed Image Retrieval'
  source_content_hash: sha256:4a4dba34e2cf3b599a3153156ef7f8eabbd5f28750488d9419e2e981ddd460d9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giannhskp/InfoCIR
  tools:
  - CLIP
  - InfoCIR
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# stylistic-vs-semantic-feature-separation

## Summary

PCA-based style debiasing — stage (A) of InfoCIR's three-stage filtering process — for CLIP image embeddings: fit PCA with pca_components=80 on ViT-B/32 ImageNet-R embeddings and retain the components with higher semantic discriminability, so downstream UMAP projections cluster images by semantic category rather than artistic style.

## When to use

Use when CLIP-based embedding projections (e.g., UMAP) cluster images by artistic style rather than semantic categories, and dimensions dominated by stylistic or background variation must be filtered out before visualization.

## When NOT to use

- The input is not a CLIP-style joint image embedding space — the documented failure mode (UMAP clustering images by artistic style) is specific to CLIP-based spaces, not arbitrary feature tables
- Label supervision is available and supervised alignment is desired — stage (B) Contrastive Debiasing, which aligns embeddings with class prototypes, is the appropriate mechanism instead of heuristic PCA filtering
- Full original dimensionality is required downstream (e.g., cosine similarity ranking against pre-computed database embeddings) — this stage keeps only the 80 retained principal components

## Inputs

- Raw CLIP image embeddings for ImageNet-R computed with the ViT-B/32 backbone (precomputed via the InfoCIR pipeline)
- Per-image class labels from class_names.csv (columns: dir_name,class_name)

## Outputs

- debiased_embeddings.npy — NumPy array of PCA-projected embeddings with 80 features per image (pca_components=80) and the same number of rows as the raw embeddings

## How to apply

Load the precomputed raw CLIP image embeddings for ImageNet-R (ViT-B/32 backbone, computed via the InfoCIR pipeline) together with per-image class labels from class_names.csv (columns: dir_name,class_name). Fit PCA on the raw embeddings using the InfoCIR configuration value pca_components=80 from the NEW_UMAP_CONFIG block of src/config.py, where 'style_debiasing': True. Project the embeddings onto the retained principal components — the heuristic stage (A) Style Debiasing discards dimensions dominated by stylistic or background variation so the remaining components have higher semantic discriminability. Save the debiased array to debiased_embeddings.npy in NumPy format, preserving one-to-one row correspondence with the raw embeddings. In the full pipeline this stage feeds (B) Contrastive Debiasing, which aligns embeddings with class prototypes, and (C) Alternative Projection via ICA before the debiased UMAP projection. Validate that the .npy file loads, has 80 feature columns, and the same number of rows as the raw embeddings.

## Related tools

- **CLIP** (Supplies the raw ViT-B/32 image embeddings whose style-dominated dimensions the PCA filter removes)
- **InfoCIR** (Defines the three-stage filtering pipeline and the NEW_UMAP_CONFIG values (pca_components=80, 'style_debiasing': True) that parameterize this stage) — https://github.com/giannhskp/InfoCIR
- **scikit-learn** (Implements the PCA fit/transform applied to the raw CLIP image embeddings)

## Examples

```
from sklearn.decomposition import PCA; import numpy as np; debiased = PCA(n_components=80).fit_transform(raw); np.save('debiased_embeddings.npy', debiased)
```

## Evaluation signals

- debiased_embeddings.npy exists and loads with np.load as a valid NumPy array
- Output array has exactly 80 feature columns, matching pca_components=80 in the NEW_UMAP_CONFIG block
- Row count of debiased_embeddings.npy is unchanged relative to the raw CLIP embeddings (one-to-one row correspondence)
- When the debiased components are projected (e.g., with the debiased UMAP approach), ImageNet-R points group by class_name from class_names.csv rather than by artistic style

## Limitations

- The PCA filter is heuristic and unsupervised: it assumes stylistic/background variation occupies discardable components and uses no labels, unlike stage (B) Contrastive Debiasing which aligns embeddings with class prototypes
- The style-bias failure mode is described for CLIP-based spaces reduced by UMAP; behavior on other backbones or non-CLIP feature spaces is not established
- Reducing to 80 principal components discards information from the raw embeddings — original embeddings are still needed for cosine similarity ranking of database images
- The paper introduces ICA as an alternative projection to separate semantic signals from stylistic noise, implying PCA-based filtering alone may not fully disentangle style from semantics

## Evidence

- [methods] Three-stage filtering context on ImageNet-R: [evidence span withheld — anchor status: not_found]
