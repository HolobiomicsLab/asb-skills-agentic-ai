---
name: llm-prompt-variant-generation
description: Use when a composed image retrieval query (reference image + text prompt)
  underperforms and the user has selected one or more ideal target images from the
  top-k results to anchor the generation of improved prompt variants.
when_to_use_negative:
- The user has not selected any ideal target images — generation is conditioned on
  the selected ideals, so without anchors the LLM only has the reference image and
  original prompt to work from
- The task is text-only retrieval or unimodal search with no reference image — this
  skill is specific to composed image retrieval where a pseudo-token from the reference
  image is combined with the text prompt
- The goal is model-level diagnosis rather than query refinement — saliency maps and
  token-attribution in the Explanation Panel are the appropriate path instead
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: mistralai/Mistral-7B-Instruct-v0.2
  role: Open-weight LLM conditioned on I_ref, T, and visual features of the selected
    ideal images I* to generate the 10 candidate prompt variants
  repo: https://github.com/giannhskp/InfoCIR
- name: CLIP
  role: Provides the joint embedding space used to compute similarity scores s = sim(f_c(I_ref,
    T'), f_i(I*)) for re-ranking candidate prompts against the ideal anchors
- name: SEARLE
  role: CIR back-end whose composed query vector (pseudo-token from the reference
    image + text prompt) is re-evaluated with each enhanced prompt variant
  repo: https://github.com/giannhskp/InfoCIR
- name: InfoCIR
  role: Plotly-Dash dashboard hosting the Prompt Enhancement Panel, the Rank-Delta
    heatmap, and the NDCG/mAP/MRR metric display
  repo: https://github.com/giannhskp/InfoCIR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1109/pacificvis68791.2026.00041
    title: 'InfoCIR: Multimedia Analysis for Composed Image Retrieval'
  source_content_hash: sha256:89d6761773eb559de756d7b18e3f34dff21986e2f475ae35d51ad01b6410eba4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giannhskp/InfoCIR
  tools:
  - CLIP
  - InfoCIR
  - SEARLE
  - mistralai/Mistral-7B-Instruct-v0.2
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# llm-prompt-variant-generation

## Summary

Generate counterfactual prompt variants for composed image retrieval by conditioning mistralai/Mistral-7B-Instruct-v0.2 on the reference image, original text prompt, and user-selected ideal target images, then score the variants against the ideal anchors with retrieval metrics (NDCG@K, mAP@K, MRR).

## When to use

Use when a composed image retrieval query (reference image + text prompt) underperforms and the user has selected one or more ideal target images from the top-k results to anchor the generation of improved prompt variants.

## When NOT to use

- The user has not selected any ideal target images — generation is conditioned on the selected ideals, so without anchors the LLM only has the reference image and original prompt to work from
- The task is text-only retrieval or unimodal search with no reference image — this skill is specific to composed image retrieval where a pseudo-token from the reference image is combined with the text prompt
- The goal is model-level diagnosis rather than query refinement — saliency maps and token-attribution in the Explanation Panel are the appropriate path instead

## Inputs

- Reference image I_ref from the Composed Image Retrieval Panel
- Natural language modification prompt T (e.g., 'wearing sunglasses')
- User-selected ideal target image(s) I* clicked in the Query Results Panel top-k gallery
- CLIP embeddings of the reference image and candidate prompts (f_c) and of the ideal images (f_i)

## Outputs

- Set of 10 alternative prompt variants T' per generation round
- Per-candidate retrieval metrics: NDCG@K, mAP@K, MRR, Coverage, Mean Rank, Mean Similarity
- Ideal-anchored Rank-Delta heatmap showing ranking changes per candidate prompt
- Structured JSON export (enhanced_prompt_variants) containing variants and their grounding inputs (reference image identifier, original prompt, ideal-image identifiers)

## How to apply

Accept the composed-query inputs (reference image I_ref and natural language prompt T) and collect the user-selected ideal image(s) I* from the top-k results in the Query Results Panel, enabling visual context grounding via ENHANCEMENT_USE_CONTEXT = True. Condition mistralai/Mistral-7B-Instruct-v0.2 on the content of I_ref, T, and the visual features of I* to generate 10 candidate prompt variants (repository default ENHANCEMENT_CANDIDATE_PROMPTS = 10). For each candidate T', compute the similarity score s = sim(f_c(I_ref, T'), f_i(I*)) using CLIP embeddings to re-rank against the ideal anchor. Evaluate each variant with NDCG@K, mAP@K, MRR, Coverage, Mean Rank and Mean Similarity, and verify the empirical effect through the Ideal-anchored Rank-Delta heatmap. Export the variants together with their grounding inputs (reference image identifier, original prompt, ideal-image identifiers) as a structured JSON file.

## Related tools

- **mistralai/Mistral-7B-Instruct-v0.2** (Open-weight LLM conditioned on I_ref, T, and visual features of the selected ideal images I* to generate the 10 candidate prompt variants) — https://github.com/giannhskp/InfoCIR
- **CLIP** (Provides the joint embedding space used to compute similarity scores s = sim(f_c(I_ref, T'), f_i(I*)) for re-ranking candidate prompts against the ideal anchors)
- **SEARLE** (CIR back-end whose composed query vector (pseudo-token from the reference image + text prompt) is re-evaluated with each enhanced prompt variant) — https://github.com/giannhskp/InfoCIR
- **InfoCIR** (Plotly-Dash dashboard hosting the Prompt Enhancement Panel, the Rank-Delta heatmap, and the NDCG/mAP/MRR metric display) — https://github.com/giannhskp/InfoCIR

## Examples

```
ENHANCEMENT_CANDIDATE_PROMPTS = 10; ENHANCEMENT_USE_CONTEXT = True  # in cir_app/src/config.py, then: python run.py
```

## Evaluation signals

- enhanced_prompt_variants JSON file exists, parses as valid JSON, and contains exactly ENHANCEMENT_CANDIDATE_PROMPTS = 10 candidate prompt entries
- Each candidate prompt is a well-formed textual modifier that can be embedded and re-ranked against the ideal anchor via s = sim(f_c(I_ref, T'), f_i(I*))
- At least one candidate prompt improves ranking metrics (NDCG@K, mAP@K, MRR, Mean Rank) relative to the original prompt, visible in the Rank-Delta heatmap as positive rank movement of the selected ideal image(s)
- Grounding metadata (reference image identifier, original prompt, ideal-image identifiers) is present for every exported variant
- Generated variants reflect visual context from the selected ideals, confirming ENHANCEMENT_USE_CONTEXT = True was honored

## Limitations

- Effectiveness depends on the user selecting genuinely ideal anchor images; poorly chosen anchors will steer the LLM toward misleading prompt variants
- Retrieval quality is bounded by the underlying CIR back-end (SEARLE / SEARLE-XL) and the CLIP backbone (ViT-B/32 default), not by the prompt enhancement itself
- The article's discussion notes limited CIR model support, and scalability/personalization are deferred to future work; the enhancement loop was demonstrated on ImageNet-R-style datasets with a class_names.csv layout
- Candidate count and context grounding are fixed configuration defaults (10 prompts, use context = True) in config.py; deviating requires manual reconfiguration

## Evidence

- [methods] InfoCIR's Prompt Enhancement Panel uses an LLM to propose alternative prompts conditioned on the user-selected ideal target images, together with retrieval metrics.: [evidence span withheld — anchor status: not_found]
- [methods] the system leverages the open-weight mistralai/Mistral-7B-Instruct-v0.2 large language model (LLM): [evidence span withheld — anchor status: not_found]
