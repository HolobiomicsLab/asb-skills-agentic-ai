---
name: vqa-content-quality-scoring
description: Use when a generated VQA question-answer pair (with image and optional
  grounding bbox) must be verified for content fidelity before acceptance, and heuristic
  checks are too brittle to trust.
when_to_use_negative:
- You only need grounding-box accuracy and not question-answer content fidelity —
  stage E_vg alone covers bbox accuracy verification.
- Input images have no candidate draft annotation yet; generation (Caption Reasoning
  → VQA Generation → Visual Grounding) must run first.
- A single-pass, cheap heuristic check suffices and interpretability of the critique
  is not required — although the paper argues such heuristics are brittle in complex
  or out-of-distribution scenarios.
tools:
- name: Qwen2.5-VL-72B-Instruct
  role: CoT verifier called via DashScope to produce the step-wise critique and content-quality
    score S_vqa
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: AutoVQA-G
  role: Framework hosting the two-stage Consistency Evaluation, weighted score aggregation
    (Eq. 7), and threshold acceptance (Eq. 9)
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: Python
  role: Runtime for main.py/pipeline.py/modules.py that execute the scoring and write
    per-image verification records
  repo: https://github.com/rohnson1999/AutoVQA-G
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1109/icassp55912.2026.11462754
    title: 'AutoVQA-G: Self-Improving Agentic Framework for Automated Visual Question
      Answering and Grounding Annotation'
  source_content_hash: sha256:b78732a1db4cca2f937cfbc68f845c6ba4cee9e14f3087f28c51cd049b6da8bf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rohnson1999/AutoVQA-G
  tools:
  - AutoVQA-G
  - Python
  - Qwen2.5-VL-72B-Instruct
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# vqa-content-quality-scoring

## Summary

Content-quality scoring stage of AutoVQA-G's CoT Consistency Evaluation module: a CoT-prompted Qwen2.5-VL-72B-Instruct verifier critiques a candidate VQA-G question-answer pair against its source image and emits a step-wise critique plus a content-quality score S_vqa that is later weighted (with S_vg) into the acceptance score S_t.

## When to use

Use when a generated VQA question-answer pair (with image and optional grounding bbox) must be verified for content fidelity before acceptance, and heuristic checks are too brittle to trust.

## When NOT to use

- You only need grounding-box accuracy and not question-answer content fidelity — stage E_vg alone covers bbox accuracy verification.
- Input images have no candidate draft annotation yet; generation (Caption Reasoning → VQA Generation → Visual Grounding) must run first.
- A single-pass, cheap heuristic check suffices and interpretability of the critique is not required — although the paper argues such heuristics are brittle in complex or out-of-distribution scenarios.

## Inputs

- Draft VQA-G annotation D_t: question, answer, object mention, GroundingDINO bbox (JSON list of dicts each with an 'image' key)
- Source image file from --image_dir
- Acceptance threshold tau (default 0.9 via --threshold)
- DashScope VERIFICATION_API_KEY for Qwen2.5-VL-72B-Instruct

## Outputs

- Content-quality score S_vqa from the CoT verifier
- Step-wise critique text, concatenated with the bbox critique into C_t
- Per-image verification record: question, answer, object_description, bbox, consistency_score, iteration, is_successful, CoT_critique, timestamp (JSON in --output_dir)

## How to apply

Load the draft annotation D_t (question, answer, object mention, GroundingDINO bbox) together with its source image, then call Qwen2.5-VL-72B-Instruct via DashScope (key in VERIFICATION_API_KEY) with CoT prompting to produce a step-wise visual critique and a content-quality score S_vqa for the question-answer pair. Run this as stage E_vqa of the two-stage assessment, in parallel with (or before) stage E_vg, which scores bbox accuracy S_vg with the same verifier model. Aggregate the two stages as S_t = w_vqa*S_vqa + w_vg*S_vg (Eq. 7) and concatenate the step-wise critiques into a single critique string C_t (Eq. 8). Apply the acceptance rule S_t >= tau (Eq. 9), with tau defaulting to 0.9 via --threshold; if the draft fails, emit the concatenated CoT_critique so the memory-augmented Prompt Optimization agent (DeepSeek V3) can refine the generation prompt in the next iteration. Iterate up to --max_iter (default 5), then select the first sample passing the threshold or, failing that, the highest-scoring iteration for the image.

## Related tools

- **Qwen2.5-VL-72B-Instruct** (CoT verifier called via DashScope to produce the step-wise critique and content-quality score S_vqa) — https://github.com/rohnson1999/AutoVQA-G
- **AutoVQA-G** (Framework hosting the two-stage Consistency Evaluation, weighted score aggregation (Eq. 7), and threshold acceptance (Eq. 9)) — https://github.com/rohnson1999/AutoVQA-G
- **Python** (Runtime for main.py/pipeline.py/modules.py that execute the scoring and write per-image verification records) — https://github.com/rohnson1999/AutoVQA-G

## Examples

```
python main.py --annotations path/to/annotations. --image_dir path/to/images/ --output_dir output/ --threshold 0.9 --max_iter 5
```

## Evaluation signals

- consistency_score in the output record reflects S_t = w_vqa*S_vqa + w_vg*S_vg and lies in [0, 1]
- is_successful is true exactly when consistency_score >= tau (default 0.9)
- CoT_critique field contains concatenated step-wise critiques rather than a single-pass verdict
- One selected record per image: the first passing sample or, failing that, the highest-scoring iteration (iteration <= --max_iter, default 5)
- Verification records include timestamp and full schema (question, answer, object_description, bbox, iteration)

## Limitations

- Verification depends on a remote Qwen2.5-VL-72B-Instruct endpoint via DashScope; failures or quota issues there block scoring
- Computational overhead of the iterative CoT generate-evaluate-refine loop is a stated limitation
- Prompt optimization is reset per image, so refinement history is not reused across samples, and max_iter (default 5) may end the loop with no sample passing tau

## Evidence
