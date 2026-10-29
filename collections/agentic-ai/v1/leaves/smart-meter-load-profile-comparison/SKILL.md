---
name: smart-meter-load-profile-comparison
description: Use when synthetic/hourly household electricity profiles must be validated
  against a real smart-meter reference dataset with socio-economic survey data, e.g.
  after Stage 3 of a generation pipeline.
when_to_use_negative:
- No completed WattCouncil pipeline run exists yet — there are no stage3_consumption_*
  outputs to compare
- The reference dataset lacks household segments matching the targeted (num_people,
  house_type, composition) combinations used for generation, so alignment would be
  invalid
- The goal is cost/token or role-performance analysis of the LLM council rather than
  consumption-profile validation (use scripts/analyze_logs.py instead)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: WattCouncil
  role: Source of the generated Stage 3 hourly consumption profiles and the convert_outputs.py
    script used to convert JSON outputs to tabular form
  repo: github.com/Singularity-AI-Lab/wattcouncil
- name: Python
  role: Runtime environment (Python 3.11 conda env llm_council) for running convert_outputs.py
    and the consistency evaluation
  repo: github.com/Singularity-AI-Lab/wattcouncil
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1145/3744256.3812595
    title: 'WattCouncil: Context-Aware Household Energy Scenario Generation With Governed
      LLMs'
  source_content_hash: sha256:b61a0bc506509ee1d74babed73f68e74557e900dea0cd635b49eef0860ea4e46
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - WattCouncil
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# smart-meter-load-profile-comparison

## Summary

Validates LLM-generated household electricity profiles (WattCouncil Stage 3 hourly consumption) against a real-world high-resolution smart-meter dataset (CER, >1 year of load measurements for 4232 households), by aligning generated profiles to matching CER household segments and comparing hourly and seasonal consumption patterns.

## When to use

Use when synthetic/hourly household electricity profiles must be validated against a real smart-meter reference dataset with socio-economic survey data, e.g. after Stage 3 of a generation pipeline.

## When NOT to use

- No completed WattCouncil pipeline run exists yet — there are no stage3_consumption_* outputs to compare
- The reference dataset lacks household segments matching the targeted (num_people, house_type, composition) combinations used for generation, so alignment would be invalid
- The goal is cost/token or role-performance analysis of the LLM council rather than consumption-profile validation (use scripts/analyze_logs.py instead)

## Inputs

- WattCouncil Stage 3 generated hourly consumption profiles (stage3_consumption_* files from a timestamped outputs/run_* directory)
- CER reference dataset: over a year of smart-meter load measurements for 4232 households plus survey-based socio-economic information
- WattCouncil targeted-generation config (config/runtime_targeted.yaml) listing representative_combos such as [1, 5, 1], [2, 4, 2], [3, 3, 2]

## Outputs

- cond_cer_evaluation. — valid JSON containing the computed framework-vs-CER consistency comparison (hourly and seasonal load-magnitude and daily-shape metrics)

## How to apply

Load the WattCouncil Stage 3 generated hourly consumption profiles (stage3_consumption_* files) from the timestamped outputs/run_* directory of a completed pipeline run, and load the CER reference dataset with its survey-based socio-economic information. Convert the generated JSON consumption outputs into tabular form with WattCouncil's convert_outputs.py script. Align generated profiles with CER household segments by matching the (num_people, house_type, composition) combinations used in targeted generation, e.g. [1, 5, 1] (1 person, Bungalow, alone), [2, 4, 2], and [3, 3, 2] from config/runtime_targeted.yaml. Compare load magnitudes and daily shapes at hourly and seasonal granularity between generated and CER profiles to compute consistency metrics. Write the results to cond_cer_evaluation. in the run outputs. Validation succeeds if cond_cer_evaluation. exists, is valid JSON containing the computed consistency comparison, and the reported metrics reproduce the framework-vs-CER consistency comparison described in the paper.

## Related tools

- **WattCouncil** (Source of the generated Stage 3 hourly consumption profiles and the convert_outputs.py script used to convert JSON outputs to tabular form) — github.com/Singularity-AI-Lab/wattcouncil
- **Python** (Runtime environment (Python 3.11 conda env llm_council) for running convert_outputs.py and the consistency evaluation) — github.com/Singularity-AI-Lab/wattcouncil

## Examples

```
python WattCouncil/convert_outputs.py outputs/run_20260206_090431/stage3_consumption_*. && python evaluate_consistency.py --generated outputs/run_20260206_090431 --cer cer_dataset --out cond_cer_evaluation.
```

## Evaluation signals

- cond_cer_evaluation. exists in the run outputs directory and parses as valid JSON
- The JSON contains the computed consistency comparison between generated and CER profiles
- Reported metrics reproduce the framework-vs-CER consistency comparison described in the paper
- Generated profiles were aligned only to CER segments matching the targeted combos (e.g. [1, 5, 1], [2, 4, 2], [3, 3, 2])
- Hourly and seasonal comparisons cover both load magnitudes and daily shapes

## Limitations

- Comparison requires the targeted-generation mode (config/runtime_targeted.yaml) whose representative_combos match CER dataset segments; profiles from non-targeted runs may not align cleanly
- Consistency is assessed against one reference dataset (CER, Irish smart-meter trial); generalization to other countries/climates is not established by this evaluation
- Validation depends on the generated JSON outputs being convertible by convert_outputs.py; corrupted or missing run outputs will break the comparison

## Evidence

- [abstract] CER evaluation scope: "We evaluate the generated profiles against the detailed CER dataset, which contains over a year of load measurements for 4232 households together with survey-based socio-economic information."
- [methods] workflow definition: [evidence span withheld — anchor status: not_found]
- [methods] validation criterion: [evidence span withheld — anchor status: not_found]
- [readme] targeted combos in README: "Targeted batches to match CER dataset."
- [readme] representative combos: "representative_combos:
      - [1, 5, 1]  # Small (1 person), Bungalow, alone
      - [2, 4, 2]  # Small, terraced, adults_only
      - [3, 3, 2]  # Medium, detached, adults_only"
