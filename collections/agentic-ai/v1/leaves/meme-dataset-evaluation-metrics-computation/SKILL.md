---
name: meme-dataset-evaluation-metrics-computation
description: Use when reproducing or benchmarking a meme-classification model on Hateful
  Memes (Test Seen/Unseen) or MultiOFF and you need ACC/AUROC/F1 scores comparable
  to published baseline tables.
when_to_use_negative:
- Task is meme explanation or interpretation rather than binary hateful/offensive
  classification scoring — the skill only covers ACC/AUROC/F1 evaluation
- Dataset is not one of the Hateful Memes or MultiOFF benchmarks, since the metric
  protocol and reported comparison rows are specific to these two datasets
- Only a single reasoning chain or no CoT generation is available — EVL-MCoT scores
  assume the n=3, m=3 multi-CoT setting
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: Long-CLIP
  role: Frozen visual (248-token context) and text encoders producing the features
    scored during evaluation
  repo: https://github.com/BGWH123/EVL-MCoT
- name: LLaVA-1.5-7B
  role: MLLM backbone generating the benign/hateful multi-CoT chains whose predictions
    are scored on Hateful Memes
  repo: https://github.com/BGWH123/EVL-MCoT
- name: GPT-4
  role: MLLM backbone generating multi-CoT chains for the MultiOFF evaluation setting
  repo: https://github.com/BGWH123/EVL-MCoT
- name: AdamW
  role: Optimizer (lr 5e-5, weight decay 0.01, cosine scheduler) for the training
    pipeline whose checkpoints feed evaluation
  repo: https://github.com/BGWH123/EVL-MCoT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/978-981-95-3349-7_31
    title: 'EVL-MCoT: Enhanced Vision-Language Multi-CoT for Harmful Meme Detection'
  source_content_hash: sha256:cbf43e8f26918e22935abaedd2f08ec8ccfad89dafd9feb1671fff37f88204df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/EVL-MCoT
  tools:
  - AdamW
  - GPT-4
  - LLaVA-1.5-7B
  - Long-CLIP
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# meme-dataset-evaluation-metrics-computation

## Summary

Computes and validates benchmark evaluation metrics (ACC, AUROC, F1) for harmful meme detection across the Hateful Memes Test Seen/Test Unseen splits and the MultiOFF dataset, as defined by the EVL-MCoT evaluation protocol.

## When to use

Use when reproducing or benchmarking a meme-classification model on Hateful Memes (Test Seen/Unseen) or MultiOFF and you need ACC/AUROC/F1 scores comparable to published baseline tables.

## When NOT to use

- Task is meme explanation or interpretation rather than binary hateful/offensive classification scoring — the skill only covers ACC/AUROC/F1 evaluation
- Dataset is not one of the Hateful Memes or MultiOFF benchmarks, since the metric protocol and reported comparison rows are specific to these two datasets
- Only a single reasoning chain or no CoT generation is available — EVL-MCoT scores assume the n=3, m=3 multi-CoT setting

## Inputs

- Hateful Memes dataset (10,000+ multimodal examples, Test Seen and Test Unseen splits, hateful/non-hateful labels)
- MultiOFF dataset (743 memes, offensive/non-offensive labels)
- Model predictions or inference outputs (e.g., from EVL-MCoT with LLaVA-1.5-7B or GPT-4 backbone)
- Baseline results from Tables 1 and 2 of the EVL-MCoT paper

## Outputs

- ACC and AUROC on Hateful Memes Test Seen and Test Unseen
- ACC and F1 on MultiOFF
- benchmark_results.csv with EVL-MCoT and baseline rows matching paper Tables 1 and 2

## How to apply

Run inference over the Hateful Memes test splits (over 10,000 multimodal examples labeled hateful or non-hateful) and the MultiOFF dataset (743 memes labeled offensive or non-offensive), then compute accuracy and AUROC on Test Seen and Test Unseen, and accuracy plus F1 on MultiOFF. Compile each method's row (e.g., EVL-MCoT 75.88/79.25 Test Seen, 75.57/79.40 Test Unseen, 70.0/63.8 on MultiOFF) together with baselines (Image-Grid, MMBT-Region, ViLBERT, VisualBERTCOCO, LLaVA-1.5-7B, GPT-4o, MOMENTA, RGCL, SFT variants) into a benchmark_results.csv mirroring Tables 1 and 2 of the paper. For EVL-MCoT specifically, pair m benign with n hateful CoT chains under the n=3, m=3 setting — using LLaVA-1.5-7B as the MLLM for Hateful Memes and GPT-4 for MultiOFF — before scoring. Treat the published values (Test Seen ACC 75.88/AUROC 79.25; Test Unseen ACC 75.57/AUROC 79.40; MultiOFF ACC 70.0/F1 63.8) as the validation targets when reproducing.

## Related tools

- **Long-CLIP** (Frozen visual (248-token context) and text encoders producing the features scored during evaluation) — https://github.com/BGWH123/EVL-MCoT
- **LLaVA-1.5-7B** (MLLM backbone generating the benign/hateful multi-CoT chains whose predictions are scored on Hateful Memes) — https://github.com/BGWH123/EVL-MCoT
- **GPT-4** (MLLM backbone generating multi-CoT chains for the MultiOFF evaluation setting) — https://github.com/BGWH123/EVL-MCoT
- **AdamW** (Optimizer (lr 5e-5, weight decay 0.01, cosine scheduler) for the training pipeline whose checkpoints feed evaluation) — https://github.com/BGWH123/EVL-MCoT

## Evaluation signals

- Test Seen row must match Table 1: ACC 75.88, AUROC 79.25
- Test Unseen row must match Table 1: ACC 75.57, AUROC 79.40
- MultiOFF row must match Table 2: ACC 70.0, F1 63.8
- All baseline rows from Tables 1-2 (Image-Grid, MMBT-Region, ViLBERT, VisualBERTCOCO, LLaVA-1.5-7B, GPT-4o, MOMENTA, SFT variants, RGCL) are present in benchmark_results.csv
- EVL-MCoT should be the best-performing method on both Hateful Memes splits and MultiOFF, as reported

## Limitations

- Repository only exposes the train/ pipeline; model definitions, configs, preprocessing scripts, and pretrained weights are marked 'coming soon', complicating full reproduction
- Ablations show removing Long-CLIP significantly reduces ACC and AUROC, so metric values depend on the exact encoder setup
- Metric comparability depends on the MLLM backbone choice (LLaVA-1.5-7B for Hateful Memes, GPT-4 for MultiOFF) under the n=3, m=3 CoT setting

## Evidence

- [results] Hatefulmeme. The experiment results in Table 1 show that EVL-MCoT achieves the best performance on both Test Seen and Test Unseen datasets.: "Hatefulmeme. The experiment results in Table 1 show that EVL-MCoT achieves the best performance on both Test Seen and Test Unseen datasets."
- [methods] EVL-MCoT 75.88 79.25 75.57 79.40: "EVL-MCoT 75.88 79.25 75.57 79.40"
- [methods] The MultiOFF dataset consists of 743 memes, annotated as offensive or non-offensive.: [evidence span withheld — anchor status: not_found]
- [methods] The Hateful Memes dataset contains over 10,000 multimodal examples for detecting hate speech in memes, labeled as hateful or non-hateful.: "The Hateful Memes dataset contains over 10,000 multimodal examples for detecting hate speech in memes, labeled as hateful or non-hateful."
- [readme] The core training code is available in the `train/` directory. Other components including model definitions, sample data, and configuration files will be added in upcoming updates.: "The core training code is available in the `train/` directory. Other components including model definitions, sample data, and configuration files will be added in upcoming updates."
