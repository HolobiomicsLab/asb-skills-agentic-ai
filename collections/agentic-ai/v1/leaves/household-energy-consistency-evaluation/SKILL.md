---
name: household-energy-consistency-evaluation
description: Use when synthetic LLM-generated hourly household electricity profiles
  must be validated against a real high-resolution consumption reference, by aligning
  generated households to matching segments (household size, house type, composition)
  and comparing hourly/seasonal load patterns.
when_to_use_negative:
- No real-world reference dataset is available — the evaluation is defined specifically
  as a framework-vs-CER comparison
- The pipeline run is incomplete or Stage 3 consumption outputs (stage3_consumption_*)
  have not been generated
- You need to evaluate stages other than Stage 3 consumption (e.g., Stage 1 family
  compositions or Stage 2 weather profiles are not validated against CER here)
edam_operation: http://edamontology.org/operation_2428
tools:
- name: WattCouncil
  role: Framework whose Stage 3 hourly consumption profiles are the subject of the
    CER consistency evaluation; provides convert_outputs.py for JSON-to-tabular conversion
    and the targeted-generation config matching CER segments
  repo: github.com/Singularity-AI-Lab/wattcouncil
- name: Python
  role: Execution environment (Python 3.11 conda env) for running convert_outputs.py,
    aligning generated profiles with CER segments, and writing the cond_cer_evaluation
    JSON
- name: CER dataset
  role: 'Real-world reference data: over a year of load measurements for 4232 households
    with survey-based socio-economic information used as the consistency benchmark'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1145/3744256.3812595
    title: 'WattCouncil: Context-Aware Household Energy Scenario Generation With Governed
      LLMs'
  source_content_hash: sha256:97ee6e2f4406599cf34e19d701eb2e26c7a793d20f0cff2037c0c70fb279b0cd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CER dataset
  - Python
  - WattCouncil
  edam_operation: http://edamontology.org/operation_2428
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# household-energy-consistency-evaluation

## Summary

Validates LLM-generated household electricity profiles (WattCouncil Stage 3 hourly consumption) against the real-world CER smart-meter dataset (>1 year of load measurements for 4232 households with survey-based socio-economic information), producing a JSON consistency comparison (cond_cer_evaluation).

## When to use

Use when synthetic LLM-generated hourly household electricity profiles must be validated against a real high-resolution consumption reference, by aligning generated households to matching segments (household size, house type, composition) and comparing hourly/seasonal load patterns.

## When NOT to use

- No real-world reference dataset is available — the evaluation is defined specifically as a framework-vs-CER comparison
- The pipeline run is incomplete or Stage 3 consumption outputs (stage3_consumption_*) have not been generated
- You need to evaluate stages other than Stage 3 consumption (e.g., Stage 1 family compositions or Stage 2 weather profiles are not validated against CER here)

## Inputs

- WattCouncil Stage 3 generated hourly consumption profiles (stage3_consumption_* files from a timestamped outputs/run_* directory)
- CER reference dataset (smart-meter load measurements for 4232 households over >1 year, with survey-based socio-economic information)
- Targeted generation config (config/runtime_targeted.yaml) defining representative_combos such as [1, 5, 1], [2, 4, 2], [3, 3, 2]

## Outputs

- cond_cer_evaluation (valid JSON file containing the computed generated-vs-CER consistency comparison and reported metrics)

## How to apply

Run the WattCouncil pipeline to completion (e.g., with config/runtime_targeted.yaml enabling targeted_generation so representative combos like [1,5,1], [2,4,2], [3,3,2] match CER household segments). Load Stage 3 outputs (stage3_consumption_*) from the timestamped run directory under outputs/, and convert the generated JSON consumption files to tabular form with WattCouncil's convert_outputs.py script. Load the CER reference dataset, align generated profiles to CER households sharing the same (num_people, house_type, composition) combination, then compare hourly and seasonal patterns—load magnitudes and daily shapes—to compute consistency metrics. Write the comparison to cond_cer_evaluation as valid JSON. Rationale: the CER dataset provides over a year of per-household smart-meter load measurements plus socio-economic survey data, giving a segment-level ground truth for judging whether generated daily routines are realistic.

## Related tools

- **WattCouncil** (Framework whose Stage 3 hourly consumption profiles are the subject of the CER consistency evaluation; provides convert_outputs.py for JSON-to-tabular conversion and the targeted-generation config matching CER segments) — github.com/Singularity-AI-Lab/wattcouncil
- **Python** (Execution environment (Python 3.11 conda env) for running convert_outputs.py, aligning generated profiles with CER segments, and writing the cond_cer_evaluation JSON)
- **CER dataset** (Real-world reference data: over a year of load measurements for 4232 households with survey-based socio-economic information used as the consistency benchmark)

## Examples

```
python main.py --config config/runtime_targeted.yaml --weather-config config/weather.yaml
```

## Evaluation signals

- cond_cer_evaluation file exists in the run outputs and parses as valid JSON
- cond_cer_evaluation contains the computed consistency comparison between generated profiles and CER, not just metadata
- Reported metrics reproduce the framework-vs-CER consistency comparison described in the paper
- Generated households were aligned to CER segments using the same (num_people, house_type, composition) combinations as the targeted batches (e.g., [1, 5, 1], [2, 4, 2], [3, 3, 2])
- Converted tabular outputs contain complete hourly profiles for all generated households before comparison

## Limitations

- Validity depends on the CER dataset access (restricted high-resolution smart-meter data); the paper motivates the work by limited access to such data
- Segment matching is limited to the representative combinations configured in runtime_targeted.yaml; unconfigured combos (e.g., [3, 2, 3], [4, 3, 3]) are commented out and thus not evaluated
- The comparison requires a completed pipeline run with checkpointed Stage 3 outputs; partial or corrupted runs yield incomplete alignment

## Evidence

- [abstract] The WattCouncil-generated household electricity profiles are evaluated against the detailed CER dataset, which contains over a year of load measurements for 4232 households together with survey-based socio-economic information.: "We evaluate the generated profiles against the detailed CER dataset, which contains over a year of load measurements for 4232 households together with survey-based socio-economic information."
- [methods] Workflow: load Stage 3 outputs, align with CER segments, compare hourly/seasonal patterns, write cond_cer_evaluation.: [evidence span withheld — anchor status: not_found]
- [methods] Validation: cond_cer_evaluation must exist as valid JSON reproducing the paper's consistency comparison.: [evidence span withheld — anchor status: not_found]
- [readme] README: targeted config for matching CER dataset.: "Targeted batches to match CER dataset."
- [readme] README: convert generated JSON outputs via convert_outputs.py.: [evidence span withheld — anchor status: not_found]
