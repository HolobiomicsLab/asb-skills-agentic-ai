---
name: grounding-only-pipeline-configuration
description: Use when you need standalone visual-grounding annotations (object mention
  + bbox) from a generate-evaluate-refine framework and want to benchmark box quality
  (mIoU, Acc@0.5IoU) without the VQA branch.
when_to_use_negative:
- You need full VQA-G annotation (question + answer + grounding) — bypassing VQA Generation
  removes the question/answer branch
- You have no ground-truth bounding boxes to compute mIoU or Acc@0.5IoU, so grounding
  quality cannot be judged
- Your objects are out-of-distribution for GroundingDINO text queries or you lack
  a reachable GroundingDINO HTTP endpoint
edam_operation: http://edamontology.org/operation_3774
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: AutoVQA-G
  role: Self-improving agentic pipeline reconfigured to bypass VQA Generation and
    keep only visual grounding
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: MiniCPM-o 2.6
  role: Local generation server (http://localhost:8000/v1) for caption reasoning and
    object mention generation
- name: GroundingDINO
  role: Predicts bounding boxes from object mentions via HTTP endpoint (POST image,
    query, box_threshold, text_threshold)
- name: Qwen2.5-VL-72B-Instruct
  role: CoT verifier via DashScope; applies the bbox accuracy verification stage producing
    weighted consistency scores
- name: DeepSeek V3
  role: Memory-augmented prompt optimizer refining mention-generation prompts from
    critiques of failed samples
- name: Python
  role: Runtime for main.py/pipeline.py driving the generate-evaluate-refine loop
  repo: https://github.com/rohnson1999/AutoVQA-G
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1109/icassp55912.2026.11462754
    title: 'AutoVQA-G: Self-Improving Agentic Framework for Automated Visual Question
      Answering and Grounding Annotation'
  source_content_hash: sha256:1a9dbc524719a71b44a0f0e088727a62366de3ebf145fd75233744b8581b5986
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
  edam_operation: http://edamontology.org/operation_3774
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# grounding-only-pipeline-configuration

## Summary

Reconfigure the AutoVQA-G modular generation stage to bypass VQA Generation, running only Caption Reasoning → object mention generation (MiniCPM-o 2.6) → GroundingDINO bounding box prediction, so that grounding annotation quality can be isolated and measured with mIoU and Acc@0.5IoU against ground-truth boxes (e.g. Visual7W/VizWiz).

## When to use

Use when you need standalone visual-grounding annotations (object mention + bbox) from a generate-evaluate-refine framework and want to benchmark box quality (mIoU, Acc@0.5IoU) without the VQA branch.

## When NOT to use

- You need full VQA-G annotation (question + answer + grounding) — bypassing VQA Generation removes the question/answer branch
- You have no ground-truth bounding boxes to compute mIoU or Acc@0.5IoU, so grounding quality cannot be judged
- Your objects are out-of-distribution for GroundingDINO text queries or you lack a reachable GroundingDINO HTTP endpoint

## Inputs

- JSON annotations file: list of dicts each with an "image" key (e.g. [{"image": "image_001.jpg"}])
- Image directory (Visual7W or VizWiz images)
- Ground-truth grounding bounding-box annotations for mIoU / Acc@0.5IoU evaluation
- API keys: GENERATION_API_KEY, VERIFICATION_API_KEY, OPTIMIZER_API_KEY; GROUNDING_DINO_URL endpoint

## Outputs

- Per-image grounding annotation JSON with object_description, bbox, consistency_score fields
- mIoU of predicted bounding boxes vs ground truth
- Acc@0.5IoU of predicted bounding boxes vs ground truth
- Per-image iteration logs (CoT_critique, is_successful, timestamp)

## How to apply

Clone github.com/rohnson1999/AutoVQA-G and reconfigure the modular generation stage (§2.1) to bypass VQA Generation, keeping Caption Reasoning → object mention generation with MiniCPM-o 2.6 (local server at http://localhost:8000/v1) followed by GroundingDINO bbox prediction (POST to GROUNDING_DINO_URL with image, query, box_threshold, text_threshold). Run the generate-evaluate-refine loop with consistency threshold τ=0.9 (--threshold 0.9) and at most 5 refinement iterations per image (--max_iter 5), applying only the bbox accuracy verification stage of the two-stage CoT assessment via Qwen2.5-VL-72B-Instruct (DashScope). Feed failed-sample critiques to DeepSeek V3, the memory-augmented prompt optimizer, to refine mention-generation prompts, resetting optimization history per image. For each image, select the first sample with consistency score ≥ 0.9, or otherwise the highest-scoring iteration, as the (object_description, bbox) output pair. Finally compute mIoU and Acc@0.5IoU of predicted boxes against ground-truth Visual7W/VizWiz grounding annotations; the full pipeline's Visual7W reference is mIoU 0.634 and Acc@0.5IoU 0.720.

## Related tools

- **AutoVQA-G** (Self-improving agentic pipeline reconfigured to bypass VQA Generation and keep only visual grounding) — https://github.com/rohnson1999/AutoVQA-G
- **MiniCPM-o 2.6** (Local generation server (http://localhost:8000/v1) for caption reasoning and object mention generation)
- **GroundingDINO** (Predicts bounding boxes from object mentions via HTTP endpoint (POST image, query, box_threshold, text_threshold))
- **Qwen2.5-VL-72B-Instruct** (CoT verifier via DashScope; applies the bbox accuracy verification stage producing weighted consistency scores)
- **DeepSeek V3** (Memory-augmented prompt optimizer refining mention-generation prompts from critiques of failed samples)
- **Python** (Runtime for main.py/pipeline.py driving the generate-evaluate-refine loop) — https://github.com/rohnson1999/AutoVQA-G

## Examples

```
python main.py --annotations path/to/annotations. --image_dir path/to/images/ --output_dir output/ --threshold 0.9 --max_iter 5
```

## Evaluation signals

- mIoU of predicted boxes against Visual7W ground truth approaches or is contextualized against the full-pipeline reference of 0.634
- Acc@0.5IoU against ground-truth boxes contextualized against the full-pipeline reference of 0.720
- Every output image has consistency_score ≥ 0.9, or otherwise carries the highest-scoring iteration within --max_iter 5
- Output JSON records contain the required keys: object_description, bbox, consistency_score (plus iteration, is_successful, CoT_critique, timestamp)
- Prompt optimization history is reset per image — no refinement history leaks across samples

## Limitations

- Computational overhead of the iterative loop is a stated limitation of the framework
- Grounding depends on GroundingDINO HTTP endpoint availability and its box_threshold/text_threshold settings; single-pass hallucinations are mitigated only up to the τ=0.9 threshold and 5-iteration cap
- Consistency scores come from LLM-based CoT verification (Qwen2.5-VL-72B-Instruct), which may fail in complex or out-of-distribution scenarios
- The repository provides no changelog, so version drift in the codebase is hard to track

## Evidence

- [results] Full-pipeline grounding reference: [evidence span withheld — anchor status: not_found]
