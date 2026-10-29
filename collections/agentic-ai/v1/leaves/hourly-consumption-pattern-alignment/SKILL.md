---
name: hourly-consumption-pattern-alignment
description: 'Use when validating LLM-generated household electricity profiles against
  a real smart-meter dataset: you have stage3_consumption_* outputs and CER segments
  defined by (num_people, house_type, composition) combinations to match.'
when_to_use_negative:
- You only need raw synthetic generation without external validation — the 3-stage
  WattCouncil pipeline (Family, Weather, Consumption) suffices and CER alignment is
  unnecessary.
- Your reference dataset does not provide household segments compatible with the (num_people,
  house_type, composition) combinations WattCouncil targets, so segment matching cannot
  be performed.
- You are running ablation studies of framework consistency rather than evaluating
  generated profiles against the real-world CER data.
tools:
- name: WattCouncil
  role: Provides the Stage 3 generated consumption profiles, the convert_outputs.py
    conversion script, and the targeted configuration that matches CER household segments
  repo: github.com/Singularity-AI-Lab/wattcouncil
- name: Python 3.11
  role: Runtime for the pipeline (conda env llm_council) and for running convert_outputs.py
    and the consistency evaluation scripts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1145/3744256.3812595
    title: 'WattCouncil: Context-Aware Household Energy Scenario Generation With Governed
      LLMs'
  source_content_hash: sha256:da7ae79be6b3e48ff37efd46d3f50408062bffb6dfee18920d6bbf480b0217c4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - Python 3.11
  - WattCouncil
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# hourly-consumption-pattern-alignment

## Summary

Aligns WattCouncil Stage 3 generated hourly consumption profiles with the CER reference dataset (4232 households, over a year of smart-meter load measurements) by matching household segments, so that hourly and seasonal consumption patterns can be compared and consistency metrics computed.

## When to use

Use when validating LLM-generated household electricity profiles against a real smart-meter dataset: you have stage3_consumption_* outputs and CER segments defined by (num_people, house_type, composition) combinations to match.

## When NOT to use

- You only need raw synthetic generation without external validation — the 3-stage WattCouncil pipeline (Family, Weather, Consumption) suffices and CER alignment is unnecessary.
- Your reference dataset does not provide household segments compatible with the (num_people, house_type, composition) combinations WattCouncil targets, so segment matching cannot be performed.
- You are running ablation studies of framework consistency rather than evaluating generated profiles against the real-world CER data.

## Inputs

- WattCouncil Stage 3 generated hourly consumption outputs (stage3_consumption_*. JSON files in outputs/run_<timestamp>/)
- CER reference dataset: over a year of smart-meter load measurements for 4232 households with survey-based socio-economic information
- Targeted configuration (config/runtime_targeted.yaml) specifying representative_combos such as [1, 5, 1], [2, 4, 2], [3, 3, 2]

## Outputs

- cond_cer_evaluation. — valid JSON containing the computed consistency comparison between generated and CER profiles
- Tabular representation of the generated JSON consumption outputs (via convert_outputs.py)
- Hourly and seasonal pattern comparison metrics (load magnitudes, daily shapes)

## How to apply

Load the WattCouncil Stage 3 generated hourly consumption profiles (stage3_consumption_*.) from the timestamped outputs/run_* directory of a completed pipeline run, and load the CER dataset containing over a year of load measurements for 4232 households with survey-based socio-economic information. Convert the generated JSON consumption outputs into tabular form with WattCouncil's convert_outputs.py script. Align generated profiles to CER household segments matching the (num_people, house_type, composition) combinations used in targeted generation, e.g. [1, 5, 1], [2, 4, 2], [3, 3, 2] — run the pipeline with config/runtime_targeted.yaml so generated batches match the CER distribution. Compare hourly and seasonal patterns (load magnitudes, daily shapes) between generated and CER profiles and compute the consistency metrics, writing results to cond_cer_evaluation. as valid JSON.

## Related tools

- **WattCouncil** (Provides the Stage 3 generated consumption profiles, the convert_outputs.py conversion script, and the targeted configuration that matches CER household segments) — github.com/Singularity-AI-Lab/wattcouncil
- **Python 3.11** (Runtime for the pipeline (conda env llm_council) and for running convert_outputs.py and the consistency evaluation scripts)

## Examples

```
python main.py --config config/runtime_targeted.yaml --weather-config config/weather.yaml
```

## Evaluation signals

- cond_cer_evaluation. exists and is valid JSON containing the computed consistency comparison
- Reported metrics reproduce the framework-vs-CER consistency comparison described in the paper
- Generated profiles are matched to CER segments only for the targeted (num_people, house_type, composition) combinations, e.g. [1, 5, 1], [2, 4, 2], [3, 3, 2]
- Hourly and seasonal comparisons cover both load magnitudes and daily shapes for each aligned segment
- Generated JSON outputs were successfully converted to tabular form via convert_outputs.py before comparison

## Limitations

- The CER dataset is specific to Ireland (smart-meter data for 4232 households); generated profiles for other countries (e.g. United Arab Emirates) may not be directly transferable, hence only targeted segment matching is performed.
- Evaluation depends on a completed pipeline run — stage3_consumption_* outputs must exist in a timestamped outputs/run_* directory; partial or interrupted runs may lack the needed profiles.
- Comparison quality is limited by the few representative combinations selected in runtime_targeted.yaml (e.g. only [1,5,1], [2,4,2], [3,3,2] in the example), not the full CER household distribution.

## Evidence

- [abstract] CER evaluation against generated profiles: "We evaluate the generated profiles against the detailed CER dataset, which contains over a year of load measurements for 4232 households together with survey-based socio-economic information."
- [methods] Targeted generation matching CER: "Targeted batches to match CER dataset."
- [readme] Representative household combinations: "Representative combinations filtered by (num_people, house_type, composition)"
- [readme] Stage 3 consumption generation: "Generate hourly household energy consumption based on family and weather"
