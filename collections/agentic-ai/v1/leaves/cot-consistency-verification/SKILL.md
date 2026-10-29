---
name: cot-consistency-verification
description: Use when automatically generated VQA-G annotations (question, answer,
  object mention, bbox) must be verified for visual consistency against the source
  image, replacing brittle heuristic checks with interpretable CoT critiques and a
  weighted score.
when_to_use_negative:
- When annotations were produced by human experts and already validated — the framework
  targets unscalable manual VQA-G annotation, not re-verification of trusted labels
- When only simple filtering of pre-scored annotations is needed without visual grounding
  checks — the module requires source images and bbox verification against them
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: Qwen2.5-VL-72B-Instruct
  role: CoT verifier that performs the two-stage content-quality and bbox-accuracy
    consistency evaluation
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: GroundingDINO
  role: Provides the bbox used in the bbox-accuracy verification stage of the consistency
    assessment
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: MiniCPM-o 2.6
  role: Local generation server producing the caption, VQA, and object mention drafts
    that the verifier evaluates
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: DeepSeek V3
  role: Prompt optimizer that consumes failed CoT critiques to refine generation prompts
    on the next iteration
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: AutoVQA-G
  role: Pipeline orchestrating the generate-evaluate-refine loop in which this verification
    step runs
  repo: https://github.com/rohnson1999/AutoVQA-G
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1109/icassp55912.2026.11462754
    title: 'AutoVQA-G: Self-Improving Agentic Framework for Automated Visual Question
      Answering and Grounding Annotation'
  source_content_hash: sha256:cab8c7468a62ced62dde5ee926b21ce2cdc3c572deec65fd3c111cbaaaa2b6f5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rohnson1999/AutoVQA-G
  tools:
  - AutoVQA-G
  - DeepSeek
  - DeepSeek V3
  - GroundingDINO
  - MiniCPM-o
  - MiniCPM-o 2.6
  - Qwen2.5-VL-72B-Instruct
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# cot-consistency-verification

## Summary

Fine-grained verification of generated VQA-G annotations using Chain-of-Thought (CoT) reasoning, producing a weighted consistency score S_t from a two-stage content-quality plus bbox-accuracy assessment, used as the acceptance/refinement signal in the AutoVQA-G loop.

## When to use

Use when automatically generated VQA-G annotations (question, answer, object mention, bbox) must be verified for visual consistency against the source image, replacing brittle heuristic checks with interpretable CoT critiques and a weighted score.

## When NOT to use

- When annotations were produced by human experts and already validated — the framework targets unscalable manual VQA-G annotation, not re-verification of trusted labels
- When only simple filtering of pre-scored annotations is needed without visual grounding checks — the module requires source images and bbox verification against them

## Inputs

- Draft VQA-G annotation per image: question, answer, object_description (object mention), bbox from GroundingDINO
- Source image from the input image directory (--image_dir)
- Consistency score threshold τ (default 0.9) and max iterations (default 5)
- DashScope API key (VERIFICATION_API_KEY) for Qwen2.5-VL-72B-Instruct

## Outputs

- Weighted consistency score S_t for each iteration
- Detailed Chain-of-Thought (CoT) visual critique per iteration
- Accept/refine decision: accept when S_t ≥ τ, otherwise route critique to the Prompt Optimization agent
- Selected result per image (first passing or highest-scoring iteration) with fields including consistency_score, iteration, is_successful, and CoT_critique

## How to apply

For each draft annotation produced by the Modular Generation stage (Caption Reasoning → VQA Generation via MiniCPM-o 2.6 → Visual Grounding via GroundingDINO), run the CoT Consistency Evaluation module using Qwen2.5-VL-72B-Instruct (accessed via DashScope with the VERIFICATION_API_KEY). The module performs a two-stage assessment: first a content-quality verification of the question–answer pair, then a bbox-accuracy verification of the grounded region, yielding detailed step-by-step visual critiques and a weighted consistency score S_t (weights w_vqa=0.7, w_vg=0.3 per the source card). Compare S_t against the threshold τ (default 0.9, set via --threshold): if S_t < τ, the CoT critique is forwarded to the memory-augmented Prompt Optimization agent (DeepSeek V3) to refine generation rubrics, with optimization memory reset per image. The loop terminates when S_t ≥ τ or after max_iter (default 5) iterations, keeping the first sample that passes the threshold or otherwise the highest-scoring iteration.

## Related tools

- **Qwen2.5-VL-72B-Instruct** (CoT verifier that performs the two-stage content-quality and bbox-accuracy consistency evaluation) — https://github.com/rohnson1999/AutoVQA-G
- **GroundingDINO** (Provides the bbox used in the bbox-accuracy verification stage of the consistency assessment) — https://github.com/rohnson1999/AutoVQA-G
- **MiniCPM-o 2.6** (Local generation server producing the caption, VQA, and object mention drafts that the verifier evaluates) — https://github.com/rohnson1999/AutoVQA-G
- **DeepSeek V3** (Prompt optimizer that consumes failed CoT critiques to refine generation prompts on the next iteration) — https://github.com/rohnson1999/AutoVQA-G
- **AutoVQA-G** (Pipeline orchestrating the generate-evaluate-refine loop in which this verification step runs) — https://github.com/rohnson1999/AutoVQA-G

## Examples

```
python main.py --annotations annotations. --image_dir images/ --output_dir output/ --threshold 0.9 --max_iter 5
```

## Evaluation signals

- Consistency score S_t for accepted samples is ≥ τ (default 0.9)
- Output records include is_successful=true for samples that passed the threshold and a non-empty CoT_critique field
- Output JSON schema fields present: image_identifier, question, answer, object_description, bbox, consistency_score, iteration, is_successful, CoT_critique, timestamp
- Each image contributes exactly one selected result: the first sample passing τ, or otherwise the highest-scoring iteration
- The iteration field reflects the accepted attempt and never exceeds --max_iter (default 5)

## Limitations

- Current computational overhead is a stated limitation of the framework
- Prompt optimization memory is reset per image, so refinement history does not transfer between samples
- If no iteration reaches τ within max_iter, the highest-scoring (possibly sub-threshold) iteration is kept, so output scores can fall below τ

## Evidence
