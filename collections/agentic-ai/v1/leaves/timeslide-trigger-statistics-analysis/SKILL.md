---
name: timeslide-trigger-statistics-analysis
description: Use when coincident/background triggers from timeslides exist per template
  bank or subbank and you need empirical per-bank P(rho2_sum|N) background models,
  or must test where non-Gaussian transients dominate relative to the chi-squared-like
  Gaussian expectation.
when_to_use_negative:
- The goal is astrophysical candidate detection or FAR ranking of real (non-timeslide)
  events — this skill characterizes noise background only, e.g. via Ranking_candidates.ipynb-style
  FAR calculation
- Only a single global Gaussian background model is acceptable, e.g. for a quadrupole-only
  comparison where template-dependence is not being modeled — a common model across
  templates is known to be suboptimal here
- The input triggers come from injections or real detection chunks rather than timeslides,
  so they do not represent noise background statistics
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3360
tools:
- name: numpy
  role: Histogram rho2_sum trigger statistics conditionally on N to construct empirical
    per-bank P(rho2_sum|N)
  repo: https://github.com/JayWadekar/gwIAS-HM
- name: scipy
  role: Overlay the Gaussian (chi-squared-like) expectation per (rho2_sum, N) pair
    via scipy.stats to identify departures from Gaussianity
  repo: https://github.com/JayWadekar/gwIAS-HM
- name: pandas
  role: Load and tag timeslide background coincident triggers with template bank/subbank
    identifiers and total mass
  repo: https://github.com/JayWadekar/gwIAS-HM
- name: matplotlib
  role: Render Fig.-3-style multi-panel per-bank P(rho2_sum|N) comparison plots
  repo: https://github.com/JayWadekar/gwIAS-HM
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1103/physrevd.110.044063
    title: New search pipeline for gravitational waves with higher-order modes using
      mode-by-mode filtering
  source_content_hash: sha256:2add7653ce8290dfa7b8d1288a87844d5ea3df745529b3a7ba0d7584def90d17
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/JayWadekar/gwIAS-HM
  tools:
  - matplotlib
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# timeslide-trigger-statistics-analysis

## Summary

Builds empirical, template-dependent 1D noise probability functions P(rho2_sum|N) from timeslide background coincident triggers of the IAS-HM GW search, quantifying bank-dependent non-Gaussianity that the ranking statistic must model.

## When to use

Use when coincident/background triggers from timeslides exist per template bank or subbank and you need empirical per-bank P(rho2_sum|N) background models, or must test where non-Gaussian transients dominate relative to the chi-squared-like Gaussian expectation.

## When NOT to use

- The goal is astrophysical candidate detection or FAR ranking of real (non-timeslide) events — this skill characterizes noise background only, e.g. via Ranking_candidates.ipynb-style FAR calculation
- Only a single global Gaussian background model is acceptable, e.g. for a quadrupole-only comparison where template-dependence is not being modeled — a common model across templates is known to be suboptimal here
- The input triggers come from injections or real detection chunks rather than timeslides, so they do not represent noise background statistics

## Inputs

- Timeslide background coincident triggers from the coincidence module (e.g. CSV/DataFrame of coincident candidates with rho2_sum values)
- Per-trigger metadata: template bank/subbank identifier, degrees of freedom N, total mass M_tot
- Gaussian (chi-squared-like) reference distribution parameters for each (rho2_sum, N) pair

## Outputs

- Empirical per-bank / per-template-group 1D noise probability functions P(rho2_sum|N)
- Non-Gaussian tail excess statistics per bank, flagging very high-mass banks (M_tot > 200 M_sun) where non-Gaussianity dominates at rho2_sum ~ 40
- Fig.-3-style multi-panel matplotlib comparison plot of per-bank P(rho2_sum|N) curves against the Gaussian expectation

## How to apply

Load the timeslide coincident background triggers produced by the coincidence module with pandas, tagging each trigger with its template bank/subbank identifier and total mass. For each bank and template group, histogram the rho2_sum trigger statistic conditionally on the number of degrees of freedom N using numpy to form the empirical 1D noise probability P(rho2_sum|N). Overlay the Gaussian (chi-squared-like) expectation for each (rho2_sum, N) pair with scipy.stats to expose departures from Gaussianity, and quantify the non-Gaussian tail excess per bank. Pay particular attention to very high-mass banks (M_tot > 200 M_sun): the article's Section III.B finding is that in these banks non-Gaussian noise dominates even at low SNR (rho2 ~ 40), and that different template groups within a bank show distinct P(rho2|N) curves, so a single background model shared across templates is suboptimal — this empirical, template-dependent model is what feeds the IAS-HM ranking statistic to account for non-Gaussian transients. Render the per-bank curves as a Fig.-3-style multi-panel comparison with matplotlib.

## Related tools

- **numpy** (Histogram rho2_sum trigger statistics conditionally on N to construct empirical per-bank P(rho2_sum|N)) — https://github.com/JayWadekar/gwIAS-HM
- **scipy** (Overlay the Gaussian (chi-squared-like) expectation per (rho2_sum, N) pair via scipy.stats to identify departures from Gaussianity) — https://github.com/JayWadekar/gwIAS-HM
- **pandas** (Load and tag timeslide background coincident triggers with template bank/subbank identifiers and total mass) — https://github.com/JayWadekar/gwIAS-HM
- **matplotlib** (Render Fig.-3-style multi-panel per-bank P(rho2_sum|N) comparison plots) — https://github.com/JayWadekar/gwIAS-HM

## Examples

```
import pandas as pd; trig = pd.read_csv('timeslide_background_coincs.csv'); P = np.histogram(trig[trig.M_tot>200].rho2_sum, bins=50, density=True)
```

## Evaluation signals

- Very high-mass panels (M_tot > 200 M_sun) must show non-Gaussian behavior dominating even at rho2_sum ~ 40, consistent with the Section III.B finding
- Different template groups within a single bank must yield visibly distinct P(rho2_sum|N) curves, demonstrating template-dependence
- Empirical curves in low-mass banks should track the chi-squared-like Gaussian expectation at low rho2_sum before deviating in the tails
- Per-bank tail excess quantification must isolate M_tot > 200 M_sun banks as the most non-Gaussian group
- Every empirical P(rho2_sum|N) curve must be built only from timeslide coincident triggers and be labeled with the correct N (degrees of freedom)

## Limitations

- A common background model across templates is suboptimal because distinct template groups within a bank exhibit distinct P(rho2|N) curves
- Non-Gaussianity is bank-dependent: it dominates at low SNR (rho2 ~ 40) only in very high-mass banks (M_tot > 200 M_sun), so findings do not transfer uniformly across the bank
- The pipeline's planned improvements (e.g. ML classifier to augment the detection statistic, re-including veto-failed candidates with a penalty) are not yet implemented, so background characterization feeds into an incomplete ranking framework

## Evidence

- [methods] The empirical per-bank probability functions show that in very high-mass banks (M_tot > 200 M_sun), non-Gaussian noise dominates even at low SNR (rho2 ~ 40): [evidence span withheld — anchor status: not_found]
- [methods] timeslide background triggers are produced by the coincidence module: [evidence span withheld — anchor status: not_found]
- [methods] validation criterion: [evidence span withheld — anchor status: not_found]
