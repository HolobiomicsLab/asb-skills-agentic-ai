---
name: cold-start-ideal-image-anchoring
description: Use when a composed image+text CIR query yields unsatisfying top-k results
  and the user has marked ideal target image(s) among them, so alternative prompt
  variants must be generated and re-ranked against those anchors.
when_to_use_negative:
- No ideal target images have been selected from the top-k results, so there are no
  anchors to condition generation or verify ranking impact
- The task is standalone prompt rewriting or text-only retrieval without a composed
  image+text CIR ranking loop to evaluate variants against
- The goal is explaining why the model retrieved certain images (use saliency maps
  / token attribution instead) rather than modifying the prompt
tools:
- name: mistralai/Mistral-7B-Instruct-v0.2
  role: Open-weight LLM conditioned on I_ref, T, and the visual features of the selected
    ideal images I* to generate the 10 counterfactual prompt variants
- name: CLIP
  role: Joint vision-language embedding space used to compute the similarity s = sim(f_c(I_ref,
    T'), f_i(I*)) for re-ranking candidates against ideal anchors
- name: SEARLE
  role: CIR back-end that produces the initial top-k ranking from which ideal target
    images are selected and against which enhanced prompts are re-evaluated
  repo: https://github.com/giannhskp/InfoCIR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1109/pacificvis68791.2026.00041
    title: 'InfoCIR: Multimedia Analysis for Composed Image Retrieval'
  source_content_hash: sha256:2d1e3f5df1d17c7aa8f5884878d0c310640e933e51d3baa0219e770768395785
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giannhskp/InfoCIR
  tools:
  - CLIP
  - SEARLE
  - mistralai/Mistral-7B-Instruct-v0.2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cold-start-ideal-image-anchoring

## Summary

Generates counterfactual prompt variants for a composed image retrieval (CIR) query by conditioning the Mistral-7B-Instruct LLM on the reference image, the original text prompt, and user-selected ideal target images, so ranking of the selected targets can be improved and verified.

## When to use

Use when a composed image+text CIR query yields unsatisfying top-k results and the user has marked ideal target image(s) among them, so alternative prompt variants must be generated and re-ranked against those anchors.

## When NOT to use

- No ideal target images have been selected from the top-k results, so there are no anchors to condition generation or verify ranking impact
- The task is standalone prompt rewriting or text-only retrieval without a composed image+text CIR ranking loop to evaluate variants against
- The goal is explaining why the model retrieved certain images (use saliency maps / token attribution instead) rather than modifying the prompt

## Inputs

- Reference image I_ref uploaded to the Composed Image Retrieval Panel
- Natural language text prompt T describing the desired modification (e.g. 'wearing sunglasses')
- User-selected ideal target image(s) I* clicked from the top-k Query Results Panel
- Top-k retrieved image ranking produced by the SEARLE CIR back-end over the ImageNet-R dataset

## Outputs

- Structured JSON file enhanced_prompt_variants containing 10 candidate prompt entries with grounding inputs (reference image id, original prompt, ideal-image ids)
- Per-candidate retrieval metrics (NDCG@K, mAP@K, MRR, Coverage, Mean Rank, Mean Similarity)
- Ideal-anchored Rank-Delta heatmap showing how each prompt variant changes the ranking of the selected target images

## How to apply

Accept the composed-query inputs (reference image I_ref and natural language prompt T) from the Composed Image Retrieval Panel, then collect the user-selected ideal image(s) I* from the initial top-k results as fixed anchors, with visual context grounding enabled (ENHANCEMENT_USE_CONTEXT = True). Condition mistralai/Mistral-7B-Instruct-v0.2 on the content of I_ref, T, and the visual features of I* to generate 10 candidate prompt variants (repository default ENHANCEMENT_CANDIDATE_PROMPTS = 10). Export the variants together with their grounding inputs (reference image identifier, original prompt, ideal-image identifiers) as a structured JSON file named enhanced_prompt_variants. Rank each variant T' against the ideal anchor via the similarity score s = sim(f_c(I_ref, T'), f_i(I*)) and check empirical effect through the Ideal-anchored Rank-Delta heatmap loop and retrieval metrics (NDCG@K, mAP@K, MRR, Coverage, Mean Rank, Mean Similarity).

## Related tools

- **mistralai/Mistral-7B-Instruct-v0.2** (Open-weight LLM conditioned on I_ref, T, and the visual features of the selected ideal images I* to generate the 10 counterfactual prompt variants)
- **CLIP** (Joint vision-language embedding space used to compute the similarity s = sim(f_c(I_ref, T'), f_i(I*)) for re-ranking candidates against ideal anchors)
- **SEARLE** (CIR back-end that produces the initial top-k ranking from which ideal target images are selected and against which enhanced prompts are re-evaluated) — https://github.com/giannhskp/InfoCIR

## Examples

```
cd cir_app && python run.py  # with src/config.py: ENHANCEMENT_CANDIDATE_PROMPTS = 10; ENHANCEMENT_USE_CONTEXT = True
```

## Evaluation signals

- enhanced_prompt_variants file exists, is valid JSON, and contains exactly ENHANCEMENT_CANDIDATE_PROMPTS (default 10) well-formed candidate prompt entries
- Each candidate entry carries complete grounding metadata: reference image identifier, original prompt, and ideal-image identifiers
- Every variant yields a finite similarity score s = sim(f_c(I_ref, T'), f_i(I*)) against each ideal anchor
- Ideal-anchored Rank-Delta heatmap shows measurable rank changes of the selected target images across prompt variants
- Retrieval metrics (NDCG@K, mAP@K, MRR) are computed for each candidate and at least one variant improves over the original prompt

## Limitations

- Generation quality depends on the user manually selecting ideal target images; without good anchors the counterfactual variants are ungrounded
- Scalability and broader model support are acknowledged as future work, so the enhancement loop may not extend to very large datasets or non-CLIP back-ends
- No changelog is available in the associated repository, so version drift of the enhancement configuration cannot be tracked

## Evidence

- [intro] LLM prompt variants coupled with Ideal-anchored Rank-Delta heatmap verification: [evidence span withheld — anchor status: not_found]
- [methods] Mistral-7B-Instruct-v0.2 is the LLM used by the system: [evidence span withheld — anchor status: not_found]
