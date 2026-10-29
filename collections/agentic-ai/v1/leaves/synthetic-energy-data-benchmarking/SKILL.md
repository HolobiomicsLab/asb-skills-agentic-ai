---
name: synthetic-energy-data-benchmarking
description: Use when a completed WattCouncil pipeline run has produced stage3_consumption_*
  outputs and the goal is to quantify how consistently the synthetic profiles match
  real smart-meter data such as the CER dataset.
when_to_use_negative:
- The pipeline run has not completed or stage3_consumption_* outputs are absent —
  there is nothing to evaluate against CER
- The research goal is measuring LLM council internal consistency via ablation studies,
  rather than external validation against a real dataset
- No high-resolution reference dataset (e.g. CER) with matching household segments
  is available for alignment
tools:
- name: WattCouncil
  role: Framework whose Stage 3 consumption outputs (stage3_consumption_*) and convert_outputs.py
    script are the evaluation inputs; runtime_targeted.yaml enables targeted batches
    matched to the CER dataset
  repo: github.com/Singularity-AI-Lab/wattcouncil
- name: Python 3.11
  role: Runtime environment (conda env llm_council) for running the pipeline, convert_outputs.py,
    and writing cond_cer_evaluation.
- name: CER dataset
  role: 'Real-world high-resolution benchmark: >1 year of load measurements for 4232
    households with survey-based socio-economic information used as the ground truth
    for consistency metrics'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1145/3744256.3812595
    title: 'WattCouncil: Context-Aware Household Energy Scenario Generation With Governed
      LLMs'
  source_content_hash: sha256:6a8afb1b9398dee077d847593ab82e0d84070a23dd3cf6c8f04b246586da0fec
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CER dataset
  - Python
  - Python 3.11
  - WattCouncil
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# synthetic-energy-data-benchmarking

## Summary

Validates LLM-generated synthetic household electricity consumption profiles (WattCouncil Stage 3 hourly outputs) against a real-world high-resolution reference dataset (CER, >1 year of load measurements for 4232 households with survey-based socio-economic information) by computing consistency metrics on hourly and seasonal consumption patterns.

## When to use

Use when a completed WattCouncil pipeline run has produced stage3_consumption_* outputs and the goal is to quantify how consistently the synthetic profiles match real smart-meter data such as the CER dataset.

## When NOT to use

- The pipeline run has not completed or stage3_consumption_* outputs are absent — there is nothing to evaluate against CER
- The research goal is measuring LLM council internal consistency via ablation studies, rather than external validation against a real dataset
- No high-resolution reference dataset (e.g. CER) with matching household segments is available for alignment

## Inputs

- WattCouncil Stage 3 generated hourly consumption profiles (stage3_consumption_* files from a timestamped run outputs directory, JSON format)
- CER reference dataset: over a year of smart-meter load measurements for 4232 households with survey-based socio-economic information
- Targeted-generation configuration with representative combinations, e.g. [1, 5, 1], [2, 4, 2], [3, 3, 2] (from config/runtime_targeted.yaml)

## Outputs

- cond_cer_evaluation. — valid JSON containing the computed consistency comparison between generated profiles and the CER dataset (hourly and seasonal pattern metrics)
- Tabular version of generated JSON consumption outputs produced by convert_outputs.py

## How to apply

Load the WattCouncil Stage 3 generated hourly consumption profiles (stage3_consumption_*) from the timestamped run outputs directory of a completed pipeline run, then convert the generated JSON consumption outputs into tabular form using WattCouncil's convert_outputs.py script. Load the CER reference dataset and align generated profiles with CER household segments by matching the (num_people, house_type, composition) combinations used in targeted generation — e.g. [1, 5, 1] (1 person, bungalow, alone), [2, 4, 2] (small, terraced, adults_only), [3, 3, 2] (medium, detached, adults_only) as configured in config/runtime_targeted.yaml. Compare hourly and seasonal consumption patterns (load magnitudes, daily shapes) between generated and CER profiles to compute the consistency metrics. Write the results to cond_cer_evaluation., which must be valid JSON containing the computed consistency comparison. For the underlying generation, run the pipeline in targeted mode (e.g. python main.py --config config/runtime_targeted.yaml --weather-config config/weather.yaml) so that representative combos match the CER segments.

## Related tools

- **WattCouncil** (Framework whose Stage 3 consumption outputs (stage3_consumption_*) and convert_outputs.py script are the evaluation inputs; runtime_targeted.yaml enables targeted batches matched to the CER dataset) — github.com/Singularity-AI-Lab/wattcouncil
- **Python 3.11** (Runtime environment (conda env llm_council) for running the pipeline, convert_outputs.py, and writing cond_cer_evaluation.)
- **CER dataset** (Real-world high-resolution benchmark: >1 year of load measurements for 4232 households with survey-based socio-economic information used as the ground truth for consistency metrics)

## Examples

```
python main.py --config config/runtime_targeted.yaml --weather-config config/weather.yaml
```

## Evaluation signals

- cond_cer_evaluation. exists in the run outputs and is valid JSON
- The file contains the computed consistency comparison (hourly and seasonal load magnitudes and daily shapes) between generated and CER profiles
- The reported metrics reproduce the framework-vs-CER consistency comparison described in the paper
- Generated profiles are aligned to CER household segments only via the representative combos used in targeted generation (e.g. [1, 5, 1], [2, 4, 2], [3, 3, 2])
- JSON consumption outputs were converted to tabular form with convert_outputs.py before comparison

## Limitations

- Alignment depends on targeted generation: only household segments matching the configured representative combinations (num_people, house_type, composition) can be compared against CER
- Generated profiles reflect the configured country/year (e.g. United Arab Emirates, 2024); cross-country or cross-climate transfer to the CER (Ireland) context may confound the comparison
- Validation succeeds only if cond_cer_evaluation. is valid JSON with the computed comparison — malformed or missing outputs fail the check

## Evidence

- [abstract] CER evaluation against generated profiles: "We evaluate the generated profiles against the detailed CER dataset, which contains over a year of load measurements for 4232 households together with survey-based socio-economic information."
- [methods] Targeted generation config for CER alignment: "Targeted batches to match CER dataset."
- [readme] Runtime command producing the run outputs to benchmark: "python main.py --config config/runtime_targeted.yaml --weather-config config/weather.yaml"
- [readme] Representative household combinations used for segment matching: "Representative combinations filtered by (num_people, house_type, composition)"
