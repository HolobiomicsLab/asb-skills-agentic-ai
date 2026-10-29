---
name: cot-bbox-accuracy-verification
description: Use when a generated VQA-G candidate (object mention + GroundingDINO
  bbox) must be visually checked against its source image using CoT reasoning, yielding
  a weighted consistency score that gates acceptance at τ=0.9.
when_to_use_negative:
- The goal is checking VQA answer/content quality rather than spatial grounding —
  apply the content quality verification stage of the two-stage assessment instead
- Bounding boxes are already human-verified ground truth used only to compute mIoU
  or Acc@0.5IoU — no CoT verification is needed
- No predicted bbox or source image is available (text-only pipeline) — bbox accuracy
  verification cannot be executed
edam_operation: http://edamontology.org/operation_2945
tools:
- name: AutoVQA-G
  role: Host framework; its Consistency Evaluation module implements the two-stage
    CoT assessment containing the bbox accuracy verification stage
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: Qwen2.5-VL-72B-Instruct
  role: CoT verifier that executes bbox accuracy verification and produces weighted
    consistency scores via DashScope (VERIFICATION_API_KEY)
- name: GroundingDINO
  role: Produces the candidate bounding boxes (from the object mention) that this
    verification stage scores
- name: MiniCPM-o 2.6
  role: Local generation server producing the object mention that the verifier checks
    the bbox against
- name: DeepSeek V3
  role: Memory-augmented prompt optimizer consuming bbox critiques from failed samples
    to refine generation prompts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1109/icassp55912.2026.11462754
    title: 'AutoVQA-G: Self-Improving Agentic Framework for Automated Visual Question
      Answering and Grounding Annotation'
  source_content_hash: sha256:0d23f3c2e93f1e33d2d4ba4f0f507a5844cb89336e994050e9f3753debec7e70
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cot-bbox-accuracy-verification

## Summary

The bbox accuracy verification stage of AutoVQA-G's two-stage CoT Consistency Evaluation: the Qwen2.5-VL-72B-Instruct verifier (via DashScope) critiques a predicted GroundingDINO bounding box against the source image and object mention, producing weighted consistency scores that gate acceptance in the generate-evaluate-refine loop.

## When to use

Use when a generated VQA-G candidate (object mention + GroundingDINO bbox) must be visually checked against its source image using CoT reasoning, yielding a weighted consistency score that gates acceptance at τ=0.9.

## When NOT to use

- The goal is checking VQA answer/content quality rather than spatial grounding — apply the content quality verification stage of the two-stage assessment instead
- Bounding boxes are already human-verified ground truth used only to compute mIoU or Acc@0.5IoU — no CoT verification is needed
- No predicted bbox or source image is available (text-only pipeline) — bbox accuracy verification cannot be executed

## Inputs

- Annotations JSON list (each entry with at least an 'image' key) plus the image directory (--image_dir)
- Generated candidate per image: object_description (MiniCPM-o 2.6) and bbox (GroundingDINO)
- VERIFICATION_API_KEY for the Qwen2.5-VL-72B-Instruct CoT verifier via DashScope

## Outputs

- CoT visual critique text and weighted consistency score per candidate
- Selected per-image JSON record with object_description, bbox, consistency_score, iteration, is_successful, CoT_critique fields
- Critiques of failed samples forwarded to the DeepSeek V3 memory-augmented prompt optimizer

## How to apply

Execute the bbox accuracy verification stage of the two-stage CoT assessment (§2.2) inside the generate-evaluate-refine loop: send the candidate (object_description from MiniCPM-o 2.6, bbox from GroundingDINO) together with the source image to the Qwen2.5-VL-72B-Instruct CoT verifier via DashScope, which returns a step-by-step visual critique and a weighted consistency score. Accept the candidate when its consistency score S_t ≥ τ (τ = 0.9, set via --threshold 0.9); on failure, route the CoT critique to the memory-augmented DeepSeek V3 Prompt Optimization agent to refine the generation prompts and regenerate, up to --max_iter 5 iterations. Select the first sample whose consistency score meets τ, or otherwise the highest-scoring iteration, as that image's annotation. Reset prompt-optimization history after each image so one sample's refinement history does not affect the next.

## Related tools

- **AutoVQA-G** (Host framework; its Consistency Evaluation module implements the two-stage CoT assessment containing the bbox accuracy verification stage) — https://github.com/rohnson1999/AutoVQA-G
- **Qwen2.5-VL-72B-Instruct** (CoT verifier that executes bbox accuracy verification and produces weighted consistency scores via DashScope (VERIFICATION_API_KEY))
- **GroundingDINO** (Produces the candidate bounding boxes (from the object mention) that this verification stage scores)
- **MiniCPM-o 2.6** (Local generation server producing the object mention that the verifier checks the bbox against)
- **DeepSeek V3** (Memory-augmented prompt optimizer consuming bbox critiques from failed samples to refine generation prompts)

## Examples

```
python main.py --annotations path/to/annotations. --image_dir path/to/images/ --output_dir output/ --threshold 0.9 --max_iter 5
```

## Evaluation signals

- Loop terminates when the consistency score S_t ≥ τ (τ = 0.9 default via --threshold 0.9) or --max_iter 5 is exhausted
- Selected record is the first sample passing the threshold, or otherwise the highest-scoring iteration for that image
- Output JSON conforms to the README schema: object_description, bbox, consistency_score, iteration, is_successful, CoT_critique
- is_successful is true only when the selected sample meets τ; failed samples carry a CoT_critique routed to the DeepSeek V3 optimizer
- Prompt-optimization history is reset per image — no refinement history leaks between consecutive samples

## Limitations

- Computational overhead of the iterative CoT verification loop is a stated limitation of the framework
- If no iteration reaches τ = 0.9 within --max_iter 5, the highest-scoring (sub-threshold) annotation is still emitted, so accepted data need not be fully verified
- Verification depends on external API availability (Qwen2.5-VL-72B-Instruct via DashScope); key or endpoint failures stall the loop

## Evidence
