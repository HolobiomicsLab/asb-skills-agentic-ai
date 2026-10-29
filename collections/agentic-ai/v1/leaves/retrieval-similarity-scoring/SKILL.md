---
name: retrieval-similarity-scoring
description: Use when you must rank a database of images against a composed query
  (reference image + natural-language modification prompt, e.g., 'make it red') in
  a joint CLIP embedding space to produce top-k retrieval results.
when_to_use_negative:
- The retrieval task is unimodal (text-only or image-only search) rather than composed
  image+text CIR
- The database embeddings are not in the same CLIP joint space as the SEARLE-composed
  query (mismatched backbone or preprocessing)
- The goal is attribution or saliency analysis rather than producing a similarity
  ranking
edam_operation: http://edamontology.org/operation_0291
tools:
- name: SEARLE
  role: Performs zero-shot textual inversion of the reference image into a pseudo-token
    within CLIP's text embedding space to compose the query
  repo: https://github.com/giannhskp/InfoCIR
- name: CLIP
  role: Joint vision-language embedding model whose text encoder embeds the composed
    prompt and whose image encoder produced the pre-computed database embeddings for
    cosine-similarity ranking
  repo: https://github.com/giannhskp/InfoCIR
- name: InfoCIR
  role: Plotly-Dash dashboard that hosts the retrieval workflow, displays top-k results
    in the Query Results Panel, and lets users mark ideal targets
  repo: https://github.com/giannhskp/InfoCIR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1109/pacificvis68791.2026.00041
    title: 'InfoCIR: Multimedia Analysis for Composed Image Retrieval'
  source_content_hash: sha256:64aeaa21c05404c8749a39542bac53ec87ad4833af3b909d019621fb92333f6f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giannhskp/InfoCIR
  tools:
  - CLIP
  - InfoCIR
  - SEARLE
  edam_operation: http://edamontology.org/operation_0291
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# retrieval-similarity-scoring

## Summary

Computes cosine similarity scores between a composed image+text query embedding (built with SEARLE textual inversion into CLIP's joint embedding space) and pre-computed CLIP database embeddings to produce ranked top-k image retrieval results, forming the scoring backbone of InfoCIR's Composed Image Retrieval workflow.

## When to use

Use when you must rank a database of images against a composed query (reference image + natural-language modification prompt, e.g., 'make it red') in a joint CLIP embedding space to produce top-k retrieval results.

## When NOT to use

- The retrieval task is unimodal (text-only or image-only search) rather than composed image+text CIR
- The database embeddings are not in the same CLIP joint space as the SEARLE-composed query (mismatched backbone or preprocessing)
- The goal is attribution or saliency analysis rather than producing a similarity ranking

## Inputs

- Reference image (uploaded file, targetpad-preprocessed)
- Natural-language modification prompt (e.g., 'wearing a red collar')
- Pre-computed CLIP embeddings for the image database (ImageNet-R, up to 30000 images)
- Pre-trained SEARLE model weights (ViT-B/32 or ViT-L/14) via PyTorch Hub

## Outputs

- Ranked list of top-k database images ordered by cosine similarity to the composed query
- Per-image similarity scores feeding the Query Results Panel and the Rank-Δ heatmap
- Scalar query-target similarity s used for gradient-based token attribution

## How to apply

Compose the query by encoding the reference image with a pre-trained SEARLE model (backbone ViT-B/32 or ViT-L/14, loaded via PyTorch Hub), which performs textual inversion by transforming the reference image into a pseudo-token within CLIP's text embedding space, then tokenize the full composed prompt with CLIP's text encoder. Preprocess reference images with the 'targetpad' preprocessing type and encode the database once to obtain pre-computed embeddings (up to DATASET_SAMPLE_SIZE=30000 images in InfoCIR). Calculate cosine similarities between the composed query vector and every pre-computed database embedding, then sort to generate the final image rankings and select the top-k requested by the user. This scalar similarity score s = sim(f_c(I_ref, T'), f_i(I*)) is also the quantity reused downstream: it is back-propagated for Grad-ECLIP gradient attribution and compared across LLM-enhanced prompt candidates for the Rank-Δ heatmap and NDCG@K, mAP@K, MRR metrics. Configure the method via config keys such as CIR_EVAL_TYPE='searle', CLIP_MODEL_NAME='ViT-B/32', and IMAGE_GALLERY_SIZE for how many results to display.

## Related tools

- **SEARLE** (Performs zero-shot textual inversion of the reference image into a pseudo-token within CLIP's text embedding space to compose the query) — https://github.com/giannhskp/InfoCIR
- **CLIP** (Joint vision-language embedding model whose text encoder embeds the composed prompt and whose image encoder produced the pre-computed database embeddings for cosine-similarity ranking) — https://github.com/giannhskp/InfoCIR
- **InfoCIR** (Plotly-Dash dashboard that hosts the retrieval workflow, displays top-k results in the Query Results Panel, and lets users mark ideal targets) — https://github.com/giannhskp/InfoCIR

## Examples

```
python -c "import torch; torch.hub.load('miccunifi/SEARLE', 'searle', backbone='ViT-B/32')"  # then run the app: cd cir_app && python run.py
```

## Evaluation signals

- Cosine similarity scores fall within [-1, 1] and the returned ranking is monotonically ordered by score
- Top-k results are semantically consistent with the requested modification (e.g., stronger activation on 'red' images after refining the prompt toward 'make it red')
- Same composed query reproduces the same ranking across runs given identical CLIP backbone (ViT-B/32) and targetpad preprocessing
- Selected ideal target images from the Query Results Panel can be re-scored so that enhanced prompt candidates improve NDCG@K, mAP@K, and MRR metrics
- The similarity score s is differentiable with respect to the input tokens, enabling back-propagation for token attribution

## Limitations

- Limited CIR model support is acknowledged as a limitation; extending model support is deferred to future work
- Scalability and personalization are not yet addressed (Future work can extend model support, scalability and personalization)
- CLIP-based embedding spaces suffer style bias, which clusters images by artistic style rather than semantics (mitigated in projection via debiasing, but the raw similarity space is affected)

## Evidence
