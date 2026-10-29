---
name: macronutrient-constraint-specification
description: Use when you must evaluate a personalized meal-planning system against
  realistic user dietary inputs but cannot run a human user study, and need reproducible
  synthetic profiles with caloric targets.
when_to_use_negative:
- You have real user dietary data from an actual human study — use the collected profiles
  instead of simulated ones.
- Your application requires calorie goals outside the reasonable 1500-3500 kcal range
  (e.g. clinical or athletic nutrition planning).
tools:
- name: NutriGen
  role: LLM framework that consumes the simulated user profiles as dietary constraints
    when generating personalized meal plans
  repo: https://github.com/SamanKhamesian/NutriGen
- name: pandas
  role: Loads the USDA nutrition database table into a DataFrame for meal sampling
- name: NumPy
  role: Random-sampling routines (with fixed seed) used to select 200 meals and draw
    calorie goals uniformly from 1500-3500 kcal
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1109/embc58623.2025.11253879
    title: 'NutriGen: Personalized Meal Plan Generator Leveraging Large Language Models
      to Enhance Dietary and Nutritional Adherence'
  source_content_hash: sha256:5a924e34c77c5b655079eed94ccc3c2d486a9579315290aaa8a0994f167cd695
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/SamanKhamesian/NutriGen
  tools:
  - NumPy
  - NutriGen
  - Python
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# macronutrient-constraint-specification

## Summary

Method for generating simulated user dietary profiles without a human study: compose diverse daily meal-intake profiles from randomly sampled USDA meals and assign each a randomized daily calorie goal drawn uniformly from 1500-3500 kcal plus protein and sugar intake targets, serialized as JSON for evaluating LLM-based meal-plan generators like NutriGen.

## When to use

Use when you must evaluate a personalized meal-planning system against realistic user dietary inputs but cannot run a human user study, and need reproducible synthetic profiles with caloric targets.

## When NOT to use

- You have real user dietary data from an actual human study — use the collected profiles instead of simulated ones.
- Your application requires calorie goals outside the reasonable 1500-3500 kcal range (e.g. clinical or athletic nutrition planning).

## Inputs

- USDA nutrition database table (loaded into a pandas DataFrame)
- Random seed for NumPy sampling (for reproducibility)
- Uniform sampling range for daily calorie goals: 1500-3500 kcal

## Outputs

- user_profiles. containing exactly 10 profile records with calorie goal, protein target, sugar target, and meal composition per profile

## How to apply

Load the USDA nutrition database table into a pandas DataFrame and randomly select 200 meal entries with NumPy's random sampling routines using a fixed seed for reproducibility. Compose ten diverse daily food-intake profiles by drawing meals from the sampled pool for each profile. Assign each profile a randomized daily calorie goal drawn uniformly from the 1500-3500 kcal range, together with randomized protein and sugar intake targets. Serialize the ten profiles (calorie goal, protein target, sugar target, and meal composition per profile) to user_profiles.. These target values then serve as the 'specified target values' against which the average total calories of generated meal plans are compared to compute each model's percentage error.

## Related tools

- **NutriGen** (LLM framework that consumes the simulated user profiles as dietary constraints when generating personalized meal plans) — https://github.com/SamanKhamesian/NutriGen
- **pandas** (Loads the USDA nutrition database table into a DataFrame for meal sampling)
- **NumPy** (Random-sampling routines (with fixed seed) used to select 200 meals and draw calorie goals uniformly from 1500-3500 kcal)

## Examples

```
import numpy as np; meals = df.sample(n=200, random_state=42); profiles = [{'calorie_goal': int(np.random.uniform(1500, 3500)), 'meals': meals.sample(n=5).to_dict('records')} for _ in range(10)]
```

## Evaluation signals

- user_profiles. exists and parses as valid JSON
- The file contains exactly 10 profile records
- Every calorie goal falls within the 1500-3500 kcal range
- Re-running with the fixed seed reproduces the same 200-meal sample and the same ten profiles
- Downstream evaluation succeeds: average total calories of each model's generated meal plans can be compared against the specified target values per input

## Limitations

- Profiles are simulated, not drawn from real users; the 1500-3500 kcal 'reasonable range' and randomized protein/sugar targets may not reflect true dietary distributions.
- Ten profiles is a small evaluation set, so per-model error metrics (e.g. Llama 3.1 8B at 1.55% vs DeepSeek V3 at 10.45% MAE) rest on limited inputs.
- Reproducibility depends on the fixed seed; changing the seed changes the sampled meal pool and all targets.

## Evidence

- [other] The profile-simulation mechanism composes ten diverse daily food intake profiles from 200 randomly selected USDA meals, with calorie goals and intervention durations randomly drawn from a reasonable range of 1500-3500 kcal.: [evidence span withheld — anchor status: not_found]
- [methods] Randomly select 200 meal entries from the dataset using NumPy's random sampling routines with a fixed seed for reproducibility.: [evidence span withheld — anchor status: not_found]
- [methods] Assign each profile a randomized daily calorie goal drawn uniformly from the 1500-3500 kcal range, together with randomized protein and sugar intake targets.: [evidence span withheld — anchor status: not_found]
