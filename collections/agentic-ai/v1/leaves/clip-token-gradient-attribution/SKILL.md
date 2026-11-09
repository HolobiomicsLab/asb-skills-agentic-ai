---
name: clip-token-gradient-attribution
description: Use when a composed image retrieval pipeline (reference image + text
  prompt) needs local explanation of which prompt tokens drove the retrieval match,
  e.g. verifying that attribution mass shifts onto 'red' after prompt refinement.
when_to_use_negative:
- The analysis goal is spatial/visual explanation of image regions — use the GradECLIP-based
  saliency map (F1) instead of token attribution
- The task is retrieval itself (ranking, top-k selection) rather than explaining an
  existing retrieval match
- The retrieval backend is not CLIP-embedding based (e.g. non-vision-language models)
  — the method depends on gradients flowing through CLIP's text encoder
edam_operation: http://edamontology.org/operation_0282
tools:
- name: CLIP
  role: Vision-language encoder whose text tower is back-propagated through to obtain
    per-token gradients of the cosine similarity score
- name: SEARLE
  role: Performs zero-shot textual inversion of the reference image into a pseudo-token
    within CLIP's text embedding space before tokenizing the composed prompt
  repo: miccunifi/SEARLE (PyTorch Hub)
- name: PyTorch
  role: Autograd engine used to back-propagate the similarity gradient to each input
    token
- name: Grad-ECLIP
  role: Gradient-based attribution method adapted to CLIP by back-propagating through
    both the image and text towers to compute signed sub-word influence scores
- name: InfoCIR
  role: Hosts the Explanation Panel that visualizes token attribution bars (F2) alongside
    saliency maps (F1) and the Rank-Δ heatmap (F3)
  repo: https://github.com/giannhskp/InfoCIR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1109/pacificvis68791.2026.00041
    title: 'InfoCIR: Multimedia Analysis for Composed Image Retrieval'
  source_content_hash: sha256:b8ef48c36d0e8c7b539586b7e530b1ae72b59f0c7d1e03c43a2109be304ad653
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CLIP
  - Grad-ECLIP
  - InfoCIR
  - PyTorch
  - SEARLE
  edam_operation: http://edamontology.org/operation_0282
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# clip-token-gradient-attribution

## Summary

Gradient-based per-token attribution for composed image retrieval queries: back-propagate the cosine-similarity score through CLIP's text encoder to compute signed influence scores for each sub-word, which InfoCIR renders as the token attribution bar chart (Explanation Panel F2) coupled to a Rank-Δ heatmap.

## When to use

Use when a composed image retrieval pipeline (reference image + text prompt) needs local explanation of which prompt tokens drove the retrieval match, e.g. verifying that attribution mass shifts onto 'red' after prompt refinement.

## When NOT to use

- The analysis goal is spatial/visual explanation of image regions — use the GradECLIP-based saliency map (F1) instead of token attribution
- The task is retrieval itself (ranking, top-k selection) rather than explaining an existing retrieval match
- The retrieval backend is not CLIP-embedding based (e.g. non-vision-language models) — the method depends on gradients flowing through CLIP's text encoder

## Inputs

- Composed query: reference image + natural language text prompt (e.g. 'make it red', 'wearing a red collar')
- SEARLE pseudo-token embedding of the reference image (via torch.hub, backbone ViT-B/32 or ViT-L/14)
- Embedding of a user-selected ideal target image (top-k candidate)
- CLIP text encoder tokenized prompt

## Outputs

- token_attribution.csv: one row per prompt token with a signed influence score
- Token attribution bar chart (Explanation Panel F2) coupled to a Rank-Δ heatmap (F3)

## How to apply

Load the composed query and encode the reference image with SEARLE, which performs textual inversion to produce a pseudo-token within CLIP's text embedding space, then tokenize the full composed prompt with CLIP's text encoder (e.g. ViT-B/32). Compute the cosine similarity s = sim(f_c(I_ref, T'), f_i(I*)) between the composed text embedding and the embedding of a user-selected ideal target image. Back-propagate the gradient of s with respect to each input token using PyTorch autograd through the CLIP text encoder, using the Grad-ECLIP method adapted to CLIP by back-propagating through both the image and text towers. Aggregate the per-token gradients into a single signed influence score per token and export the rows as a (token, attribution score) table, e.g. token_attribution.csv. Enable this via SALIENCY_GENERATE_TEXT_ATTRIBUTION = True in config.py and visualize the scores as the F2 bar chart in the Explanation Panel.

## Related tools

- **CLIP** (Vision-language encoder whose text tower is back-propagated through to obtain per-token gradients of the cosine similarity score)
- **SEARLE** (Performs zero-shot textual inversion of the reference image into a pseudo-token within CLIP's text embedding space before tokenizing the composed prompt) — miccunifi/SEARLE (PyTorch Hub)
- **PyTorch** (Autograd engine used to back-propagate the similarity gradient to each input token)
- **Grad-ECLIP** (Gradient-based attribution method adapted to CLIP by back-propagating through both the image and text towers to compute signed sub-word influence scores)
- **InfoCIR** (Hosts the Explanation Panel that visualizes token attribution bars (F2) alongside saliency maps (F1) and the Rank-Δ heatmap (F3)) — https://github.com/giannhskp/InfoCIR

## Examples

```
SALIENCY_GENERATE_TEXT_ATTRIBUTION = True  # in cir_app/src/config.py, then python run.py to serve the dashboard with F2 token attribution bars enabled
```

## Evaluation signals

- token_attribution.csv exists and contains one row per prompt token
- Attribution magnitudes are non-negative (signed direction stored separately from magnitude)
- Top-ranked tokens correspond to high-influence terms consistent with the paper's scenario, where attribution weights reflect stronger activation on 'red' after refinement
- Token attribution bar chart (F2) renders in the Explanation Panel when SALIENCY_GENERATE_TEXT_ATTRIBUTION = True
- Attribution scores are consistent with the Rank-Δ heatmap (F3) behavior when prompts are perturbed

## Limitations

- Attribution depends on the chosen ideal target image; scores change when the user selects a different candidate from the top-k results
- Saliency/token attribution is limited by SALIENCY_MAX_CANDIDATES config (e.g. limited to top-5 results), bounding how many candidates can be explained
- Standard embedding and unimodal explainability tools are insufficient for CIR because they lack retrieval-specific signals such as rank position, prompt evolution, and ideal-image anchors
- Requires a CUDA-capable GPU and 8GB+ RAM (16GB+ recommended for large datasets) for practical back-propagation through CLIP

## Evidence

- [methods] token attribution mechanism is gradient-based: [evidence span withheld — anchor status: not_found]
