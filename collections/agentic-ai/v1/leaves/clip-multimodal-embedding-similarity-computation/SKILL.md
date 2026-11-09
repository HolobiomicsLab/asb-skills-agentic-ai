---
name: clip-multimodal-embedding-similarity-computation
description: Use when a composed image retrieval query (reference image + natural-language
  modification prompt) must be matched against a pre-computed CLIP embedding database,
  e.g. ranking top-k results for a query like an image of a dog plus 'wearing a red
  collar'.
when_to_use_negative:
- The task is plain text-to-image or image-to-image retrieval without a reference-image-plus-text
  composed query — a standard CLIP zero-shot encoder suffices and no textual inversion
  is needed
- The retrieval model is not CLIP-based (InfoCIR itself notes limited CIR model support
  as a limitation and defers extending model support to future work)
- You only need to interpret or explain an already-computed ranking rather than produce
  one — use Grad-ECLIP saliency/token attribution instead of recomputing similarities
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: CLIP
  role: Vision-language backbone embedding images and text into a joint feature space;
    provides the image and text towers whose cosine similarity ranks candidates
- name: SEARLE
  role: Zero-shot textual-inversion model that transforms the reference image into
    a pseudo-token within CLIP's text embedding space to build the composed query
- name: PyTorch Hub
  role: Loads pretrained SEARLE models (searle, backbone='ViT-B/32' or 'ViT-L/14')
    for query construction
  repo: https://github.com/giannhskp/InfoCIR
- name: InfoCIR
  role: 'Dash-based system hosting the retrieval workflow: composed query construction,
    cosine-similarity ranking, and downstream explanation panels'
  repo: https://github.com/giannhskp/InfoCIR
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1109/pacificvis68791.2026.00041
    title: 'InfoCIR: Multimedia Analysis for Composed Image Retrieval'
  source_content_hash: sha256:8f8ab6229b235c10245ee3f05d5649a3fceb86dd425e3e3cb512c60a8cd9478c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giannhskp/InfoCIR
  tools:
  - CLIP
  - GradECLIP
  - InfoCIR
  - PyTorch Hub
  - SEARLE
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# clip-multimodal-embedding-similarity-computation

## Summary

Compute cosine similarities between CLIP image embeddings and composed-query text embeddings in CLIP's joint feature space, using a SEARLE textual-inversion pseudo-token to fuse a reference image with a text prompt. This is the ranking backbone of composed image retrieval in InfoCIR.

## When to use

Use when a composed image retrieval query (reference image + natural-language modification prompt) must be matched against a pre-computed CLIP embedding database, e.g. ranking top-k results for a query like an image of a dog plus 'wearing a red collar'.

## When NOT to use

- The task is plain text-to-image or image-to-image retrieval without a reference-image-plus-text composed query — a standard CLIP zero-shot encoder suffices and no textual inversion is needed
- The retrieval model is not CLIP-based (InfoCIR itself notes limited CIR model support as a limitation and defers extending model support to future work)
- You only need to interpret or explain an already-computed ranking rather than produce one — use Grad-ECLIP saliency/token attribution instead of recomputing similarities

## Inputs

- Reference image file (user-uploaded)
- Natural-language modification prompt (e.g. 'wearing sunglasses', 'make it red')
- Pre-computed CLIP image embedding database (e.g. ImageNet-R, up to DATASET_SAMPLE_SIZE=30000 images)
- Pretrained SEARLE checkpoint for CLIP backbone ViT-B/32 or ViT-L/14
- class_names.csv with columns dir_name,class_name mapping class directories to labels

## Outputs

- Ranked list of top-k retrieved images ordered by cosine similarity to the composed query
- Composed query embedding vector in CLIP's joint feature space
- Per-candidate cosine similarity scores usable as input to saliency/attribution panels

## How to apply

Load a pretrained SEARLE textual-inversion model on a CLIP backbone (ViT-B/32 or ViT-L/14) via PyTorch Hub and encode the reference image into a pseudo-token φ(I_ref) within CLIP's text embedding space. Form the composed prompt 'a photo of φ(I_ref) that C', where C is the user's text modification, and encode it with the CLIP text tower. Compute the joint cosine similarity s = cos(V, T) between the composed text embedding and each candidate image embedding, optionally using TargetPad image preprocessing (CIR_PREPROCESS_TYPE='targetpad'). Rank database images by descending cosine similarity to return the top-k results; pre-compute database embeddings once for fast retrieval. The resulting similarity scores also serve as the substrate for downstream Grad-ECLIP saliency attribution, which differentiates this similarity with respect to input image pixels.

## Related tools

- **CLIP** (Vision-language backbone embedding images and text into a joint feature space; provides the image and text towers whose cosine similarity ranks candidates)
- **SEARLE** (Zero-shot textual-inversion model that transforms the reference image into a pseudo-token within CLIP's text embedding space to build the composed query)
- **PyTorch Hub** (Loads pretrained SEARLE models (searle, backbone='ViT-B/32' or 'ViT-L/14') for query construction) — https://github.com/giannhskp/InfoCIR
- **InfoCIR** (Dash-based system hosting the retrieval workflow: composed query construction, cosine-similarity ranking, and downstream explanation panels) — https://github.com/giannhskp/InfoCIR

## Examples

```
import torch; model = torch.hub.load('miccunifi/SEARLE', 'searle', backbone='ViT-B/32'); s = torch.cosine_similarity(img_emb, txt_emb)
```

## Evaluation signals

- Top-k results are ordered by monotonically decreasing cosine similarity between the composed-query text embedding and candidate image embeddings
- The composed prompt correctly interpolates the reference image pseudo-token, e.g. 'a photo of φ(I_ref) that C' with φ(I_ref) produced by the SEARLE model for the specified CLIP backbone
- Pre-computed database embeddings are consistent with the chosen CLIP model name (CLIP_MODEL_NAME='ViT-B/32' by default) and TargetPad preprocessing, otherwise similarities are invalid
- Retrieval quality is sanity-checked via ranking metrics used by the system (NDCG@K, mAP@K, MRR, Mean Rank) when evaluating enhanced vs original prompts
- The retrieved top-k set agrees with the UMAP embedding view, where the query embedding appears near the highlighted top-k result points

## Limitations

- Limited CIR model support is acknowledged as a limitation; the pipeline assumes SEARLE-style textual inversion on a CLIP backbone
- Style bias in CLIP-based embedding spaces can cluster images by artistic style rather than semantic categories, motivating the debiased UMAP pipeline for the visualization step (not the ranking itself)
- Dataset scale is capped in practice by DATASET_SAMPLE_SIZE (30000 images); scalability for larger collections is deferred to future work

## Evidence

- [methods] Textual inversion of reference image into pseudo-token: [evidence span withheld — anchor status: not_found]
- [methods] Cosine similarity ranking of database images: "The system then calculates cosine similarities between this vector and pre-computed database embeddings to generate the final image rankings."
