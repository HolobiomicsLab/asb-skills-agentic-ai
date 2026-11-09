---
name: vqag-annotation-pipeline-execution
description: Use when you need to automatically generate question-answer-annotation
  pairs with bounding-box grounding for an image set, replacing manual VQA-G annotation
  or brittle single-pass VLM generation.
when_to_use_negative:
- When annotations already exist and are manually verified high-fidelity VQA-G data
  — the framework targets unscalable manual annotation, not re-annotation of trusted
  data.
- When you need real-time/low-latency annotation — computational overhead is a stated
  limitation of the iterative loop.
- When no GroundingDINO bbox endpoint or LLM API endpoints (MiniCPM-o, Qwen verifier,
  DeepSeek optimizer) can be provisioned — the pipeline cannot run.
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: AutoVQA-G
  role: The self-improving agentic framework being executed; installed via pip from
    its repository and run through main.py
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: MiniCPM-o 2.6
  role: Local generation server (http://localhost:8000/v1) for caption reasoning,
    VQA generation, and object mention generation
- name: Qwen2.5-VL-72B-Instruct
  role: CoT verifier via DashScope for the two-stage Consistency Evaluation (content
    quality + bbox accuracy)
- name: DeepSeek V3
  role: Prompt optimizer via DeepSeek API; memory-augmented refinement of generation
    prompts from failed-sample critiques
- name: GroundingDINO
  role: HTTP endpoint (default http://localhost:8001/predict) returning bbox detections
    from image + object mention queries
- name: Python
  role: Runtime for main.py and pipeline.py (generate-evaluate-refine loop)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1109/icassp55912.2026.11462754
    title: 'AutoVQA-G: Self-Improving Agentic Framework for Automated Visual Question
      Answering and Grounding Annotation'
  source_content_hash: sha256:aee3f66605e0f9b63a0f07ca31edde0b5d14d305b3ae6a85ec8bfe589556f99e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rohnson1999/AutoVQA-G
  tools:
  - AutoVQA-G
  - DeepSeek
  - DeepSeek V3
  - GPT-4o
  - Gemini
  - GroundingDINO
  - MiniCPM-o
  - MiniCPM-o 2.6
  - Python
  - Qwen2.5-VL-72B-Instruct
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# vqag-annotation-pipeline-execution

## Summary

Executes the AutoVQA-G generate-evaluate-refine annotation pipeline to produce high-fidelity Visual Question Answering with Grounding (VQA-G) annotations, combining modular generation (caption, VQA, object mention + GroundingDINO bbox), CoT-based consistency verification, and memory-augmented prompt optimization.

## When to use

Use when you need to automatically generate question-answer-annotation pairs with bounding-box grounding for an image set, replacing manual VQA-G annotation or brittle single-pass VLM generation.

## When NOT to use

- When annotations already exist and are manually verified high-fidelity VQA-G data — the framework targets unscalable manual annotation, not re-annotation of trusted data.
- When you need real-time/low-latency annotation — computational overhead is a stated limitation of the iterative loop.
- When no GroundingDINO bbox endpoint or LLM API endpoints (MiniCPM-o, Qwen verifier, DeepSeek optimizer) can be provisioned — the pipeline cannot run.

## Inputs

- JSON annotations file (list of dicts, each with at least an "image" key, e.g. {"image": "image_001.jpg"})
- Directory of input images (--image_dir)
- Optional auth credentials: GENERATION_API_KEY, VERIFICATION_API_KEY, OPTIMIZER_API_KEY, GROUNDING_DINO_API_KEY

## Outputs

- JSON results file in output_dir per image with fields: image_identifier, question, answer, object_description, bbox, consistency_score, iteration, is_successful, CoT_critique, timestamp
- Run logs in the output directory

## How to apply

Clone the AutoVQA-G repository, install requirements with pip, and configure four endpoints via environment variables: MiniCPM-o 2.6 as the local generation server (caption/VQA/mention generation), Qwen2.5-VL-72B-Instruct via DashScope as the CoT verifier, DeepSeek V3 as the prompt optimizer, and a GroundingDINO HTTP endpoint for bounding boxes. Prepare a JSON annotation file where each entry has at least an "image" key, then run main.py with --annotations, --image_dir, --threshold 0.9 (consistency score acceptance threshold τ) and --max_iter 5. The pipeline performs Modular Generation (Caption Reasoning → VQA Generation → Visual Grounding), then two-stage CoT Consistency Verification (content quality + bbox accuracy) producing weighted consistency scores, then Memory-Augmented Prompt Optimization that refines prompts from critiques; the loop terminates when the consistency score S_t ≥ τ or max_iter is reached. Prompt optimization is reset per image so one sample's refinement history does not affect the next. Each image contributes one selected output: the first sample passing the threshold, or otherwise the highest-scoring iteration.

## Related tools

- **AutoVQA-G** (The self-improving agentic framework being executed; installed via pip from its repository and run through main.py) — https://github.com/rohnson1999/AutoVQA-G
- **MiniCPM-o 2.6** (Local generation server (http://localhost:8000/v1) for caption reasoning, VQA generation, and object mention generation)
- **Qwen2.5-VL-72B-Instruct** (CoT verifier via DashScope for the two-stage Consistency Evaluation (content quality + bbox accuracy))
- **DeepSeek V3** (Prompt optimizer via DeepSeek API; memory-augmented refinement of generation prompts from failed-sample critiques)
- **GroundingDINO** (HTTP endpoint (default http://localhost:8001/predict) returning bbox detections from image + object mention queries)
- **Python** (Runtime for main.py and pipeline.py (generate-evaluate-refine loop))

## Examples

```
python main.py --annotations path/to/annotations. --image_dir path/to/images/ --output_dir output/ --threshold 0.9 --max_iter 5
```

## Evaluation signals

- Each output record has consistency_score ≥ threshold (default τ = 0.9) or is flagged as the highest-scoring iteration when max_iter (default 5) is exhausted.
- Output JSON schema check: every record contains image_identifier, question, answer, object_description, bbox, consistency_score, iteration, is_successful, CoT_critique, timestamp.
- Each image yields exactly one selected result (first threshold-passing sample or best-scoring iteration), and prompt optimization history is reset between images.
- Downstream metric check: on Visual7W the reproduced annotations reach mIoU 0.634 / VQA-G 0.747; on Viz Wiz, mIoU 0.649 / VQA-G 0.737, matching Table 2 reference values within tolerance.
- bbox entries correspond to GroundingDINO detections accepted in the bbox accuracy verification stage of the two-stage assessment.

## Limitations

- Current computational overhead is a stated limitation of the iterative refine loop.
- Verification quality depends on external LLM endpoints (Qwen2.5-VL CoT verifier, DeepSeek optimizer); model hallucinations can still yield inconsistent fidelity if scores pass threshold prematurely.
- The repository provides no changelog, so version drift in the code relative to the paper is possible.

## Evidence
