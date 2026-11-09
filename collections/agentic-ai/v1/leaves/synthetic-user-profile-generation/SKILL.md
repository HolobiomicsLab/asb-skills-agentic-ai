---
name: synthetic-user-profile-generation
description: Use when evaluating a personalized-recommendation or meal-planning system
  but no human study data exist, and you need reproducible synthetic user profiles
  with randomized goals drawn from a plausible range (e.g., 1500-3500 kcal).
when_to_use_negative:
- Real user dietary data or a human user study is available — measured profiles are
  preferable to simulated ones
- The downstream evaluation requires profiles outside the plausible adult daily-intake
  range (e.g., clinical or pediatric targets below 1500 or above 3500 kcal)
- The evaluation metric depends on user behavior or adherence dynamics rather than
  static intake targets and meal compositions
edam_operation: http://edamontology.org/operation_0438
tools:
- name: Python 3.10
  role: Implementation language for the profile-simulation and evaluation pipeline
  repo: github.com/SamanKhamesian/NutriGen
- name: pandas
  role: Loads the USDA nutrition database table into a DataFrame for meal sampling
  repo: github.com/SamanKhamesian/NutriGen
- name: NumPy
  role: Random sampling of 200 meals with a fixed seed and uniform draws of calorie
    goals from 1500-3500 kcal
  repo: github.com/SamanKhamesian/NutriGen
- name: NutriGen
  role: Downstream LLM framework whose evaluation consumes the synthetic profiles
    and generated meal plans
  repo: github.com/SamanKhamesian/NutriGen
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1109/embc58623.2025.11253879
    title: 'NutriGen: Personalized Meal Plan Generator Leveraging Large Language Models
      to Enhance Dietary and Nutritional Adherence'
  source_content_hash: sha256:a73ffc711ec5d964b2b1c07a42af75d3e19950c5fa9505c439aa046e80cc752a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NumPy
  - NutriGen
  - Python
  - Python 3.10
  - pandas
  edam_operation: http://edamontology.org/operation_0438
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# synthetic-user-profile-generation

## Summary

A simulation skill for composing realistic synthetic user inputs (e.g., ten daily dietary profiles sampled from a USDA meal pool with randomized calorie/protein/sugar targets) so that systems like NutriGen can be evaluated without running a human user study.

## When to use

Use when evaluating a personalized-recommendation or meal-planning system but no human study data exist, and you need reproducible synthetic user profiles with randomized goals drawn from a plausible range (e.g., 1500-3500 kcal).

## When NOT to use

- Real user dietary data or a human user study is available — measured profiles are preferable to simulated ones
- The downstream evaluation requires profiles outside the plausible adult daily-intake range (e.g., clinical or pediatric targets below 1500 or above 3500 kcal)
- The evaluation metric depends on user behavior or adherence dynamics rather than static intake targets and meal compositions

## Inputs

- USDA nutrition database table (meal/food entries with nutritional values) loadable as a pandas DataFrame
- Random seed value for reproducible NumPy sampling
- Sampling parameters: pool size (200 meals), number of profiles (10), calorie goal range (1500-3500 kcal)

## Outputs

- user_profiles.: 10 profile records, each with calorie goal, protein target, sugar target, and meal composition
- Reproducible sampling code (pandas/NumPy with fixed seed)

## How to apply

Load the USDA nutrition database table into a pandas DataFrame, then use NumPy random sampling with a fixed seed to select a pool of 200 meal entries for reproducibility. Compose ten diverse daily food-intake profiles by drawing meals from this sampled pool for each profile, so profiles vary in meal composition. Assign each profile a randomized daily calorie goal drawn uniformly from the 1500-3500 kcal range, together with randomized protein and sugar intake targets, mimicking the diversity of real users without recruiting them. Serialize all ten profiles (calorie goal, protein target, sugar target, and per-profile meal composition) to user_profiles., which downstream model evaluation (e.g., comparing generated meal-plan calories against each profile's target) consumes as input. Validate the artifact before use: the file must parse as JSON, contain exactly 10 profile records, and every calorie goal must fall within the 1500-3500 kcal range.

## Related tools

- **Python 3.10** (Implementation language for the profile-simulation and evaluation pipeline) — github.com/SamanKhamesian/NutriGen
- **pandas** (Loads the USDA nutrition database table into a DataFrame for meal sampling) — github.com/SamanKhamesian/NutriGen
- **NumPy** (Random sampling of 200 meals with a fixed seed and uniform draws of calorie goals from 1500-3500 kcal) — github.com/SamanKhamesian/NutriGen
- **NutriGen** (Downstream LLM framework whose evaluation consumes the synthetic profiles and generated meal plans) — github.com/SamanKhamesian/NutriGen

## Examples

```
import pandas as pd, numpy as np; meals = pd.read_csv('usda_nutrition.csv').sample(n=200, random_state=42); profiles = [{'calorie_goal': int(g), 'meals': list(meals.sample(3)['name'])} for g in np.random.uniform(1500, 3500, 10)]; import ; .dump(profiles, open('user_profiles.', 'w'))
```

## Evaluation signals

- user_profiles. exists and parses as valid JSON
- The parsed file contains exactly 10 profile records
- Every profile's calorie goal lies within the 1500-3500 kcal range
- Each profile record includes calorie goal, protein target, sugar target, and meal composition fields
- Re-running the script with the same fixed seed reproduces the identical set of profiles (deterministic sampling)

## Limitations

- Profiles are simulated from randomly sampled USDA meals, so they capture meal-composition diversity but not real human dietary habits or adherence behavior
- Calorie goals are drawn uniformly from 1500-3500 kcal, which may not represent clinical, pediatric, or athletic populations outside that range
- Reproducibility depends on the fixed random seed; changing libraries or versions (e.g., numpy==2.2.2) can alter sampling behavior

## Evidence

- [methods] To install the necessary dependencies, run:

sh
pip install -r requirements.txt: "To install the necessary dependencies, run:

sh
pip install -r requirements.txt"
