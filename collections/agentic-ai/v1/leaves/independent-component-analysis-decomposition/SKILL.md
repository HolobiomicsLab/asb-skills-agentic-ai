---
name: independent-component-analysis-decomposition
description: Use when CLIP embeddings clustered by an embedding-space visualization
  group images by artistic style rather than semantic class, and a debiased projection
  pipeline needs an alternative projection after PCA style debiasing and contrastive
  debiasing.
when_to_use_negative:
- Input embeddings have not yet passed style debiasing (stage A) and contrastive debiasing
  (stage B); ICA is the third stage, not a replacement for them
- The analysis goal is a supervised metric projection pipeline without style bias
  issues — if the UMAP projection already clusters semantically, stage (C) adds no
  value
- Input is raw image data or text prompts rather than a numeric embedding matrix
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: scikit-learn
  role: Provides the Independent Component Analysis decomposition fitted on the debiased
    CLIP embedding matrix and the unmixing transform
- name: InfoCIR
  role: 'Defines the three-stage filtering pipeline whose stage (C) applies ICA, via
    the ''alternative_projection'': ''ica'' setting in NEW_UMAP_CONFIG'
  repo: https://github.com/giannhskp/InfoCIR
- name: CLIP
  role: Supplies the joint image-text embedding space whose ImageNet-R embeddings
    are debiased and then ICA-transformed
- name: UMAP
  role: Consumes the ICA-filtered embeddings for the low-dimensional projection whose
    style bias the filtering pipeline corrects
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1109/pacificvis68791.2026.00041
    title: 'InfoCIR: Multimedia Analysis for Composed Image Retrieval'
  source_content_hash: sha256:14aeb45b8f596ec863ff8560d8441fbdd94871761900bfec0c3726c68126284a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giannhskp/InfoCIR
  tools:
  - CLIP
  - InfoCIR
  - UMAP
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# independent-component-analysis-decomposition

## Summary

Stage (C) of InfoCIR's ImageNet-R embedding filtering pipeline: fit an Independent Component Analysis decomposition on contrastively-debiased CLIP embeddings and apply the unmixing transform, so that semantic signals are separated from stylistic noise before UMAP projection.

## When to use

Use when CLIP embeddings clustered by an embedding-space visualization group images by artistic style rather than semantic class, and a debiased projection pipeline needs an alternative projection after PCA style debiasing and contrastive debiasing.

## When NOT to use

- Input embeddings have not yet passed style debiasing (stage A) and contrastive debiasing (stage B); ICA is the third stage, not a replacement for them
- The analysis goal is a supervised metric projection pipeline without style bias issues — if the UMAP projection already clusters semantically, stage (C) adds no value
- Input is raw image data or text prompts rather than a numeric embedding matrix

## Inputs

- Contrastively-debiased CLIP embedding matrix of the ImageNet-R dataset (output of pipeline stages A and B)
- InfoCIR UMAP-enhancement configuration dict (NEW_UMAP_CONFIG) with 'alternative_projection': 'ica'

## Outputs

- Fitted ICA unmixing matrix from the decomposition
- Source-transformed (ICA-space) embedding matrix saved as a .npy file, shaped (n_samples, n_components)

## How to apply

Load the contrastively-debiased CLIP embeddings of ImageNet-R produced by pipeline stages (A) PCA-based style debiasing (retaining dimensions with higher semantic discriminability, e.g. pca_components=80 in NEW_UMAP_CONFIG) and (B) contrastive debiasing against class prototypes. Fit a scikit-learn ICA decomposition on the debiased embedding matrix as configured by 'alternative_projection': 'ica' in the InfoCIR UMAP-enhancement settings, then extract the ICA unmixing matrix and apply it to obtain the source-transformed (ICA-space) embedding matrix, saving the result as a .npy file. The rationale, per the paper, is that ICA acts as the final 'Alternative Projection' stage (C) that separates semantic signals from stylistic noise so that the downstream UMAP projection clusters by class rather than by artistic style.

## Related tools

- **scikit-learn** (Provides the Independent Component Analysis decomposition fitted on the debiased CLIP embedding matrix and the unmixing transform)
- **InfoCIR** (Defines the three-stage filtering pipeline whose stage (C) applies ICA, via the 'alternative_projection': 'ica' setting in NEW_UMAP_CONFIG) — https://github.com/giannhskp/InfoCIR
- **CLIP** (Supplies the joint image-text embedding space whose ImageNet-R embeddings are debiased and then ICA-transformed)
- **UMAP** (Consumes the ICA-filtered embeddings for the low-dimensional projection whose style bias the filtering pipeline corrects)

## Examples

```
from sklearn.decomposition import FastICA; ica = FastICA(n_components=80); S = ica.fit_transform(X_debiased); np.save('ica_sources.npy', S)
```

## Evaluation signals

- Both .npy artifacts (unmixing matrix and transformed embedding matrix) exist after the run
- The ICA-space source matrix has shape (n_samples, n_components) matching the number of input embeddings
- The transformed representation exhibits improved class-level separability, i.e. reduced stylistic grouping consistent with the paper's claim that ICA separates semantic signals from stylistic noise
- The configuration dict used contains 'alternative_projection': 'ica', confirming stage (C) was selected over other alternative projections
- Input embeddings can be traced back to upstream stages: PCA style debiasing retained dimensions with higher semantic discriminability and contrastive debiasing aligned embeddings with class prototypes

## Limitations

- The paper acknowledges scalability as a limitation deferred to future work; ICA on large embedding matrices (the README configures DATASET_SAMPLE_SIZE = 30000) may be expensive
- Model support is limited; extending model support beyond SEARLE/CLIP backbones is noted as future work
- ICA is only the third stage of the pipeline — skipping stages (A) and (B) is not supported by the described workflow

## Evidence

- [methods] Three-stage pipeline: [evidence span withheld — anchor status: not_found]
