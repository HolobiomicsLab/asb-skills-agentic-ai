---
name: gradient-based-image-saliency-attribution
description: Use when a CLIP-based composed image retrieval system returns results
  and you must explain, per retrieved image, which pixel regions and prompt tokens
  drove the match.
when_to_use_negative:
- You need a global/embedding-space view of the whole dataset rather than a per-image
  local explanation — use the debiased UMAP projection instead
- The retrieval model is not CLIP-based, so the image-text joint cosine similarity
  cannot be differentiated through a shared embedding space
- You require a quantitative saliency criterion — the paper reports no such metric
  and validates maps only via expert visual review
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3377
- http://edamontology.org/topic_3474
tools:
- name: GradECLIP
  role: 'Gradient-based saliency technique adapted from scratch: differentiate the
    image-text cosine similarity w.r.t. image pixels through the CLIP image tower'
- name: SEARLE
  role: Textual-inversion back-end that encodes the reference image into the pseudo-token
    φ(I_ref) used in the composed prompt
  repo: https://github.com/giannhskp/InfoCIR
- name: CLIP
  role: Joint vision-language embedding model whose image tower is back-propagated
    through to obtain pixel-space gradients
- name: PyTorch
  role: Framework for the SEARLE torch.hub load, CLIP forward/backward passes, and
    gradient normalization/upsampling
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1109/pacificvis68791.2026.00041
    title: 'InfoCIR: Multimedia Analysis for Composed Image Retrieval'
  source_content_hash: sha256:96089ae636966c34a8831897e0acf75031f5a48687a8099ede22cb3a6aac860e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giannhskp/InfoCIR
  tools:
  - CLIP
  - GradECLIP
  - PyTorch
  - SEARLE
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3377
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# gradient-based-image-saliency-attribution

## Summary

Adapts Grad-ECLIP to produce post-retrieval saliency heatmaps for Composed Image Retrieval: gradients of the image-text cosine similarity are back-propagated through the CLIP image tower to highlight which image regions drive alignment with the composed prompt's tokens.

## When to use

Use when a CLIP-based composed image retrieval system returns results and you must explain, per retrieved image, which pixel regions and prompt tokens drove the match.

## When NOT to use

- You need a global/embedding-space view of the whole dataset rather than a per-image local explanation — use the debiased UMAP projection instead
- The retrieval model is not CLIP-based, so the image-text joint cosine similarity cannot be differentiated through a shared embedding space
- You require a quantitative saliency criterion — the paper reports no such metric and validates maps only via expert visual review

## Inputs

- Pretrained SEARLE model checkpoint on a CLIP backbone (ViT-B/32 or ViT-L/14)
- Reference image (e.g. from ImageNet-R with class_names.csv labels)
- Composed prompt text: 'a photo of φ(I_ref) that C'
- Candidate/retrieved image(s) from the top-k ranking

## Outputs

- saliency_map.png: normalized gradient attribution upsampled to image resolution, rendered as a heatmap overlay on the reference or candidate image
- Token-attribution bar chart of influential prompt tokens (paired text-side explanation)

## How to apply

Load a pretrained SEARLE textual-inversion model on a CLIP backbone (e.g. ViT-B/32 or ViT-L/14 via torch.hub.load('miccunifi/SEARLE', ...)), encode the reference image into a pseudo-token φ(I_ref), and form the prompt 'a photo of φ(I_ref) that C'. Compute the joint cosine similarity s = cos(V, T) between the candidate image embedding V and the CLIP text embedding T of the composed prompt. Differentiate s with respect to the input image pixels by back-propagating through the CLIP image tower, implementing the Grad-ECLIP adaptation from scratch rather than treating saliency as a black box. Normalize the gradient-based attribution, upsample it to image resolution, and render it as a heatmap overlaid on the reference or any retrieved image; optionally pair it with token-level attribution bars. Limit computation to the top-k candidates (e.g. SALIENCY_MAX_CANDIDATES = 5 in the InfoCIR config) since each map requires a separate backward pass.

## Related tools

- **GradECLIP** (Gradient-based saliency technique adapted from scratch: differentiate the image-text cosine similarity w.r.t. image pixels through the CLIP image tower)
- **SEARLE** (Textual-inversion back-end that encodes the reference image into the pseudo-token φ(I_ref) used in the composed prompt) — https://github.com/giannhskp/InfoCIR
- **CLIP** (Joint vision-language embedding model whose image tower is back-propagated through to obtain pixel-space gradients)
- **PyTorch** (Framework for the SEARLE torch.hub load, CLIP forward/backward passes, and gradient normalization/upsampling)

## Examples

```
import torch; m = torch.hub.load('miccunifi/SEARLE', 'searle', backbone='ViT-B/32'); s = torch.cosine_similarity(img_emb, clip.encode_text(tokens)).sum(); s.backward()
```

## Evaluation signals

- saliency_map.png exists and is a valid image file
- Highlighted heatmap regions correspond to areas the model attends to when matching the query (expert visual review)
- Gradient attribution is normalized before upsampling to image resolution
- Token-attribution bars are consistent with the salient image regions for the same composed prompt
- Saliency generation respects the configured candidate budget (e.g. top-5 via SALIENCY_MAX_CANDIDATES)

## Limitations

- No quantitative saliency criterion is reported by the paper; validation relies on expert visual review
- Computation is per-candidate, so saliency for all retrieved images is expensive — cap via SALIENCY_MAX_CANDIDATES
- Scalability and broader model support are acknowledged as future work, so the CLIP/SEARLE coupling may not transfer directly to non-CLIP CIR back-ends

## Evidence

- [methods] Grad-ECLIP adaptation applied post-retrieval: [evidence span withheld — anchor status: not_found]
- [methods] Gradient backprop through CLIP image tower: [evidence span withheld — anchor status: not_found]
- [methods] No quantitative saliency criterion: [evidence span withheld — anchor status: not_found]
