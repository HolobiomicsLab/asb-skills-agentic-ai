---
name: bounding-box-accuracy-assessment
description: Use when verifying candidate VQA-G annotation drafts (question, answer,
  object mention, GroundingDINO bbox) where the grounding box must be checked for
  visual accuracy before acceptance.
when_to_use_negative:
- Verifying pure text QA pairs with no visual grounding bbox — the E_vg stage specifically
  assesses bbox accuracy against the image, so drafts without an object mention and
  GroundingDINO box cannot be scored by it.
- Generating VQA-G annotations from scratch rather than verifying candidate drafts
  — use the modular generation pipeline (Caption Reasoning → VQA Generation → Visual
  Grounding) instead of the verification stage.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Qwen2.5-VL-72B-Instruct
  role: CoT verifier called via DashScope for the E_vg stage, producing the bbox-accuracy
    critique and score
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: GroundingDINO
  role: Produces the proposed object bbox that E_vg verifies for accuracy against
    the source image
- name: AutoVQA-G
  role: Framework hosting the Consistency Evaluation module, weighted score aggregation
    (Eq. 7), and threshold acceptance (Eq. 9)
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: Python
  role: Runtime for main.py/pipeline.py/modules.py implementing the verification loop
    and output records
  repo: https://github.com/rohnson1999/AutoVQA-G
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1109/icassp55912.2026.11462754
    title: 'AutoVQA-G: Self-Improving Agentic Framework for Automated Visual Question
      Answering and Grounding Annotation'
  source_content_hash: sha256:0f934e451f24bcff913442e41808806ebc62184753bff7555b3c85c98c1268ad
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rohnson1999/AutoVQA-G
  tools:
  - AutoVQA-G
  - GroundingDINO
  - Python
  - Qwen2.5-VL-72B-Instruct
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# bounding-box-accuracy-assessment

## Summary

Stage E_vg of AutoVQA-G's Consistency Evaluation module: a CoT-prompted Qwen2.5-VL-72B-Instruct verifier assesses the accuracy of a GroundingDINO-proposed bounding box against the source image, yielding a bbox-accuracy score S_vg that is weight-combined with the content-quality score S_vqa.

## When to use

Use when verifying candidate VQA-G annotation drafts (question, answer, object mention, GroundingDINO bbox) where the grounding box must be checked for visual accuracy before acceptance.

## When NOT to use

- Verifying pure text QA pairs with no visual grounding bbox — the E_vg stage specifically assesses bbox accuracy against the image, so drafts without an object mention and GroundingDINO box cannot be scored by it.
- Generating VQA-G annotations from scratch rather than verifying candidate drafts — use the modular generation pipeline (Caption Reasoning → VQA Generation → Visual Grounding) instead of the verification stage.

## Inputs

- Draft annotation D_t (question, answer, object mention, GroundingDINO bbox) produced by the upstream generation pipeline
- Source image file (e.g., image_001.jpg from the image directory)
- GroundingDINO detection output (bbox and confidence from a POST with image, query, box_threshold, text_threshold)
- Acceptance threshold tau (default 0.9 via --threshold)

## Outputs

- Bbox-accuracy score S_vg and step-wise CoT critique from the E_vg verifier
- Weighted consistency score S_t = w_vqa*S_vqa + w_vg*S_vg (Eq. 7)
- Per-image verification JSON record with fields: image_identifier, question, answer, object_description, bbox, consistency_score, iteration, is_successful, CoT_critique, timestamp

## How to apply

Run the two-stage assessment on the draft annotation D_t: first call Qwen2.5-VL-72B-Instruct via DashScope with CoT prompting (stage E_vqa) for a content-quality score S_vqa, then call it again (stage E_vg) to produce a step-wise critique and a bbox-accuracy score S_vg for the proposed grounding box. Aggregate as S_t = w_vqa*S_vqa + w_vg*S_vg (Eq. 7) and concatenate the two critiques into C_t (Eq. 8). Accept the draft if S_t >= tau (Eq. 9, default tau = 0.9 set via --threshold); on failure, emit the concatenated CoT_critique so the memory-augmented Prompt Optimization agent (DeepSeek V3) can refine the generation prompt in the next iteration. Write a per-image verification record and select the first passing sample or the highest-scoring iteration.

## Related tools

- **Qwen2.5-VL-72B-Instruct** (CoT verifier called via DashScope for the E_vg stage, producing the bbox-accuracy critique and score) — https://github.com/rohnson1999/AutoVQA-G
- **GroundingDINO** (Produces the proposed object bbox that E_vg verifies for accuracy against the source image)
- **AutoVQA-G** (Framework hosting the Consistency Evaluation module, weighted score aggregation (Eq. 7), and threshold acceptance (Eq. 9)) — https://github.com/rohnson1999/AutoVQA-G
- **Python** (Runtime for main.py/pipeline.py/modules.py implementing the verification loop and output records) — https://github.com/rohnson1999/AutoVQA-G

## Examples

```
python main.py --annotations path/to/annotations. --image_dir path/to/images/ --output_dir output/ --threshold 0.9 --max_iter 5
```

## Evaluation signals

- is_successful is true only when the weighted consistency score S_t >= tau (default 0.9); check consistency_score against the threshold in the output JSON.
- The output record must contain all schema fields (question, answer, object_description, bbox, consistency_score, iteration, is_successful, CoT_critique, timestamp).
- Per-image selection invariant: the first sample passing the threshold is chosen, or otherwise the highest-scoring iteration for that image.
- The E_vg critique is step-wise CoT reasoning and is concatenated with the E_vqa critique into the single CoT_critique string C_t (Eq. 8).
- The loop terminates when S_t >= tau or --max_iter (default 5) refinement iterations are reached; iteration count in records must not exceed max_iter.

## Limitations

- Computational overhead of the iterative CoT verification loop is a stated limitation.
- Verification quality depends on the Qwen2.5-VL-72B-Instruct endpoint configured via VERIFICATION_API_KEY; models/endpoints must be changed manually in modules.py.
- Prompt optimization state is reset per image, so refinement history does not carry over between samples.
- No changelog available for the repository.

## Evidence

- [methods] Two-stage CoT verification: "Two-stage assessment — content quality verification + bbox accuracy verification — producing weighted consistency scores"
- [methods] CoT verifier model: "`VERIFICATION_API_KEY` | CoT verifier | `Qwen2.5-VL-72B-Instruct` via DashScope"
