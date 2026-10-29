---
name: benchmark-results-table-compilation
description: Use when reproducing or auditing a paper's reported metrics and you need
  a machine-readable table of the proposed method's results plus every baseline row
  from its result tables (e.g., Tables 1 and 2 of the EVL-MCoT paper).
when_to_use_negative:
- When you need to re-train or re-evaluate baselines yourself rather than compile
  their published numbers from the paper's tables
- When the paper's result tables cannot be located or their metric schema (ACC/AUROC
  vs ACC/F1) is unclear, since row values cannot be validated
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: Long-CLIP
  role: frozen visual and text encoder used to produce the visual/text representations
    whose alignment scores yield the ACC and AUROC values being compiled
  repo: https://github.com/BGWH123/EVL-MCoT
- name: LLaVA-1.5-7B
  role: MLLM backbone that generates the benign/hateful multi-CoT chains whose inference
    results feed the Hateful Memes rows of the table
  repo: https://github.com/BGWH123/EVL-MCoT
- name: GPT-4
  role: MLLM backbone for generating multi-CoT chains in the MultiOFF evaluation whose
    results populate the Table 2 row
- name: AdamW
  role: optimizer (learning rate 5e-5, weight decay 0.01, cosine scheduler) used when
    training the alignment layer before metrics are recorded
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/978-981-95-3349-7_31
    title: 'EVL-MCoT: Enhanced Vision-Language Multi-CoT for Harmful Meme Detection'
  source_content_hash: sha256:92b838f308766ecb430a3511b9f5f319d3694a5fe9a2b95e69f49636da16c13f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/EVL-MCoT
  tools:
  - AdamW
  - GPT-4
  - LLaVA-1.5-7B
  - Long-CLIP
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# benchmark-results-table-compilation

## Summary

Compiles a method's reported benchmark scores together with all baseline rows into a structured CSV that mirrors the paper's result tables, enabling systematic comparison of EVL-MCoT (75.88/79.25 Test Seen ACC/AUROC, etc.) against baselines like ViLBERT, VisualBERTCOCO, LLaVA-1.5-7B, and GPT-4o.

## When to use

Use when reproducing or auditing a paper's reported metrics and you need a machine-readable table of the proposed method's results plus every baseline row from its result tables (e.g., Tables 1 and 2 of the EVL-MCoT paper).

## When NOT to use

- When you need to re-train or re-evaluate baselines yourself rather than compile their published numbers from the paper's tables
- When the paper's result tables cannot be located or their metric schema (ACC/AUROC vs ACC/F1) is unclear, since row values cannot be validated

## Inputs

- Inference outputs from the EVL-MCoT pipeline (ACC and AUROC on Hateful Memes test seen/unseen splits; ACC and F1 on MultiOFF's 743 memes)
- Baseline result rows from Table 1 (Hateful Memes Test Seen/Test Unseen) and Table 2 (MultiOFF) of the paper

## Outputs

- benchmark_results.csv with one row per method (EVL-MCoT plus all baselines) and metric columns matching Tables 1 and 2

## How to apply

Run the reproduction pipeline (multi-CoT generation with n=3, m=3 chains, Long-CLIP encoding, prototype-guided token fusion, and context-guided attention) to obtain inference scores, then compile the proposed method's row into benchmark_results.csv: for Hateful Memes record ACC 75.88 and AUROC 79.25 on Test Seen and ACC 75.57 and AUROC 79.40 on Test Unseen; for MultiOFF record ACC 70.0 and F1 63.8. Add one row per baseline from Tables 1 and 2 (Image-Grid, MMBT-Region, ViLBERT, VisualBERTCOCO, LLaVA-1.5-7B, GPT-4o, MOMENTA, SFT variants, RGCL, etc.) with the metric columns matching each table's schema. Validate by checking that the compiled EVL-MCoT row values exactly match the reported metrics and that all baseline rows are present.

## Related tools

- **Long-CLIP** (frozen visual and text encoder used to produce the visual/text representations whose alignment scores yield the ACC and AUROC values being compiled) — https://github.com/BGWH123/EVL-MCoT
- **LLaVA-1.5-7B** (MLLM backbone that generates the benign/hateful multi-CoT chains whose inference results feed the Hateful Memes rows of the table) — https://github.com/BGWH123/EVL-MCoT
- **GPT-4** (MLLM backbone for generating multi-CoT chains in the MultiOFF evaluation whose results populate the Table 2 row)
- **AdamW** (optimizer (learning rate 5e-5, weight decay 0.01, cosine scheduler) used when training the alignment layer before metrics are recorded)

## Examples

```
python compile_benchmark_results.py --evl_mcot "75.88,79.25,75.57,79.40,70.0,63.8" --tables 1,2 --output benchmark_results.csv
```

## Evaluation signals

- The compiled EVL-MCoT row matches Table 1 exactly: Test Seen ACC 75.88, AUROC 79.25; Test Unseen ACC 75.57, AUROC 79.40
- The compiled MultiOFF row matches Table 2 exactly: ACC 70.0 and F1 63.8
- Every baseline row from Tables 1 and 2 (Image-Grid, MMBT-Region, ViLBERT, VisualBERTCOCO, LLaVA-1.5-7B, GPT-4o, MOMENTA, SFT variants, RGCL, etc.) is present in benchmark_results.csv
- Metric columns are consistent with each table's schema: ACC and AUROC for Hateful Memes rows, ACC and F1 for MultiOFF rows
- EVL-MCoT scores rank highest across the compiled rows, as the paper claims best performance on Test Seen, Test Unseen, and MultiOFF

## Limitations

- Reproduced scores depend on the multi-CoT setting (n=3, m=3) and MLLM choice (LLaVA-1.5-7B for Hateful Memes, GPT-4 for MultiOFF); deviations change the numbers being compiled
- The repository README notes that model definitions, configs, and evaluation scripts are 'coming soon', so full reproduction may be blocked and the table may need to rely on paper-reported values
- Metrics for baselines are taken as published; no re-evaluation of baseline rows is performed in this compilation step

## Evidence

- [results] it reaches an accuracy of 75.88 and AUROC of 79.25 on Test Seen: "it reaches an accuracy of 75.88 and AUROC of 79.25 on Test Seen"
- [methods] EVL-MCoT 75.88 79.25 75.57 79.40: "EVL-MCoT 75.88 79.25 75.57 79.40"
- [results] EVL-MCoT achieves the best performance on the MultiOFF dataset, with an accuracy of 70.0 and an F1 score of 63.8: [evidence span withheld — anchor status: not_found]
- [methods] The proposed EVL-MCoT is evaluated with the setting n=3, m=3 using LLaVA as the MLLM: "The proposed EVL-MCoT is evaluated with the setting n=3, m=3 using LLaVA as the MLLM"
- [methods] Hateful Memes dataset contains over 10,000 multimodal examples: "The Hateful Memes dataset contains over 10,000 multimodal examples for detecting hate speech in memes, labeled as hateful or non-hateful."
