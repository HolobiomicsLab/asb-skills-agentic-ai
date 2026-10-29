---
name: non-gaussian-tail-characterization
description: Use when timeslide background coincident triggers must be converted into
  per-bank empirical P(ρ²_sum|N) curves to test Gaussianity of the HM trigger-ranking
  statistic, especially for high-total-mass template banks.
when_to_use_negative:
- Input is raw strain time series or single-detector triggers rather than coincident/background
  candidates from the coincidence module — the empirical P(ρ²_sum|N) requires timeslide
  coincident triggers.
- The goal is candidate ranking/FAR computation or p_astro tables (Ranking_candidates.ipynb,
  p_astro+detection_tables.ipynb) — this skill only characterizes the background distribution
  that feeds such downstream steps.
- An analytic Gaussian background is acceptable (e.g. low-mass quadrupole-only banks
  where the empirical curve matches the chi-squared expectation) and a template-dependent
  empirical model is not needed.
edam_operation: http://edamontology.org/operation_2950
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: pandas
  role: load and tag timeslide coincident triggers with bank/subbank identifier and
    total mass
  repo: https://github.com/JayWadekar/gwIAS-HM
- name: numpy
  role: build density-normalized histograms of rho2_sum conditional on N per bank/template
    group
  repo: https://github.com/JayWadekar/gwIAS-HM
- name: scipy
  role: overlay the Gaussian (chi-squared-like) expectation for each (rho2_sum, N)
    pair and quantify tail excess
  repo: https://github.com/JayWadekar/gwIAS-HM
- name: matplotlib
  role: render the per-bank P(rho2_sum|N) curves as a Fig.-3-style multi-panel comparison
    plot
  repo: https://github.com/JayWadekar/gwIAS-HM
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1103/physrevd.110.044063
    title: New search pipeline for gravitational waves with higher-order modes using
      mode-by-mode filtering
  source_content_hash: sha256:94a103a1dd366d09428d1089f7622a792d639756cd705a4b93517e640e76abb1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/JayWadekar/gwIAS-HM
  tools:
  - matplotlib
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_2950
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# non-gaussian-tail-characterization

## Summary

Constructs an empirical, template-dependent 1D noise probability function P(ρ²_sum|N) from timeslide background coincident triggers of the IAS-HM GW search, and quantifies where the tails depart from the Gaussian (chi-squared-like) expectation — notably non-Gaussian dominance already at ρ²_sum ~ 40 in very high-mass banks (M_tot > 200 M_sun).

## When to use

Use when timeslide background coincident triggers must be converted into per-bank empirical P(ρ²_sum|N) curves to test Gaussianity of the HM trigger-ranking statistic, especially for high-total-mass template banks.

## When NOT to use

- Input is raw strain time series or single-detector triggers rather than coincident/background candidates from the coincidence module — the empirical P(ρ²_sum|N) requires timeslide coincident triggers.
- The goal is candidate ranking/FAR computation or p_astro tables (Ranking_candidates.ipynb, p_astro+detection_tables.ipynb) — this skill only characterizes the background distribution that feeds such downstream steps.
- An analytic Gaussian background is acceptable (e.g. low-mass quadrupole-only banks where the empirical curve matches the chi-squared expectation) and a template-dependent empirical model is not needed.

## Inputs

- timeslide background coincident trigger catalog from the coincidence module (e.g. pandas/CSV/HDF5 table with rho2_sum, degrees of freedom N, bank/subbank identifier, total mass M_tot)
- template bank / subbank metadata mapping triggers to template groups and total mass

## Outputs

- per-bank and per-template-group empirical 1D noise probability curves P(ρ²_sum|N)
- per-bank non-Gaussian tail excess metrics (deviation from the scipy.stats chi-squared-like expectation, flagged for M_tot > 200 M_sun banks)
- Fig.-3-style multi-panel comparison plot of P(ρ²_sum|N) curves per template bank

## How to apply

Load the timeslide background coincident triggers produced by the coincidence module with pandas, tagging each trigger with its template bank/subbank identifier and total mass. For each bank and template group, histogram ρ²_sum conditionally on the degrees of freedom N (density-normalized) with numpy to obtain the empirical P(ρ²_sum|N). Overlay the Gaussian chi-squared-like expectation for each (ρ²_sum, N) pair using scipy.stats to expose departures from Gaussianity. Quantify the non-Gaussian tail excess per bank and isolate very high-mass banks (M_tot > 200 M_sun), testing whether they deviate from Gaussian already at ρ²_sum ~ 40; different template groups within a bank must be curve-compared because a common background model across templates is suboptimal. Render the per-bank curves as a Fig.-3-style multi-panel matplotlib comparison for use in the empirical template-dependent background model of the ranking statistic.

## Related tools

- **pandas** (load and tag timeslide coincident triggers with bank/subbank identifier and total mass) — https://github.com/JayWadekar/gwIAS-HM
- **numpy** (build density-normalized histograms of rho2_sum conditional on N per bank/template group) — https://github.com/JayWadekar/gwIAS-HM
- **scipy** (overlay the Gaussian (chi-squared-like) expectation for each (rho2_sum, N) pair and quantify tail excess) — https://github.com/JayWadekar/gwIAS-HM
- **matplotlib** (render the per-bank P(rho2_sum|N) curves as a Fig.-3-style multi-panel comparison plot) — https://github.com/JayWadekar/gwIAS-HM

## Examples

```
from scipy.stats import chi2; import numpy as np; P_emp, edges = np.histogram(df.query('subbank==@b and N==@N').rho2_sum, bins=100, density=True); p_gauss = chi2.sf(edges, df=N)
```

## Evaluation signals

- Very high-mass bank panels (M_tot > 200 M_sun) must show non-Gaussian noise dominating even at low SNR, i.e. rho2_sum ~ 40, consistent with the Section III.B finding.
- Different template groups within a single bank must yield distinct P(ρ²_sum|N) curves, demonstrating that a common background model across templates is suboptimal.
- For low-total-mass banks, the empirical P(ρ²_sum|N) should approach the Gaussian chi-squared-like expectation; large departures there indicate a binning/conditioning error.
- Each empirical curve must be conditioned on a fixed N (degrees of freedom); mixing N values within one histogram invalidates the comparison.
- The tail excess per bank should be reported quantitatively (e.g. ratio of empirical to Gaussian probability in the high-ρ²_sum tail), not only visually.

## Limitations

- The empirical background is template-dependent per bank/subbank; applying one common background model across templates is suboptimal by the paper's own finding.
- ML methods for the non-Gaussian correction are still in preparation per the gwIAS-HM README, so the empirical construction is the current state.
- The IAS-HM pipeline currently hardcodes only Hanford and Livingston detectors and the three modes (22, 33, 44), constraining the trigger statistics available for background construction.

## Evidence
