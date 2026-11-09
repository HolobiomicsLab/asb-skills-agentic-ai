---
name: visual-grounding-miou-evaluation
description: 'Use when comparing generated VQA-G annotations against baselines on
  grounding accuracy: you have predicted bboxes with object mentions and need mIoU,
  Acc@0.5IoU, and VQA-G scores on Visual7W or Viz Wiz conditions.'
when_to_use_negative:
- The task is to generate or refine VQA-G annotations (the generate-evaluate-refine
  loop) rather than to score grounding accuracy of existing annotations
- Evaluation data lacks ground-truth bounding boxes, so mIoU and Acc@0.5IoU cannot
  be computed against references
- The comparison target is annotation throughput or cost rather than visual grounding
  quality versus multimodal LLM baselines
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: AutoVQA-G
  role: Framework whose generated VQA-G annotations are scored with mIoU and VQA-G
    metrics; output JSON supplies bbox and consistency_score per image
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: GroundingDINO
  role: HTTP bbox endpoint producing the predicted bounding boxes that are evaluated
    by IoU against reference boxes
- name: GPT-4o
  role: Competing baseline (zero-shot, CoT, tool-assisted) whose VQA-G annotations
    receive the same mIoU/VQA-G scoring
- name: Gemini
  role: Competing baseline (zero-shot, CoT, tool-assisted) whose VQA-G annotations
    receive the same mIoU/VQA-G scoring
- name: Python
  role: Language in which main.py, the metric computation, and the table2_metrics.csv
    assembly are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1109/icassp55912.2026.11462754
    title: 'AutoVQA-G: Self-Improving Agentic Framework for Automated Visual Question
      Answering and Grounding Annotation'
  source_content_hash: sha256:5c871c9804d834ece5ca02ceda03086d5e4e271be74bc43d329744261e218447
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rohnson1999/AutoVQA-G
  tools:
  - AutoVQA-G
  - DeepSeek
  - GPT-4o
  - Gemini
  - GroundingDINO
  - MiniCPM-o
  - Python
  - Qwen2.5-VL-72B-Instruct
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# visual-grounding-miou-evaluation

## Summary

Protocol for scoring visual grounding quality of VQA-G annotations using mIoU, Acc@0.5IoU, and the overall VQA-G metric, used to benchmark AutoVQA-G against zero-shot, CoT, and tool-assisted GPT-4o and Gemini baselines on Visual7W and Viz Wiz.

## When to use

Use when comparing generated VQA-G annotations against baselines on grounding accuracy: you have predicted bboxes with object mentions and need mIoU, Acc@0.5IoU, and VQA-G scores on Visual7W or Viz Wiz conditions.

## When NOT to use

- The task is to generate or refine VQA-G annotations (the generate-evaluate-refine loop) rather than to score grounding accuracy of existing annotations
- Evaluation data lacks ground-truth bounding boxes, so mIoU and Acc@0.5IoU cannot be computed against references
- The comparison target is annotation throughput or cost rather than visual grounding quality versus multimodal LLM baselines

## Inputs

- Visual7W tellingQA annotation JSON (list of dicts with an 'image' key)
- Viz Wiz evaluation annotation JSON
- Image directories for both evaluation conditions
- AutoVQA-G output JSON per image (bbox, object_description, consistency_score, is_successful)
- Baseline VQA-G annotations from GPT-4o and Gemini (zero-shot, CoT, tool-assisted variants)

## Outputs

- table2_metrics.csv with mIoU, Acc@0.5IoU, and VQA-G scores for AutoVQA-G and each baseline on Visual7W and Viz Wiz
- Per-image grounding scores (IoU between predicted and reference bboxes)
- Mismatch flags for reproduced values deviating from reference Table 2 values

## How to apply

Run AutoVQA-G (main.py) on the Visual7W tellingQA annotations and on the Viz Wiz evaluation annotations with the GroundingDINO bbox endpoint active, consistency threshold τ=0.9 and max_iter=5 for the generate-evaluate-refine loop; run the GPT-4o and Gemini baselines (zero-shot, CoT, tool-assisted) on the same two evaluation conditions. For each image, select the first sample whose consistency score passes the threshold, or otherwise the highest-scoring iteration, as recorded in the output JSON (bbox, consistency_score, is_successful). Compute visual grounding mIoU, Acc@0.5IoU (IoU ≥ 0.5 counts as correct), and the overall VQA-G score for AutoVQA-G and each baseline. Assemble the metrics into table2_metrics.csv and compare each reproduced value against the reference Table 2 values (AutoVQA-G mIoU 0.634 / VQA-G 0.747 on Visual7W; 0.649 / 0.737 on Viz Wiz), flagging any mismatch beyond tolerance and checking that the reported ordering — AutoVQA-G above all GPT-4o and Gemini baselines — is preserved.

## Related tools

- **AutoVQA-G** (Framework whose generated VQA-G annotations are scored with mIoU and VQA-G metrics; output JSON supplies bbox and consistency_score per image) — https://github.com/rohnson1999/AutoVQA-G
- **GroundingDINO** (HTTP bbox endpoint producing the predicted bounding boxes that are evaluated by IoU against reference boxes)
- **GPT-4o** (Competing baseline (zero-shot, CoT, tool-assisted) whose VQA-G annotations receive the same mIoU/VQA-G scoring)
- **Gemini** (Competing baseline (zero-shot, CoT, tool-assisted) whose VQA-G annotations receive the same mIoU/VQA-G scoring)
- **Python** (Language in which main.py, the metric computation, and the table2_metrics.csv assembly are executed)

## Examples

```
python main.py --annotations path/to/annotations. --image_dir path/to/images/ --output_dir output/ --threshold 0.9 --max_iter 5
```

## Evaluation signals

- Reproduced AutoVQA-G values match reference Table 2: mIoU 0.634 / VQA-G 0.747 on Visual7W and mIoU 0.649 / VQA-G 0.737 on Viz Wiz within tolerance
- Reproduced ordering matches Table 2: AutoVQA-G achieves superior visual grounding accuracy compared to all GPT-4o and Gemini baselines on both evaluation conditions
- Per-image selection rule respected: the first sample passing the 0.9 threshold, or otherwise the highest-scoring iteration, is used for scoring
- table2_metrics.csv contains entries for all baseline variants (zero-shot, CoT, tool-assisted) on both Visual7W and Viz Wiz
- Output JSON records is_successful and consistency_score per image so acceptance versus max_iter fallback is auditable

## Limitations

- Current computational overhead of the iterative refinement loop is a stated limitation
- Single-pass annotation systems compared as baselines are susceptible to VLM hallucinations, yielding inconsistent fidelity that can depress baseline scores
- When no sample passes the threshold within max_iter 5 iterations, the highest-scoring iteration is used instead, which may lower grounding scores on difficult images

## Evidence

- [other] Reproduce the main comparison table (Table 2): VQA-G score, mIoU, Acc@0.5IoU vs GPT-4o and Gemini baselines: [evidence span withheld — anchor status: not_found]
