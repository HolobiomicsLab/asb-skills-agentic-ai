---
name: visual-grounding-bbox-detection
description: Use when you need to produce (object mention, bounding box) grounding
  annotations for a directory of images and want verifiable bbox accuracy via CoT
  verification, or need a full-pipeline grounding baseline (mIoU, Acc@0.5IoU) on Visual7W/VizWiz-style
  data.
when_to_use_negative:
- When the input is already a curated VQA-G dataset with human-verified bounding boxes
  — the loop's CoT verification and refinement add compute without benefit (the paper
  notes computational overhead as a limitation).
- When the task is pure VQA text generation without any grounding requirement — the
  Visual Grounding stage (object mention + GroundingDINO BBox) is unnecessary.
- When no GroundingDINO endpoint or equivalent open-vocabulary detector is available
  and the images fall outside the detector's grounding capability — verification will
  repeatedly fail and burn the full 5 iterations.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3474
tools:
- name: GroundingDINO
  role: Open-vocabulary visual grounding endpoint that maps the object mention (query)
    to a bounding box; accepts POST with image, query, box_threshold, text_threshold
    and returns a detections list with bbox and confidence
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: MiniCPM-o 2.6
  role: Local generation server used for Caption Reasoning and object mention generation
    that supplies the query string for GroundingDINO
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: Qwen2.5-VL-72B-Instruct
  role: CoT verifier performing the bbox accuracy verification stage of the two-stage
    consistency assessment, yielding the weighted consistency score S_t
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: DeepSeek V3
  role: Memory-augmented prompt optimizer that refines generation prompts from CoT
    critiques of failed grounding samples (history reset per image)
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: Python
  role: Driver for main.py/pipeline.py, computing mIoU and Acc@0.5IoU against ground-truth
    boxes and writing the output JSON
  repo: https://github.com/rohnson1999/AutoVQA-G
provenance:
  source_task_ids:
  - task_003
  - task_002
  - task_007
  - task_008
  source_papers:
  - doi: 10.1109/icassp55912.2026.11462754
    title: 'AutoVQA-G: Self-Improving Agentic Framework for Automated Visual Question
      Answering and Grounding Annotation'
  source_content_hash: sha256:5013f8e87c1d5650c79169628970c25b03a23044205d9008ea2bec50402b2d98
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
  - Python
  - Qwen2.5-VL-72B-Instruct
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# visual-grounding-bbox-detection

## Summary

Generates visual grounding annotations — an object mention paired with a predicted bounding box — by prompting MiniCPM-o 2.6 for an object mention and querying a GroundingDINO HTTP endpoint for the bbox, optionally inside the AutoVQA-G generate-evaluate-refine loop with CoT bbox-accuracy verification. This addresses the paper's goal of producing high-fidelity, automatically generated grounding data for VQA-G datasets.

## When to use

Use when you need to produce (object mention, bounding box) grounding annotations for a directory of images and want verifiable bbox accuracy via CoT verification, or need a full-pipeline grounding baseline (mIoU, Acc@0.5IoU) on Visual7W/VizWiz-style data.

## When NOT to use

- When the input is already a curated VQA-G dataset with human-verified bounding boxes — the loop's CoT verification and refinement add compute without benefit (the paper notes computational overhead as a limitation).
- When the task is pure VQA text generation without any grounding requirement — the Visual Grounding stage (object mention + GroundingDINO BBox) is unnecessary.
- When no GroundingDINO endpoint or equivalent open-vocabulary detector is available and the images fall outside the detector's grounding capability — verification will repeatedly fail and burn the full 5 iterations.

## Inputs

- JSON annotation file: list of dicts, each with at least an 'image' key (e.g. [{"image": "image_001.jpg"}])
- Directory of input images (--image_dir)
- Ground-truth grounding bounding boxes (for computing mIoU / Acc@0.5IoU in the VG-only evaluation)
- Running endpoints: MiniCPM-o 2.6 generation server, GroundingDINO HTTP endpoint, Qwen2.5-VL-72B-Instruct verifier, DeepSeek V3 optimizer

## Outputs

- Per-image JSON record with image_identifier, object_description, bbox ([x1, y1, x2, y2]-style list e.g. [120, 85, 340, 260]), consistency_score, iteration, is_successful, CoT_critique, timestamp (plus question/answer in full VQA-G mode)
- Aggregate grounding metrics: mIoU and Acc@0.5IoU against ground-truth boxes (full-pipeline reference: mIoU 0.634, Acc@0.5IoU 0.720 on Visual7W)

## How to apply

Prepare a JSON annotation list where each entry has an 'image' key, then run the pipeline with python main.py --annotations <file> --image_dir <dir> --output_dir output/ --threshold 0.9 --max_iter 5. The modular generation stage performs Caption Reasoning → VQA Generation → Visual Grounding, where the object mention is generated by MiniCPM-o 2.6 (localhost:8000/v1) and the bbox comes from a GroundingDINO endpoint (http://localhost:8001/predict) that accepts a POST with image, query, box_threshold, and text_threshold. Score each draft with the two-stage CoT verification (Qwen2.5-VL-72B-Instruct via DashScope), using the bbox accuracy verification stage to weight the consistency score S_t against τ=0.9; on failure, feed critiques to the DeepSeek V3 memory-augmented prompt optimizer (reset per image). Accept the first iteration with S_t ≥ 0.9, or after max_iter 5 keep the highest-scoring iteration. For a standalone VG-only condition, bypass VQA Generation and keep only Caption Reasoning → object mention → GroundingDINO, then compute mIoU and Acc@0.5IoU against ground-truth grounding annotations.

## Related tools

- **GroundingDINO** (Open-vocabulary visual grounding endpoint that maps the object mention (query) to a bounding box; accepts POST with image, query, box_threshold, text_threshold and returns a detections list with bbox and confidence) — https://github.com/rohnson1999/AutoVQA-G
- **MiniCPM-o 2.6** (Local generation server used for Caption Reasoning and object mention generation that supplies the query string for GroundingDINO) — https://github.com/rohnson1999/AutoVQA-G
- **Qwen2.5-VL-72B-Instruct** (CoT verifier performing the bbox accuracy verification stage of the two-stage consistency assessment, yielding the weighted consistency score S_t) — https://github.com/rohnson1999/AutoVQA-G
- **DeepSeek V3** (Memory-augmented prompt optimizer that refines generation prompts from CoT critiques of failed grounding samples (history reset per image)) — https://github.com/rohnson1999/AutoVQA-G
- **Python** (Driver for main.py/pipeline.py, computing mIoU and Acc@0.5IoU against ground-truth boxes and writing the output JSON) — https://github.com/rohnson1999/AutoVQA-G

## Examples

```
python main.py --annotations path/to/annotations. --image_dir path/to/images/ --output_dir output/ --threshold 0.9 --max_iter 5
```

## Evaluation signals

- Full-pipeline Visual7W reference: mIoU 0.634 and Acc@0.5IoU 0.720 (VQAScore 0.896, overall VQA-G score 0.747); results should fall within reasonable tolerance of these figures
- Every output record contains the required fields: object_description, bbox, consistency_score, iteration, is_successful (plus image_identifier, CoT_critique, timestamp)
- Loop termination invariant: each image has exactly one selected result — the first sample with consistency score ≥ 0.9 (τ) or, after 5 iterations, the highest-scoring iteration
- bbox field is a valid 4-element coordinate list and the GroundingDINO response includes bbox and confidence within a detections list
- Ablation sanity: removing the refinement loop (single-pass) should drop mIoU toward the reference 0.380, confirming the iterative refinement is responsible for grounding gains

## Limitations

- Current computational overhead of the iterative loop is a stated limitation of the framework
- Prompt optimization memory is reset per image, so refinement patterns learned on one image do not transfer to the next
- Bounding-box quality depends on the external GroundingDINO endpoint (POST with image, query, box_threshold, text_threshold); a brittle detector propagates failures through the loop

## Evidence

- [methods] Full-pipeline grounding reference metrics on Visual7W: [evidence span withheld — anchor status: not_found]
