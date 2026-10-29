---
name: counterfactual-prompt-conditioning
description: Use when a composed image retrieval query (reference image + text prompt)
  underperforms and the user has identified ideal target images in the top-k results
  that the prompt should be re-expressed to reach.
when_to_use_negative:
- The user has not selected any ideal target images — counterfactual conditioning
  on I* is undefined and the enhancer lacks its visual anchors
- The goal is model-level diagnosis of retrieval failure (use saliency maps and token-level
  attribution instead) rather than prompt refinement toward a known target
- The task is unimodal text-only retrieval without a reference image, since the composed-query
  pipeline requires a reference image plus text prompt
edam_operation: http://edamontology.org/operation_0306
tools:
- name: mistralai/Mistral-7B-Instruct-v0.2
  role: Open-weight LLM conditioned on I_ref, T, and the visual features of I* to
    generate the counterfactual prompt variants
  repo: https://github.com/giannhskp/InfoCIR
- name: CLIP
  role: Provides the joint vision-language embedding space (f_c, f_i) in which candidate
    prompts T' are scored against the ideal anchors via cosine similarity
- name: SEARLE
  role: CIR back-end whose textual inversion of the reference image into a pseudo-token
    produces the composed query embedding that the enhanced prompts modify
  repo: https://github.com/giannhskp/InfoCIR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1109/pacificvis68791.2026.00041
    title: 'InfoCIR: Multimedia Analysis for Composed Image Retrieval'
  source_content_hash: sha256:106d7152977d1d221ac85aa1d7ba9811f6a796a7e1ced30383ad5b31200877fa
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giannhskp/InfoCIR
  tools:
  - CLIP
  - SEARLE
  - mistralai/Mistral-7B-Instruct-v0.2
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# counterfactual-prompt-conditioning

## Summary

Generates counterfactual prompt variants for composed image retrieval by conditioning mistralai/Mistral-7B-Instruct-v0.2 on the reference image, the original text prompt, and the visual context of user-selected ideal target images, then re-ranking candidates via CLIP similarity to the ideal anchors.

## When to use

Use when a composed image retrieval query (reference image + text prompt) underperforms and the user has identified ideal target images in the top-k results that the prompt should be re-expressed to reach.

## When NOT to use

- The user has not selected any ideal target images — counterfactual conditioning on I* is undefined and the enhancer lacks its visual anchors
- The goal is model-level diagnosis of retrieval failure (use saliency maps and token-level attribution instead) rather than prompt refinement toward a known target
- The task is unimodal text-only retrieval without a reference image, since the composed-query pipeline requires a reference image plus text prompt

## Inputs

- Reference image I_ref uploaded to the Composed Image Retrieval Panel
- Natural language prompt T describing the desired modification (e.g., 'make it red', 'add a hat')
- User-selected ideal target image(s) I* clicked from the top-k Query Results Panel
- CLIP embeddings of the database and the composed query (SEARLE pseudo-token composition)

## Outputs

- enhanced_prompt_variants JSON file with 10 candidate prompt entries plus grounding inputs (reference image identifier, original prompt, ideal-image identifiers)
- Per-candidate retrieval metrics: NDCG@K, mAP@K, MRR, Coverage, Mean Rank, Mean Similarity
- Ideal-anchored Rank-Delta heatmap showing how each candidate prompt affects the ranking of the selected target images

## How to apply

Accept the composed-query inputs (reference image I_ref and natural language prompt T) from the Composed Image Retrieval Panel, and collect user-selected ideal image(s) I* from the initial top-k results as fixed anchors, with visual context grounding enabled (ENHANCEMENT_USE_CONTEXT = True). Condition mistralai/Mistral-7B-Instruct-v0.2 on I_ref, T, and the visual features of I* to generate 10 candidate prompt variants (ENHANCEMENT_CANDIDATE_PROMPTS = 10). Evaluate each variant T' with retrieval metrics (NDCG@K, mAP@K, MRR, Coverage, Mean Rank, Mean Similarity) and re-rank against the ideal anchor via the similarity score s = sim(f_c(I_ref, T'), f_i(I*)). Export the variants together with their grounding inputs (reference image identifier, original prompt, ideal-image identifiers) as a structured JSON file named enhanced_prompt_variants, then verify empirical effect through the Ideal-anchored Rank-Delta heatmap loop.

## Related tools

- **mistralai/Mistral-7B-Instruct-v0.2** (Open-weight LLM conditioned on I_ref, T, and the visual features of I* to generate the counterfactual prompt variants) — https://github.com/giannhskp/InfoCIR
- **CLIP** (Provides the joint vision-language embedding space (f_c, f_i) in which candidate prompts T' are scored against the ideal anchors via cosine similarity)
- **SEARLE** (CIR back-end whose textual inversion of the reference image into a pseudo-token produces the composed query embedding that the enhanced prompts modify) — https://github.com/giannhskp/InfoCIR

## Examples

```
# src/config.py: ENHANCEMENT_CANDIDATE_PROMPTS = 10; ENHANCEMENT_USE_CONTEXT = True; then cd cir_app && python run.py
```

## Evaluation signals

- enhanced_prompt_variants exists, is valid JSON, and contains exactly ENHANCEMENT_CANDIDATE_PROMPTS = 10 candidate prompt entries
- Each candidate entry is a well-formed textual modifier that can be embedded and re-ranked against the ideal anchor via s = sim(f_c(I_ref, T'), f_i(I*))
- NDCG@K, mAP@K, and MRR for at least one candidate exceed those of the original prompt T
- The Ideal-anchored Rank-Delta heatmap shows the selected target images moving toward better rank positions for chosen variants
- Each entry records its grounding inputs: reference image identifier, original prompt, and ideal-image identifiers

## Limitations

- Candidates are only as good as the SEARLE/CLIP embedding space; style bias in CLIP-based spaces can distort similarity judgments of the generated variants
- The method supports only the configured CIR back-ends (SEARLE, SEARLE-XL, Phi, OTI); extending model support is acknowledged as future work
- Effect verification depends on the user correctly identifying ideal target images; mislabeled anchors propagate into misleading Rank-Delta feedback

## Evidence

- [intro] an interactive relevance-feedback workflow coupling LLM-generated prompt variants with visual verification via saliency maps, token-level attribution, and an Ideal-anchored Rank-Δ heatmap.: [evidence span withheld — anchor status: not_found]
- [methods] the system leverages the open-weight mistralai/Mistral-7B-Instruct-v0.2 large language model (LLM): [evidence span withheld — anchor status: not_found]
