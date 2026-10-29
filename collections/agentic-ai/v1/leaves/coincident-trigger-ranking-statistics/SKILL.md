---
name: coincident-trigger-ranking-statistics
description: Use when a higher-mode GW search admits excessive background triggers
  under a naive summed-HM cut and you need to compare or deploy a marginalized ranking
  statistic over physically allowed harmonic SNR combinations.
when_to_use_negative:
- You are running a quadrupole-only (2,2) search with no higher-mode SNR time series
  — there is nothing to marginalize over
- You need parameter-estimation posteriors rather than trigger collection/ranking
  (cogwheel is the PE tool, not this ranking skill)
- The code path requires an arbitrary number of ell=m modes or detectors — the pipeline
  currently hard-codes three modes (22,33,44) and Hanford+Livingston
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: IAS-HM search pipeline (gwIAS-HM)
  role: Provides the triggering and coincidence modules that produce mode-by-mode
    SNR time series and background candidates to be ranked
  repo: https://github.com/JayWadekar/gwIAS-HM
- name: numpy/scipy
  role: Implement the matched filtering of each harmonic template and the SNR time-series
    combination in the ranking statistics
- name: matplotlib
  role: Render the background_trigger_comparison.png comparing trigger counts across
    the four statistics
- name: lal / lalsimulation
  role: Generate the (2,2), (3,3), (4,4) harmonic templates used in matched filtering
- name: band eraser
  role: Data-conditioning step that excises narrow time-varying noisy bands before
    filtering, reducing loud non-Gaussian background entering the trigger set
  repo: https://github.com/JayWadekar/gwIAS-HM
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1103/physrevd.110.044063
    title: New search pipeline for gravitational waves with higher-order modes using
      mode-by-mode filtering
  source_content_hash: sha256:5224204d607a249bfb8e0239f52367aadcbb8d6e1db122098b66f05b468934f7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/JayWadekar/gwIAS-HM
  tools:
  - IAS-HM search pipeline
  - IAS-HM search pipeline (gwIAS-HM)
  - band eraser
  - lal
  - lal / lalsimulation
  - matplotlib
  - numpy
  - numpy/scipy
  - scipy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# coincident-trigger-ranking-statistics

## Summary

A method from the IAS-HM pipeline for evaluating multiple trigger-ranking statistics — (2,2)-only SNR, naive summed-HM SNR (|ρ⊥22|² + |ρ⊥33|² + |ρ⊥44|²), distance-to-physical-injection, and a marginalized statistic over harmonic SNR time series — on collected background triggers, to reduce trigger load without losing sensitivity to higher-mode signals.

## When to use

Use when a higher-mode GW search admits excessive background triggers under a naive summed-HM cut and you need to compare or deploy a marginalized ranking statistic over physically allowed harmonic SNR combinations.

## When NOT to use

- You are running a quadrupole-only (2,2) search with no higher-mode SNR time series — there is nothing to marginalize over
- You need parameter-estimation posteriors rather than trigger collection/ranking (cogwheel is the PE tool, not this ranking skill)
- The code path requires an arbitrary number of ell=m modes or detectors — the pipeline currently hard-codes three modes (22,33,44) and Hanford+Livingston

## Inputs

- Conditioned GW strain chunk (e.g. a public O3a ~5 hour chunk from gw-openscience.org)
- Mode-by-mode SNR time series for (2,2), (3,3), (4,4) harmonics from the triggering module
- Collected coincident/background candidates with veto tests and coherent statistic from the coincidence module

## Outputs

- Per-statistic background trigger counts at fixed thresholds
- Comparison plot (background_trigger_comparison.png) showing trigger load for each ranking statistic
- Ranked candidate list with false alarm rates

## How to apply

Matched-filter the conditioned strain separately with (2,2), (3,3) and (4,4) harmonic templates, then run the coincidence module to collect coincident/background candidates, perform veto tests and compute the coherent statistic. On those background triggers, evaluate four ranking statistics side by side: (2,2)-only SNR, naive summed-HM SNR, distance-to-physical-injection, and the marginalized statistic that combines the harmonic SNR time series using an empirical template-dependent background model to account for non-Gaussian transients, triggering only on physically allowed mode combinations. Count background triggers at fixed thresholds per statistic and render the comparison (e.g. background_trigger_comparison.png with matplotlib). The rationale is that the summed-HM statistic admits many more background triggers than a (2,2)-only cut, while the marginalized statistic preserves the HM pipeline's sensitivity at much lower search cost.

## Related tools

- **IAS-HM search pipeline (gwIAS-HM)** (Provides the triggering and coincidence modules that produce mode-by-mode SNR time series and background candidates to be ranked) — https://github.com/JayWadekar/gwIAS-HM
- **numpy/scipy** (Implement the matched filtering of each harmonic template and the SNR time-series combination in the ranking statistics)
- **matplotlib** (Render the background_trigger_comparison.png comparing trigger counts across the four statistics)
- **lal / lalsimulation** (Generate the (2,2), (3,3), (4,4) harmonic templates used in matched filtering)
- **band eraser** (Data-conditioning step that excises narrow time-varying noisy bands before filtering, reducing loud non-Gaussian background entering the trigger set) — https://github.com/JayWadekar/gwIAS-HM

## Examples

```
jupyter notebook Pipeline_modules/Collecting_triggers_coherent_marginalized_score.ipynb
```

## Evaluation signals

- Background trigger counts under the marginalized statistic are significantly lower than under the naive summed-HM statistic at matched thresholds
- The reproduced trigger-count trend qualitatively matches the paper's Fig. 2: marginalized statistic suppresses loud non-Gaussian background relative to summed-HM while retaining physically allowed higher-mode combinations
- Sensitivity of the HM pipeline is preserved (marginalized statistic does not discard physically allowed mode combinations)
- Each collected trigger passes veto tests and carries a computed coherent statistic from the coincidence module
- All four statistics are evaluated on the identical set of collected background triggers for a fair comparison

## Limitations

- Three modes (22,33,44) are hard-coded in places; the ranking cannot yet be generalized to an arbitrary number of ell=m modes or a quadrupole-only comparison without code changes
- Only Hanford and Livingston detectors are hardcoded, restricting multi-detector coherent-statistic computation
- Candidates that fail veto tests are currently removed entirely rather than re-included with a penalty, which can affect background statistics
- An empirical template-dependent background model is used for non-Gaussian transients; the ML classifier to augment the detection statistic is still planned

## Evidence

- [abstract] Marginalized ranking statistic preserves sensitivity and reduces triggers: "We develop computationally cheap trigger-ranking statistics to optimally combine the different signal-to-noise ratios (SNR) time series from different harmonics, which ensure only physically allowed"
- [abstract] Non-Gaussian background model: "We use an empirical template-dependent background model in our ranking statistic to account for non-Gaussian transients."
- [methods] Coincidence module duties: "3. Trig_Coin_test_with_injection.ipynb: Using individual injections, we test the triggering module (for data pre-processing and matched filtering) and coincidence module"
- [methods] Veto and coherent statistic step: "(for collecting coincident/background candidates, performing veto tests and computing coherent statistic)"
- [readme] README notebook on collecting triggers with the marginalized score: "1. Collecting triggers based on the coherent marginalized score
2. Using the band eraser"
