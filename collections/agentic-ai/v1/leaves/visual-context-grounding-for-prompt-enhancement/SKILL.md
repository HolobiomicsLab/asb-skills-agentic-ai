---
name: visual-context-grounding-for-prompt-enhancement
description: Use when a composed image retrieval query (reference image + text prompt)
  returns poor results and the user has selected ideal target images from the top-k
  ranking that should anchor LLM-generated prompt rewrites.
when_to_use_negative:
- The user has not selected any ideal target images, so there is no visual anchor
  and the counterfactual generation is ungrounded
- The task is single-image or pure text retrieval rather than composed image retrieval
  where a reference image and text prompt are fused via SEARLE/CLIP textual inversion
- The goal is model-internal diagnosis (saliency maps, token attribution) rather than
  prompt refinement — use the Explanation Panel workflow instead
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: mistralai/Mistral-7B-Instruct-v0.2
  role: Open-weight LLM conditioned on I_ref, T, and the visual features of the selected
    ideal images I* to generate alternative prompt variants T'
  repo: https://github.com/giannhskp/InfoCIR
- name: CLIP
  role: Provides the joint vision-language embedding space used to compute similarity
    s = sim(f_c(I_ref, T'), f_i(I*)) between each candidate prompt and the ideal anchor
  repo: https://github.com/giannhskp/InfoCIR
- name: SEARLE
  role: CIR back-end that performs textual inversion of the reference image into a
    pseudo-token; embeddings it produces are re-ranked with each generated prompt
    variant
  repo: https://github.com/giannhskp/InfoCIR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1109/pacificvis68791.2026.00041
    title: 'InfoCIR: Multimedia Analysis for Composed Image Retrieval'
  source_content_hash: sha256:d231552a2c1453a77e452b9cc31906b7bf2782bbe9fe216e9e96a5ef514152f9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giannhskp/InfoCIR
  tools:
  - CLIP
  - SEARLE
  - mistralai/Mistral-7B-Instruct-v0.2
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# visual-context-grounding-for-prompt-enhancement

## Summary

Generates counterfactual prompt variants for composed image retrieval by conditioning the Mistral-7B-Instruct-v0.2 LLM on the reference image, the original text prompt, and the visual features of user-selected ideal target images, so that rewrites are grounded in what the user actually wants rather than blind trial-and-error.

## When to use

Use when a composed image retrieval query (reference image + text prompt) returns poor results and the user has selected ideal target images from the top-k ranking that should anchor LLM-generated prompt rewrites.

## When NOT to use

- The user has not selected any ideal target images, so there is no visual anchor and the counterfactual generation is ungrounded
- The task is single-image or pure text retrieval rather than composed image retrieval where a reference image and text prompt are fused via SEARLE/CLIP textual inversion
- The goal is model-internal diagnosis (saliency maps, token attribution) rather than prompt refinement — use the Explanation Panel workflow instead

## Inputs

- Reference image I_ref (user-uploaded image file)
- Natural language modification prompt T (e.g. 'make it red', 'add a hat')
- User-selected ideal target image(s) I* clicked from the top-k Query Results Panel
- Configuration flags: ENHANCEMENT_USE_CONTEXT = True, ENHANCEMENT_CANDIDATE_PROMPTS = 10

## Outputs

- 10 candidate enhanced prompt variants T' (textual modifiers of the original prompt)
- Structured JSON file enhanced_prompt_variants containing variants plus grounding inputs (reference image identifier, original prompt, ideal-image identifiers)
- Per-candidate retrieval metrics (NDCG@K, mAP@K, MRR, Coverage, Mean Rank, Mean Similarity) and an Ideal-anchored Rank-Delta heatmap

## How to apply

Accept the composed-query inputs (reference image I_ref and natural language prompt T) from the Composed Image Retrieval Panel, then collect the user-selected ideal image(s) I* from the Query Results Panel to serve as fixed anchors. Enable visual context grounding via ENHANCEMENT_USE_CONTEXT = True so mistralai/Mistral-7B-Instruct-v0.2 is conditioned on the content of I_ref, T, and the visual features of I*, generating a set of alternative prompt variants (10 candidates per ENHANCEMENT_CANDIDATE_PROMPTS = 10). Each candidate can be re-ranked against the ideal anchor via similarity s = sim(f_c(I_ref, T'), f_i(I*)) using SEARLE/CLIP embeddings, and its empirical effect verified through the Ideal-anchored Rank-Delta heatmap. Export the variants with their grounding inputs (reference image identifier, original prompt, ideal-image identifiers) as structured JSON. Rationale: grounding on the ideal images replaces the blind trial-and-error diagnosis the article criticizes, where developers tweak prompts without understanding the failure mechanism.

## Related tools

- **mistralai/Mistral-7B-Instruct-v0.2** (Open-weight LLM conditioned on I_ref, T, and the visual features of the selected ideal images I* to generate alternative prompt variants T') — https://github.com/giannhskp/InfoCIR
- **CLIP** (Provides the joint vision-language embedding space used to compute similarity s = sim(f_c(I_ref, T'), f_i(I*)) between each candidate prompt and the ideal anchor) — https://github.com/giannhskp/InfoCIR
- **SEARLE** (CIR back-end that performs textual inversion of the reference image into a pseudo-token; embeddings it produces are re-ranked with each generated prompt variant) — https://github.com/giannhskp/InfoCIR

## Examples

```
cd cir_app && python run.py  # then select ideal images in the Query Results Panel and click 'Enhance Prompt' (ENHANCEMENT_USE_CONTEXT=True)
```

## Evaluation signals

- enhanced_prompt_variants file exists and is valid JSON
- The JSON contains the configured number of candidate prompt entries (ENHANCEMENT_CANDIDATE_PROMPTS = 10)
- Each candidate entry is a well-formed textual modifier that can be re-ranked against the ideal anchor via s = sim(f_c(I_ref, T'), f_i(I*))
- Each entry carries its grounding inputs: reference image identifier, original prompt, and ideal-image identifiers
- Empirical effect is confirmed through the Ideal-anchored Rank-Delta heatmap loop, with per-candidate NDCG@K, mAP@K, MRR, Coverage, Mean Rank and Mean Similarity scores

## Limitations

- Only ImageNet-R is supported as the dataset with the default config; dataset path must be set via DATASET_ROOT_PATH with a class_names.csv mapping dir_name to class_name
- Scalability, extended model support, and personalization are acknowledged limitations deferred to future work
- Ranking improvements are validated only against user-selected ideal anchors, so quality depends on the appropriateness of the selected ideal images

## Evidence

- [methods] Condition on ideal targets: [evidence span withheld — anchor status: not_found]
- [other] Prompt Enhancement Panel purpose: [evidence span withheld — anchor status: not_found]
- [intro] Motivation against blind tweaking: [evidence span withheld — anchor status: not_found]
