---
name: multimodal-llm-baseline-benchmarking
description: Use when you must quantify whether an agentic VQA-G annotation pipeline
  outperforms multimodal LLMs on visual grounding accuracy (mIoU, Acc@0.5IoU) and
  overall VQA-G score across fixed evaluation conditions.
when_to_use_negative:
- Input is already an annotated VQA-G dataset with ground-truth bounding boxes and
  you only need model training rather than system comparison
- The evaluation task lacks spatial grounding annotations, so mIoU and Acc@0.5IoU
  cannot be computed and only QA accuracy would be measured
- Baselines and the framework cannot be run on identical evaluation conditions, making
  the comparison confounded
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: AutoVQA-G
  role: Framework under test; its generate-evaluate-refine outputs are benchmarked
    against the LLM baselines
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: GPT-4o
  role: Competing multimodal LLM baseline run in zero-shot, CoT, and tool-assisted
    variants to produce competing VQA-G annotations
- name: Gemini
  role: Competing multimodal LLM baseline run in zero-shot, CoT, and tool-assisted
    variants on the same two evaluation conditions
- name: Python
  role: Environment for running main.py and computing mIoU / VQA-G metrics into table2_metrics.csv
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1109/icassp55912.2026.11462754
    title: 'AutoVQA-G: Self-Improving Agentic Framework for Automated Visual Question
      Answering and Grounding Annotation'
  source_content_hash: sha256:55d1a0a854492a142520448bf9c1ab54a46a7798391869fbc5f4b6165495c507
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# multimodal-llm-baseline-benchmarking

## Summary

Benchmark an automated annotation framework against zero-shot, CoT, and tool-assisted GPT-4o and Gemini baselines by computing visual grounding mIoU, Acc@0.5IoU, and overall VQA-G scores on identical evaluation conditions, reproducing the paper's Table 2 comparison showing AutoVQA-G achieves superior grounding accuracy.

## When to use

Use when you must quantify whether an agentic VQA-G annotation pipeline outperforms leading multimodal LLMs on visual grounding accuracy (mIoU, Acc@0.5IoU) and overall VQA-G score across fixed evaluation conditions.

## When NOT to use

- Input is already an annotated VQA-G dataset with ground-truth bounding boxes and you only need model training rather than system comparison
- The evaluation task lacks spatial grounding annotations, so mIoU and Acc@0.5IoU cannot be computed and only QA accuracy would be measured
- Baselines and the framework cannot be run on identical evaluation conditions, making the comparison confounded

## Inputs

- Visual7W tellingQA annotation JSON files with image paths
- Viz Wiz evaluation annotation JSON files with image paths
- Image directories for both evaluation conditions
- GPT-4o and Gemini baseline VQA-G annotations (zero-shot, CoT, tool-assisted variants) on the same conditions
- Reference Table 2 metric values for tolerance comparison

## Outputs

- table2_metrics.csv with mIoU, Acc@0.5IoU, and VQA-G score per system per evaluation condition
- Reproduced comparison table flagging any baseline-vs-reference mismatch beyond tolerance
- Selected per-image annotation JSON records with consistency_score and iteration fields

## How to apply

Run the framework under test (AutoVQA-G via main.py with threshold 0.9 and max_iter 5) and each competing baseline (GPT-4o and Gemini in zero-shot, CoT, and tool-assisted variants) on the same evaluation annotations, here the Visual7W tellingQA and Viz Wiz conditions. For each image, select the first sample whose consistency score passes the threshold, or otherwise the highest-scoring iteration. Compute visual grounding mIoU and Acc@0.5IoU from the predicted versus reference bounding boxes, aggregate them into the overall VQA-G score, and assemble the metrics into a table2_metrics.csv. Validate by checking the reproduced ordering matches Table 2: AutoVQA-G mIoU 0.634 / VQA-G 0.747 on Visual7W and 0.649 / 0.737 on Viz Wiz, exceeding all GPT-4o and Gemini baselines within tolerance.

## Related tools

- **AutoVQA-G** (Framework under test; its generate-evaluate-refine outputs are benchmarked against the LLM baselines) — https://github.com/rohnson1999/AutoVQA-G
- **GPT-4o** (Competing multimodal LLM baseline run in zero-shot, CoT, and tool-assisted variants to produce competing VQA-G annotations)
- **Gemini** (Competing multimodal LLM baseline run in zero-shot, CoT, and tool-assisted variants on the same two evaluation conditions)
- **Python** (Environment for running main.py and computing mIoU / VQA-G metrics into table2_metrics.csv)

## Examples

```
python main.py --annotations path/to/annotations. --image_dir path/to/images/ --output_dir output/ --threshold 0.9 --max_iter 5
```

## Evaluation signals

- Reproduced AutoVQA-G mIoU on Visual7W matches 0.634 and VQA-G score matches 0.747 within tolerance
- Reproduced AutoVQA-G mIoU on Viz Wiz matches 0.649 and VQA-G score matches 0.737 within tolerance
- Table 2 ordering holds: AutoVQA-G grounding accuracy exceeds every GPT-4o and Gemini baseline on both conditions
- Per-image output selection rule verified: first sample with consistency_score >= 0.9, else highest-scoring of max 5 iterations
- Metrics table schema is valid table2_metrics.csv with rows for each system x condition combination

## Limitations

- Computational overhead is a stated limitation of the framework being benchmarked
- No changelog available for the repository, so code version drift during reproduction cannot be tracked
- Comparison is limited to Visual7W and Viz Wiz evaluation conditions; results may not generalize to other domains

## Evidence
