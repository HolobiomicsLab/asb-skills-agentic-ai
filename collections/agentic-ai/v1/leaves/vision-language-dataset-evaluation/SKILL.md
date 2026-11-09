---
name: vision-language-dataset-evaluation
description: Use when you have auto-generated (question, answer, object mention, bbox)
  annotations with ground-truth grounding boxes and need to quantify fidelity via
  consistency scores and IoU-based metrics.
when_to_use_negative:
- Input is human-annotated gold-standard VQA-G data that needs no automated fidelity
  verification — the skill targets hallucination-prone single-pass model outputs.
- The task requires no spatial grounding (bbox) component and only text QA fidelity
  is of interest — the two-stage bbox accuracy verification and IoU metrics would
  be inapplicable.
- Evaluation must run fully offline with no LLM access — the CoT verifier depends
  on Qwen2.5-VL-72B-Instruct via DashScope and the optimizer on DeepSeek V3.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: AutoVQA-G
  role: Framework whose generate-evaluate-refine loop and Consistency Evaluation module
    implement the verification and scoring workflow
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: Qwen2.5-VL-72B-Instruct
  role: CoT verifier performing the two-stage content quality + bbox accuracy assessment
    and producing weighted consistency scores
- name: MiniCPM-o 2.6
  role: Local generation server for caption, VQA, and object mention generation fed
    into the evaluation loop
- name: GroundingDINO
  role: Produces predicted bounding boxes from object mentions for IoU-based evaluation
    against ground truth
- name: DeepSeek V3
  role: Memory-augmented prompt optimizer that refines generation prompts from critiques
    of failed (low-consistency) samples
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1109/icassp55912.2026.11462754
    title: 'AutoVQA-G: Self-Improving Agentic Framework for Automated Visual Question
      Answering and Grounding Annotation'
  source_content_hash: sha256:2c4d1a550d77aac80b51e700b6d4a7f62037e1ce4dcf7b109e036b5e17f2bd33
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# vision-language-dataset-evaluation

## Summary

Evaluates auto-generated VQA-G annotation quality using a CoT-based Consistency Evaluation module (two-stage content quality + bbox accuracy verification producing weighted consistency scores) and grounding metrics (mIoU, Acc@0.5IoU). On Visual7W, the full AutoVQA-G pipeline reference baseline is mIoU 0.634 and Acc@0.5IoU 0.720.

## When to use

Use when you have auto-generated (question, answer, object mention, bbox) annotations with ground-truth grounding boxes and need to quantify fidelity via consistency scores and IoU-based metrics.

## When NOT to use

- Input is human-annotated gold-standard VQA-G data that needs no automated fidelity verification — the skill targets hallucination-prone single-pass model outputs.
- The task requires no spatial grounding (bbox) component and only text QA fidelity is of interest — the two-stage bbox accuracy verification and IoU metrics would be inapplicable.
- Evaluation must run fully offline with no LLM access — the CoT verifier depends on Qwen2.5-VL-72B-Instruct via DashScope and the optimizer on DeepSeek V3.

## Inputs

- JSON annotation file (list of dicts with at least an 'image' key) pointing to images in an image directory
- Ground-truth Visual7W/VizWiz grounding annotations (bounding boxes) for IoU-based evaluation
- Predicted (object_description, bbox, consistency_score) records from the AutoVQA-G output JSON schema

## Outputs

- Per-image annotation records with object_description, bbox, consistency_score, iteration, is_successful, CoT_critique
- mIoU and Acc@0.5IoU scores of predicted bounding boxes vs ground-truth grounding annotations (written to grounding_only_results)

## How to apply

Run the generate-evaluate-refine loop with --threshold 0.9 and --max_iter 5, applying the two-stage CoT assessment: content quality verification plus bbox accuracy verification via Qwen2.5-VL-72B-Instruct (DashScope), yielding weighted consistency scores. Accept the first sample with consistency score S_t >= 0.9, otherwise keep the highest-scoring iteration per image; prompt optimization history is reset per image. For the grounding-only condition (object mention via MiniCPM-o 2.6 + GroundingDINO bbox, bypassing VQA generation), compute mIoU and Acc@0.5IoU of predicted boxes against Visual7W/VizWiz ground-truth grounding annotations. Judge success against the full-pipeline reference baseline on Visual7W: mIoU 0.634, Acc@0.5IoU 0.720 (VQAScore 0.896, overall VQA-G score 0.747).

## Related tools

- **AutoVQA-G** (Framework whose generate-evaluate-refine loop and Consistency Evaluation module implement the verification and scoring workflow) — https://github.com/rohnson1999/AutoVQA-G
- **Qwen2.5-VL-72B-Instruct** (CoT verifier performing the two-stage content quality + bbox accuracy assessment and producing weighted consistency scores)
- **MiniCPM-o 2.6** (Local generation server for caption, VQA, and object mention generation fed into the evaluation loop)
- **GroundingDINO** (Produces predicted bounding boxes from object mentions for IoU-based evaluation against ground truth)
- **DeepSeek V3** (Memory-augmented prompt optimizer that refines generation prompts from critiques of failed (low-consistency) samples)

## Examples

```
python main.py --annotations path/to/annotations.json --image_dir path/to/images/ --output_dir output/ --threshold 0.9 --max_iter 5
```

## Evaluation signals

- Selected sample per image has consistency_score >= 0.9 (threshold tau), or is the highest-scoring iteration when max_iter=5 is exhausted
- mIoU of predicted boxes approximates the reference baseline of 0.634 on Visual7W
- Acc@0.5IoU approximates the reference baseline of 0.720 on Visual7W
- Output JSON contains the required schema fields: object_description, bbox, consistency_score, iteration, is_successful, CoT_critique
- is_successful=true only for samples whose loop terminated via S_t >= tau rather than the iteration cap

## Limitations

- Current computational overhead of the iterative generate-evaluate-refine loop is a stated limitation of the framework
- When no iteration reaches the 0.9 threshold within max_iter=5, only the highest-scoring (possibly below-threshold) sample is retained
- Prompt optimization history is reset per image, so refinements do not transfer across images
- No changelog is available for the repository

## Evidence

- [other] baseline metrics: [evidence span withheld — anchor status: not_found]
