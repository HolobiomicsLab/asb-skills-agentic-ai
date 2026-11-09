---
name: chain-of-thought-configuration-comparison
description: Use when you need to quantify how removing or reducing multi-chain CoT
  reasoning (chain count n, chains-per-perspective m) degrades harmful-meme detection
  accuracy/AUROC/F1 versus a full multi-CoT setting.
when_to_use_negative:
- 'When the goal is to reproduce the model end-to-end: the EVL-MCoT repository only
  contains the train/ pipeline (models/, configs/, scripts/ are marked ''coming soon''),
  so configurations cannot be re-run from the README alone'
- When evaluating datasets or metrics outside Hateful Memes Test Seen/Unseen and MultiOFF,
  since the anchor values and deltas are specific to these benchmarks
- When comparing across different MLLMs indiscriminately — Hateful Memes numbers use
  LLaVA while MultiOFF uses GPT-4, so deltas are not MLLM-controlled
edam_operation: http://edamontology.org/operation_2945
tools:
- name: LLaVA-1.5-7B
  role: MLLM used to generate the multi-CoT reasoning chains for the Hateful Memes
    evaluation setting (n=3, m=3)
  repo: https://github.com/BGWH123/EVL-MCoT
- name: GPT-4
  role: MLLM used to generate the multi-CoT reasoning chains for the MultiOFF evaluation
    setting (n=3, m=3)
- name: Long-CLIP
  role: Frozen text/visual encoder supporting the long (150-word) CoT sequences; removing
    it significantly reduces ACC and AUROC, so it must stay fixed across compared
    configurations
  repo: https://github.com/BGWH123/EVL-MCoT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/978-981-95-3349-7_31
    title: 'EVL-MCoT: Enhanced Vision-Language Multi-CoT for Harmful Meme Detection'
  source_content_hash: sha256:44dfaba99b2e40e19575ab9fd8ee245443c371037c0e1bf9614956e79df90765
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/EVL-MCoT
  tools:
  - GPT-4
  - LLaVA
  - LLaVA-1.5-7B
  - Long-CLIP
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# chain-of-thought-configuration-comparison

## Summary

Ablation-analysis skill for quantifying how chain-of-thought reasoning configuration (total chain count n, chains-per-perspective m) affects multimodal meme-detection performance, by computing per-metric deltas of ablated CoT settings against the full (n=3, m=3) EVL-MCoT anchor.

## When to use

Use when you need to quantify how removing or reducing multi-chain CoT reasoning (chain count n, chains-per-perspective m) degrades harmful-meme detection accuracy/AUROC/F1 versus a full multi-CoT setting.

## When NOT to use

- When the goal is to reproduce the model end-to-end: the EVL-MCoT repository only contains the train/ pipeline (models/, configs/, scripts/ are marked 'coming soon'), so configurations cannot be re-run from the README alone
- When evaluating datasets or metrics outside Hateful Memes Test Seen/Unseen and MultiOFF, since the anchor values and deltas are specific to these benchmarks
- When comparing across different MLLMs indiscriminately — Hateful Memes numbers use LLaVA while MultiOFF uses GPT-4, so deltas are not MLLM-controlled

## Inputs

- Reported ablation results for CoT settings (n=1,m=1), (n=1,m=3), (n=3,m=1) from the article's result tables
- Full-setting (n=3,m=3) anchor results: 75.88 ACC / 79.25 AUROC on Hateful Memes Test Seen (LLaVA-1.5-7B MLLM); 70.0 ACC / 63.8 F1 on MultiOFF (GPT-4 MLLM)
- Dataset/metric specification per evaluation (Hateful Memes Test Seen: ACC, AUROC; MultiOFF: ACC, F1)

## Outputs

- Tidy comparison table with columns: dataset, CoT setting (n, m), metric, absolute value, delta versus (n=3,m=3)
- Grouped bar plot (matplotlib) visualizing per-metric deltas across CoT configurations
- Quantified conclusion on whether removing multi-CoT entirely (n=1, m=1) causes the largest degradation

## How to apply

Load reported results for the Multi-CoT settings (n=1,m=1), (n=1,m=3), (n=3,m=1), and (n=3,m=3) from the article's result tables for Hateful Memes Test Seen (ACC, AUROC) and MultiOFF (ACC, F1). Anchor the baseline to the full EVL-MCoT setting (n=3,m=3): 75.88 ACC / 79.25 AUROC on Test Seen evaluated with LLaVA as the MLLM, and 70.0 ACC / 63.8 F1 on MultiOFF evaluated with GPT-4 as the MLLM. Compute per-metric deltas (ablated minus full) for each configuration to separate the effect of chain count from the effect of balancing hateful vs benign chains. Assemble a tidy table (dataset, CoT setting, metric, value, delta) and render a grouped bar plot of deltas with matplotlib. The rationale is that single-chain reasoning is unreliable, so the (n=1,m=1) ablation should show the largest degradation, confirming structured multi-CoT reasoning matters.

## Related tools

- **LLaVA-1.5-7B** (MLLM used to generate the multi-CoT reasoning chains for the Hateful Memes evaluation setting (n=3, m=3)) — https://github.com/BGWH123/EVL-MCoT
- **GPT-4** (MLLM used to generate the multi-CoT reasoning chains for the MultiOFF evaluation setting (n=3, m=3))
- **Long-CLIP** (Frozen text/visual encoder supporting the long (150-word) CoT sequences; removing it significantly reduces ACC and AUROC, so it must stay fixed across compared configurations) — https://github.com/BGWH123/EVL-MCoT

## Examples

```
df['delta'] = df['ablated'] - df['full']; df.pivot(index='setting', columns='metric', values='delta').plot(kind='bar')  # matplotlib grouped bar plot of ablation deltas
```

## Evaluation signals

- The (n=3, m=3) row of the derived table exactly matches the reported 75.88 ACC / 79.25 AUROC on Hateful Memes Test Seen and 70.0 ACC / 63.8 F1 on MultiOFF
- Every delta is reproducible by subtracting the full-setting value from the ablated value in the source tables
- The (n=1, m=1) w/o-CoT configuration reproduces the expected largest degradation: 69.20 ACC / 69.70 AUROC on Test Seen (deltas −6.68 ACC, −9.55 AUROC) and 61.08 ACC / 55.70 F1 on MultiOFF (deltas −8.87 ACC, −7.98 F1)
- Each dataset/metric pairing is respected: ACC+AUROC for Hateful Memes, ACC+F1 for MultiOFF, with no cross-dataset mixing of metrics
- Bar plot of deltas orders configurations consistently with the tidy table (e.g., (n=1,m=1) shows the deepest bars)

## Limitations

- Deltas confound MLLM choice with CoT configuration: Hateful Memes results are evaluated with LLaVA while MultiOFF uses GPT-4
- Only ablation values for the four (n,m) grid points are reported; other configurations or Test Unseen deltas for intermediate settings are not fully available in the source tables
- The public repository is incomplete (models/, configs/, scripts/ 'coming soon'; no requirements.txt or pretrained weights), so the ablation cannot be independently re-executed from the README
- Ablated w/o-CoT results are not directly comparable in intent to ablations that only shorten chains (shortening chains while maintaining performance is explicitly listed as future work)

## Evidence

- [results] it reaches an accuracy of 75.88 and AUROC of 79.25 on Test Seen: "it reaches an accuracy of 75.88 and AUROC of 79.25 on Test Seen"
- [results] The experimental results in Table 2 show that EVL-MCoT achieves the best performance on the MultiOFF dataset, with an accuracy of 70.0 and an F1 score of 63.8.: [evidence span withheld — anchor status: not_found]
- [intro] their reasoning processes can be unreliable due to their reliance on a single reasoning chain, which makes them susceptible to inconsistencies and errors: "their reasoning processes can be unreliable due to their reliance on a single reasoning chain, which makes them susceptible to inconsistencies and errors, as well as being heavily influenced by the"
- [methods] The proposed EVL-MCoT is evaluated with the setting n=3, m=3 using LLaVA as the MLLM: "The proposed EVL-MCoT is evaluated with the setting n=3, m=3 using LLaVA as the MLLM"
- [readme] The core training code is available in the `train/` directory. Other components including model definitions, sample data, and configuration files will be added in upcoming updates.: "The core training code is available in the `train/` directory. Other components including model definitions, sample data, and configuration files will be added in upcoming updates."
