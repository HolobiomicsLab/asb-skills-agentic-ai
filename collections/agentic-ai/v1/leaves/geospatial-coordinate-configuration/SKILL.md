---
name: geospatial-coordinate-configuration
description: Use when running WattCouncil's Stage 2 Weather Generation for a target
  country and you must supply capital coordinates (e.g. 24.4539/54.3773 for the UAE)
  and hemisphere classification to drive seasonal weather profiles.
when_to_use_negative:
- Stages 1 or 3 need to be configured — coordinates and hemisphere settings only affect
  Stage 2 Weather Generation
- The target country already has an accurate capitals entry and hemisphere classification
  in config/weather.yaml; in that case simply pass the existing weather config
- You only need TMY real meteorological data without location-specific tuning — although
  coordinates are still required, the LLM-based seasonal generation settings are irrelevant
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3813
tools:
- name: WattCouncil
  role: Framework whose Stage 2 Weather Generation consumes the geospatial configuration
    to produce seasonal weather profiles
  repo: github.com/Singularity-AI-Lab/wattcouncil
- name: TMY (Typical Meteorological Year) weather data
  role: Real meteorological data source processed for the configured capital location
    as an alternative to LLM-based weather generation
- name: Python 3.11
  role: Runtime environment (conda env 'llm_council') in which main.py is invoked
    with --weather-config
  repo: github.com/Singularity-AI-Lab/wattcouncil
- name: conda
  role: Environment manager used to create the llm_council environment that runs the
    pipeline
  repo: github.com/Singularity-AI-Lab/wattcouncil
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1145/3744256.3812595
    title: 'WattCouncil: Context-Aware Household Energy Scenario Generation With Governed
      LLMs'
  source_content_hash: sha256:0fb23c0c75538ca439d60a6cbfa37990835dd1b71b8d25e451f0826d3ac79a82
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepInfra API
  - Python
  - Python 3.11
  - Qwen/Qwen2.5-72B-Instruct
  - TMY (Typical Meteorological Year) weather data
  - WattCouncil
  - conda
  - google/gemini-2.5-flash
  - meta-llama/Llama-4-Maverick-17B
  - pip
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3813
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# geospatial-coordinate-configuration

## Summary

Configure the geospatial parameters (capital latitude/longitude and southern-hemisphere classification) in config/weather.yaml so WattCouncil's Stage 2 can generate seasonal hourly weather profiles for the correct location, either via LLM generation or TMY real meteorological data.

## When to use

Use when running WattCouncil's Stage 2 Weather Generation for a target country and you must supply capital coordinates (e.g. 24.4539/54.3773 for the UAE) and hemisphere classification to drive seasonal weather profiles.

## When NOT to use

- Stages 1 or 3 need to be configured — coordinates and hemisphere settings only affect Stage 2 Weather Generation
- The target country already has an accurate capitals entry and hemisphere classification in config/weather.yaml; in that case simply pass the existing weather config
- You only need TMY real meteorological data without location-specific tuning — although coordinates are still required, the LLM-based seasonal generation settings are irrelevant

## Inputs

- config/runtime.yaml specifying the pipeline country and year (e.g. United Arab Emirates, 2024)
- config/weather.yaml with 'capitals' latitude/longitude mapping and 'southern_hemisphere' country list
- Stage 1 family output (stage1_families_*) from the run directory

## Outputs

- stage2_hourly_* seasonal hourly weather profile saved in the timestamped run output folder
- raw/ checkpoint files for smart resume

## How to apply

Edit config/weather.yaml to add an entry under 'capitals' for the pipeline country defined in config/runtime.yaml, giving the capital's latitude and longitude (e.g. 'United Arab Emirates': latitude 24.4539, longitude 54.3773). If the country lies in the southern hemisphere, add it to the 'southern_hemisphere' list (as done for Australia and New Zealand) so seasonal profiles are offset correctly. Pass the weather config at runtime with the --weather-config flag (e.g. 'python main.py --config config/runtime.yaml --weather-config config/weather.yaml') so Stage 2 generates seasonal hourly weather profiles using either the LLM council or TMY real data for that location. Validate the resulting profile against the JSON schema schemas/stage2_hourly_weather and save it as stage2_hourly_* in the timestamped run folder, checkpointing each item to raw/ for smart resume.

## Related tools

- **WattCouncil** (Framework whose Stage 2 Weather Generation consumes the geospatial configuration to produce seasonal weather profiles) — github.com/Singularity-AI-Lab/wattcouncil
- **TMY (Typical Meteorological Year) weather data** (Real meteorological data source processed for the configured capital location as an alternative to LLM-based weather generation)
- **Python 3.11** (Runtime environment (conda env 'llm_council') in which main.py is invoked with --weather-config) — github.com/Singularity-AI-Lab/wattcouncil
- **conda** (Environment manager used to create the llm_council environment that runs the pipeline) — github.com/Singularity-AI-Lab/wattcouncil

## Examples

```
python main.py --config config/runtime.yaml --weather-config config/weather.yaml
```

## Evaluation signals

- config/weather.yaml contains a 'capitals' entry whose key exactly matches the 'country' value in config/runtime.yaml (e.g. 'United Arab Emirates')
- Southern-hemisphere countries appear in the 'southern_hemisphere' list (e.g. Australia, New Zealand) so seasons are offset
- The generated profile validates against schemas/stage2_hourly_weather; failed candidates route through REGENERATE_FULL/partial-regeneration
- stage2_hourly_* file exists in the timestamped run output folder and every item is checkpointed to raw/
- CLI run includes the --weather-config flag pointing at config/weather.yaml

## Limitations

- Coordinates are only defined for capitals listed in config/weather.yaml; a country absent from the mapping must be added manually before Stage 2 can run
- TMY-based generation depends on meteorological data availability for the configured capital location, whereas LLM-based generation depends on DeepInfra API access and model correctness
- The hemisphere classification must be maintained manually; a missing entry for a southern-hemisphere country would misalign seasonal profiles

## Evidence

- [methods] Weather config: capital coordinates and hemisphere list: [evidence span withheld — anchor status: not_found]
- [readme] README weather.yaml structure: "capitals:
  "United Arab Emirates":
    latitude: 24.4539
    longitude: 54.3773"
- [readme] README southern hemisphere list: "southern_hemisphere:
  - "Australia"
  - "New Zealand""
- [methods] Stage 2 description: "Generate seasonal weather profiles (using **LLM** or **TMY** real data)"
- [readme] README CLI usage with weather config: "python main.py --config config/runtime.yaml --weather-config config/weather.yaml"
