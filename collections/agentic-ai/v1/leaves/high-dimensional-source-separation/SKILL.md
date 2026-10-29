---
name: high-dimensional-source-separation
description: Use when dimensionality reductions of CLIP-based embedding spaces cluster
  images by artistic style rather than semantic class, and upstream PCA style debiasing
  plus contrastive debiasing have already been applied.
when_to_use_negative:
- Input embeddings have not undergone stage (A) PCA style debiasing and stage (B)
  contrastive debiasing — ICA is specified as the third stage operating on already-debiased
  embeddings
- The goal is ordinary dimensionality reduction for retrieval ranking (cosine similarity
  over pre-computed database embeddings) rather than removing style bias from a UMAP
  projection space
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: scikit-learn
  role: Provides the Independent Component Analysis decomposition fitted on the debiased
    CLIP embedding matrix and its unmixing transform
- name: CLIP
  role: Source of the joint vision-language embeddings (ImageNet-R) that the ICA stage
    projects
- name: InfoCIR
  role: Hosts the three-stage filtering pipeline and exposes ICA via the 'alternative_projection'
    setting in its UMAP-enhancement configuration
  repo: https://github.com/giannhskp/InfoCIR
- name: UMAP
  role: Consumes the ICA-transformed embeddings for low-dimensional spatial projection
    of top-k results
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1109/pacificvis68791.2026.00041
    title: 'InfoCIR: Multimedia Analysis for Composed Image Retrieval'
  source_content_hash: sha256:a45889d715185eed9063056c49eb9230984df9568a11974af9d97b269c9a8b9b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giannhskp/InfoCIR
  tools:
  - CLIP
  - InfoCIR
  - UMAP
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# high-dimensional-source-separation

## Summary

Stage (C) of InfoCIR's three-stage embedding filtering pipeline: apply Independent Component Analysis (ICA) as an alternative projection on debiased CLIP embeddings to separate semantic signals from stylistic noise before UMAP visualization.

## When to use

Use when dimensionality reductions of CLIP-based embedding spaces cluster images by artistic style rather than semantic class, and upstream PCA style debiasing plus contrastive debiasing have already been applied.

## When NOT to use

- Input embeddings have not undergone stage (A) PCA style debiasing and stage (B) contrastive debiasing — ICA is specified as the third stage operating on already-debiased embeddings
- The goal is ordinary dimensionality reduction for retrieval ranking (cosine similarity over pre-computed database embeddings) rather than removing style bias from a UMAP projection space

## Inputs

- Contrastively-debiased CLIP embedding matrix for ImageNet-R (post stage A PCA style debiasing and stage B contrastive debiasing), as float arrays loaded for scikit-learn
- InfoCIR NEW_UMAP_CONFIG settings with 'alternative_projection': 'ica'

## Outputs

- ICA unmixing matrix from the fitted sklearn decomposition
- Source-transformed (ICA-space) embedding matrix with shape (n_samples, n_components), saved as a .npy file

## How to apply

Load the ImageNet-R CLIP embeddings that have passed through the pipeline's earlier stages: (A) PCA-based style debiasing retaining dimensions with higher semantic discriminability, and (B) contrastive debiasing, which encourages embeddings to align more closely with their class prototypes. Fit an Independent Component Analysis decomposition on the debiased embedding matrix using scikit-learn, enabled in the InfoCIR UMAP-enhancement config via 'alternative_projection': 'ica'. Extract the fitted ICA unmixing matrix and apply it to the debiased embeddings to obtain a source-transformed embedding matrix of shape (n_samples, n_components), saved as a .npy file. The rationale is that ICA, as the third filtering stage, separates independent sources so that stylistic factors no longer dominate the semantic structure prior to UMAP projection of top-k retrieval results.

## Related tools

- **scikit-learn** (Provides the Independent Component Analysis decomposition fitted on the debiased CLIP embedding matrix and its unmixing transform)
- **CLIP** (Source of the joint vision-language embeddings (ImageNet-R) that the ICA stage projects)
- **InfoCIR** (Hosts the three-stage filtering pipeline and exposes ICA via the 'alternative_projection' setting in its UMAP-enhancement configuration) — https://github.com/giannhskp/InfoCIR
- **UMAP** (Consumes the ICA-transformed embeddings for low-dimensional spatial projection of top-k results)

## Examples

```
from sklearn.decomposition import FastICA; ica = FastICA(n_components=80); S = ica.fit_transform(X_debiased); np.save('ica_sources.npy', S)
```

## Evaluation signals

- Both .npy artifacts exist after the run: the saved ICA unmixing matrix and the transformed embedding matrix
- The transformed source matrix has shape (n_samples, n_components) matching the input embedding count
- The ICA-space representation exhibits improved class-level separability in downstream UMAP plots (reduced stylistic grouping), consistent with the claim that ICA separates semantic signals from stylistic noise
- The run used the configured 'alternative_projection': 'ica' entry in NEW_UMAP_CONFIG, alongside 'style_debiasing': True and 'contrastive_debiasing': True

## Limitations

- ICA is positioned as an 'alternative projection' — the paper frames it as one option within the debiased UMAP enhancement, not a replacement for stages (A) and (B)
- Effectiveness depends on upstream debiasing quality; the paper motivates the whole filter chain by the observation that UMAP on raw CLIP spaces clusters images by artistic style rather than semantic categories
- InfoCIR acknowledges limited CIR model support and scalability as open limitations, which bounds the settings in which this pipeline has been validated

## Evidence

- [methods] Three-stage filter definition: [evidence span withheld — anchor status: not_found]
