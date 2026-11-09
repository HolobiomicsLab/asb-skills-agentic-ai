---
name: visual-question-answer-pair-generation
description: Use when you need to auto-generate VQA-G annotations (question, answer,
  object mention, bounding box) for a directory of images instead of manually annotating
  a VQA-G dataset.
when_to_use_negative:
- Your inputs are already human-verified VQA-G annotations rather than raw images
  needing draft generation.
- The task only requires question-answer pairs without grounding — the pipeline's
  Visual Grounding stage (object mention + GroundingDINO bbox) would add unnecessary
  compute, which the paper notes is a stated limitation.
- You cannot host the MiniCPM-o 2.6 generation server or the GroundingDINO HTTP endpoint
  that the modular generation stage depends on.
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: AutoVQA-G
  role: Framework whose modular generation pipeline (Caption Reasoning -> VQA Generation
    -> Visual Grounding) this skill reconstructs
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: MiniCPM-o 2.6
  role: Local generation server for caption reasoning, VQA pair generation, and object
    mention generation via http://localhost:8000/v1
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: GroundingDINO
  role: Maps the generated object mention to spatial bounding-box coordinates via
    a POST endpoint returning a detections list with bbox and confidence
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: Python
  role: Implementation language; modules live in modules.py with dataclass definitions
    in data_structures.py, invoked via main.py
  repo: https://github.com/rohnson1999/AutoVQA-G
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1109/icassp55912.2026.11462754
    title: 'AutoVQA-G: Self-Improving Agentic Framework for Automated Visual Question
      Answering and Grounding Annotation'
  source_content_hash: sha256:526447d8a47914b10bded71f7c5d7875f8228c65a88f9e20dd0786907792a8f8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rohnson1999/AutoVQA-G
  tools:
  - AutoVQA-G
  - GroundingDINO
  - MiniCPM-o
  - MiniCPM-o 2.6
  - Python
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# visual-question-answer-pair-generation

## Summary

Generates draft visual question-answer (and grounding) annotations per image by chaining Caption Reasoning, VQA Generation, and Visual Grounding with MiniCPM-o 2.6 and GroundingDINO, as implemented in AutoVQA-G's per-iteration draft generator.

## When to use

Use when you need to auto-generate VQA-G annotations (question, answer, object mention, bounding box) for a directory of images instead of manually annotating a VQA-G dataset.

## When NOT to use

- Your inputs are already human-verified VQA-G annotations rather than raw images needing draft generation.
- The task only requires question-answer pairs without grounding — the pipeline's Visual Grounding stage (object mention + GroundingDINO bbox) would add unnecessary compute, which the paper notes is a stated limitation.
- You cannot host the MiniCPM-o 2.6 generation server or the GroundingDINO HTTP endpoint that the modular generation stage depends on.

## Inputs

- Annotations JSON file: a list of dictionaries, each with at least an "image" key (e.g. [{"image": "image_001.jpg"}])
- Directory of input images (--image_dir)
- Prompt templates from prompts/system_prompts.py
- Environment configuration: GENERATION_API_KEY (MiniCPM-o 2.6 endpoint), GROUNDING_DINO_URL (http://localhost:8001/predict)

## Outputs

- A single JSON results file in output_dir; each image contributes one record with fields: image_identifier, question, answer, object_description, bbox (four-coordinate list), consistency_score, iteration, is_successful, CoT_critique, timestamp

## How to apply

Load an annotations JSON (a list of dictionaries each with at least an 'image' key) and resolve image paths against the image directory. For each image, query the local MiniCPM-o 2.6 server (OpenAI-compatible endpoint http://localhost:8000/v1) using prompt templates from prompts/system_prompts.py to produce the structured caption context C_R; then condition MiniCPM-o 2.6 on C_R to generate the draft question-answer pair (q_t, a_t). For grounding, first generate the object mention with MiniCPM-o 2.6, then POST to the GroundingDINO endpoint (http://localhost:8001/predict) with 'image', 'query', 'box_threshold', and 'text_threshold' and parse the returned 'detections' list ('bbox', 'confidence') — the two-stage design follows the paper's localization where a mention m_t is deterministically mapped to a bounding box via b_t = arg max p(b | I, m_t) (Eq. 4). Assemble each record with question, answer, object_description, bbox (four-coordinate list), consistency_score, and iteration metadata, and write all records as a single JSON file into output_dir. Drafts are then passed downstream for CoT consistency scoring against the acceptance threshold tau = 0.9 (max 5 refinement iterations).

## Related tools

- **AutoVQA-G** (Framework whose modular generation pipeline (Caption Reasoning -> VQA Generation -> Visual Grounding) this skill reconstructs) — https://github.com/rohnson1999/AutoVQA-G
- **MiniCPM-o 2.6** (Local generation server for caption reasoning, VQA pair generation, and object mention generation via http://localhost:8000/v1) — https://github.com/rohnson1999/AutoVQA-G
- **GroundingDINO** (Maps the generated object mention to spatial bounding-box coordinates via a POST endpoint returning a detections list with bbox and confidence) — https://github.com/rohnson1999/AutoVQA-G
- **Python** (Implementation language; modules live in modules.py with dataclass definitions in data_structures.py, invoked via main.py) — https://github.com/rohnson1999/AutoVQA-G

## Examples

```
python main.py --annotations path/to/annotations. --image_dir path/to/images/ --output_dir output/ --threshold 0.9 --max_iter 5
```

## Evaluation signals

- Output JSON file exists, is parseable, and every per-image record contains the fields question, answer, object_description, and bbox.
- Every bbox value is a four-coordinate list (e.g. [120, 85, 340, 260]).
- Each record's consistency_score can be compared against the acceptance threshold tau = 0.9; accepted records should satisfy S_t >= tau or be the highest-scoring iteration within --max_iter (default 5).
- is_successful flags and iteration metadata show the generate-evaluate-refine loop ran and prompt optimization was reset per image (one sample's refinement history does not affect the next).
- Records carry a timestamp and image_identifier resolving correctly against the supplied image_dir.

## Limitations

- Single-pass generation is susceptible to VLM hallucinations, so drafts must pass the CoT Consistency Evaluation before acceptance.
- Computational overhead of the iterative generate-evaluate-refine loop is a stated limitation of the framework.
- The consistency threshold and endpoints (MiniCPM-o 2.6 at http://localhost:8000/v1, GroundingDINO at http://localhost:8001/predict) must be configurable; GroundingDINO must accept POST with 'image', 'query', 'box_threshold', 'text_threshold' and return a 'detections' list containing 'bbox' and 'confidence'.
- Prompt optimization is reset per image, so cross-image transfer of refinement history is not performed.

## Evidence

- [other] The VG Generation module localizes in two stages: the first stage generates an object mention, and the second stage uses a deterministic function to map the mention to spatial bounding-box coordinates (Eq. 4: b_t = arg max p(b | I, m_t)).: [evidence span withheld — anchor status: not_found]
