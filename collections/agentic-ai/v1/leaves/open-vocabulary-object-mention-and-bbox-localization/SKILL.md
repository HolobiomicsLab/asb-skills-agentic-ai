---
name: open-vocabulary-object-mention-and-bbox-localization
description: Use when a VQA-G or visual grounding annotation pipeline must produce,
  per image, an open-vocabulary object description plus a four-coordinate bounding
  box without closed-set categories.
when_to_use_negative:
- The task only requires question-answer generation without spatial grounding — use
  the Caption Reasoning and VQA Generation modules alone
- Objects of interest belong to a fixed closed set of categories where a supervised
  detector with calibrated scores is available and preferred
- No GroundingDINO-compatible HTTP endpoint (POST with image/query/box_threshold/text_threshold
  returning a 'detections' list) can be deployed
edam_operation: http://edamontology.org/operation_3779
edam_topics:
- http://edamontology.org/topic_3395
tools:
- name: MiniCPM-o 2.6
  role: Local generation server that produces the open-vocabulary object mention conditioned
    on the caption reasoning context C_R
  repo: github.com/rohnson1999/AutoVQA-G
- name: GroundingDINO
  role: Open-vocabulary detector that maps the object mention to a bounding box via
    POST /predict returning 'detections' with 'bbox' and 'confidence'
  repo: github.com/rohnson1999/AutoVQA-G
- name: Python
  role: Implementation language for modules.py (Visual Grounding module) and data_structures.py
    in the AutoVQA-G repo
  repo: github.com/rohnson1999/AutoVQA-G
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1109/icassp55912.2026.11462754
    title: 'AutoVQA-G: Self-Improving Agentic Framework for Automated Visual Question
      Answering and Grounding Annotation'
  source_content_hash: sha256:6e535ae7524d6cc92a7101ce2d92d66682ec14b57e71b6549e5626c66604e316
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - AutoVQA-G
  - GroundingDINO
  - MiniCPM-o
  - MiniCPM-o 2.6
  - Python
  edam_operation: http://edamontology.org/operation_3779
  edam_topics:
  - http://edamontology.org/topic_3395
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# open-vocabulary-object-mention-and-bbox-localization

## Summary

Two-stage visual grounding procedure from AutoVQA-G in which a local VLM (MiniCPM-o 2.6) first generates a free-form object mention for an image, and a deterministic grounding step then maps that mention to spatial bounding-box coordinates (Eq. 4: b_t = arg max p(b | I, m_t)) via an open-vocabulary detector (GroundingDINO).

## When to use

Use when a VQA-G or visual grounding annotation pipeline must produce, per image, an open-vocabulary object description plus a four-coordinate bounding box without closed-set categories.

## When NOT to use

- The task only requires question-answer generation without spatial grounding — use the Caption Reasoning and VQA Generation modules alone
- Objects of interest belong to a fixed closed set of categories where a supervised detector with calibrated scores is available and preferred
- No GroundingDINO-compatible HTTP endpoint (POST with image/query/box_threshold/text_threshold returning a 'detections' list) can be deployed

## Inputs

- Image file resolved against --image_dir (e.g. image_001.jpg)
- Structured caption reasoning context C_R from the Caption Reasoning module (MiniCPM-o 2.6 output)
- GroundingDINO service parameters: image, query (object mention), box_threshold, text_threshold

## Outputs

- Object mention string stored as 'object_description' (e.g. 'red car on left')
- Four-coordinate bounding box list stored as 'bbox' (e.g. [120, 85, 340, 260]) with detection 'confidence'
- Per-image draft JSON record with fields question, answer, object_description, bbox, consistency_score, iteration metadata

## How to apply

In the AutoVQA-G draft-generation stage, first condition MiniCPM-o 2.6 (OpenAI-compatible endpoint at http://localhost:8000/v1, prompts from prompts/system_prompts.py) on the caption reasoning context C_R to generate an object mention m_t. Then send a POST request to the GroundingDINO endpoint (GROUNDING_DINO_URL, default http://localhost:8001/predict) with fields 'image', 'query' (the mention), 'box_threshold', and 'text_threshold', and parse the returned 'detections' list, taking the 'bbox' and 'confidence' of the best detection as the argmax localization b_t. Assemble the per-image record with question, answer, object_description, and bbox (a four-coordinate list), noting that bbox accuracy is separately verified downstream by the CoT Consistency Evaluation module. The mention-to-box mapping is deliberately deterministic so that grounding errors are isolated from generation errors and can be scored against the acceptance threshold tau = 0.9.

## Related tools

- **MiniCPM-o 2.6** (Local generation server that produces the open-vocabulary object mention conditioned on the caption reasoning context C_R) — github.com/rohnson1999/AutoVQA-G
- **GroundingDINO** (Open-vocabulary detector that maps the object mention to a bounding box via POST /predict returning 'detections' with 'bbox' and 'confidence') — github.com/rohnson1999/AutoVQA-G
- **Python** (Implementation language for modules.py (Visual Grounding module) and data_structures.py in the AutoVQA-G repo) — github.com/rohnson1999/AutoVQA-G

## Examples

```
curl -X POST http://localhost:8001/predict -d '{"image": "image_001.jpg", "query": "red car on left", "box_threshold": 0.3, "text_threshold": 0.25}'
```

## Evaluation signals

- Every per-image record contains an 'object_description' string and a 'bbox' that is a four-coordinate list
- GroundingDINO response parsed successfully into a 'detections' list with non-empty 'bbox' and 'confidence' fields
- Downstream bbox accuracy verification stage (second stage of the two-stage Consistency Evaluation) accepts the grounding, contributing to a consistency score S_t >= tau = 0.9
- Selected record is the first iteration passing the threshold, or the highest-scoring iteration when no iteration passes within --max_iter (default 5)
- Output JSON file in output_dir is parseable and each record's bbox coordinates are consistent with the object mention in the query sent to GroundingDINO

## Limitations

- Computational overhead of the local generation server plus a separate grounding service is a stated limitation of the framework
- Single-pass mentions are susceptible to VLM hallucinations; only the iterative generate-evaluate-refine loop with consistency scoring mitigates inconsistent fidelity
- GroundingDINO endpoint contract (fields and 'detections' list format) must match exactly; endpoints/models are only configurable in modules.py
- Prompt optimization state is reset per image, so grounding refinement history from one sample does not carry to the next

## Evidence

- [methods] Two-stage grounding rationale: [evidence span withheld — anchor status: not_found]
