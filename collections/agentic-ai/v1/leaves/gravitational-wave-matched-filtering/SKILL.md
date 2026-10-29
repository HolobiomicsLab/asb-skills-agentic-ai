---
name: gravitational-wave-matched-filtering
description: Use when analyzing public LVK/GWOSC strain data to search for compact-binary
  coalescences with higher-order modes, or when you need per-harmonic SNR time series
  (rho_22, rho_33, rho_44) rather than quadrupole-only matched filtering.
when_to_use_negative:
- You only need a quadrupole-only (2,2) search with no higher-mode content — the per-harmonic
  machinery triples the filtering cost for no gain
- You need parameter estimation on already-detected events — use the companion cogwheel
  PE code instead of the search/triggering modules
- Your detector network goes beyond Hanford and Livingston — the current code hardcodes
  those two detectors and the three modes (22,33,44)
tools:
- name: gwIAS-HM
  role: Search pipeline providing the triggering module (data pre-processing and matched
    filtering), band eraser, and coincidence module used for mode-by-mode filtering
  repo: https://github.com/JayWadekar/gwIAS-HM
- name: lal / lalsimulation
  role: Generate the (2,2), (3,3), (4,4) harmonic waveform templates used in matched
    filtering
- name: numpy/scipy
  role: Implement the filtering of conditioned strain against harmonic templates
- name: band eraser
  role: Excises narrow time-varying noisy bands in time-frequency space before matched
    filtering
  repo: https://github.com/JayWadekar/gwIAS-HM
- name: matplotlib
  role: Render the background trigger-count comparison across ranking statistics (background_trigger_comparison.png)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1103/physrevd.110.044063
    title: New search pipeline for gravitational waves with higher-order modes using
      mode-by-mode filtering
  source_content_hash: sha256:a18c344044afff41115c673a9a7c9cb83544dec2e316f181bd3403b9af4915bc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/JayWadekar/gwIAS-HM
  tools:
  - IAS-HM search pipeline
  - band eraser
  - gwIAS-HM
  - lal
  - lal / lalsimulation
  - matplotlib
  - numpy
  - numpy/scipy
  - scipy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7368421052631579
n_claims: 19
---

# gravitational-wave-matched-filtering

## Summary

Mode-by-mode matched filtering of GW strain with (2,2), (3,3) and (4,4) harmonic templates in the IAS-HM pipeline: instead of building templates from harmonic combinations, data are filtered with each harmonic separately and the resulting SNR time series are combined later via physically constrained trigger-ranking statistics.

## When to use

Use when analyzing public LVK/GWOSC strain data to search for compact-binary coalescences with higher-order modes, or when you need per-harmonic SNR time series (rho_22, rho_33, rho_44) rather than quadrupole-only matched filtering.

## When NOT to use

- You only need a quadrupole-only (2,2) search with no higher-mode content — the per-harmonic machinery triples the filtering cost for no gain
- You need parameter estimation on already-detected events — use the companion cogwheel PE code instead of the search/triggering modules
- Your detector network goes beyond Hanford and Livingston — the current code hardcodes those two detectors and the three modes (22,33,44)

## Inputs

- Public LVK strain data from GWOSC (gw-openscience.org), e.g. an O3a strain chunk
- Mode-by-mode harmonic template banks for (2,2), (3,3), (4,4) built with lalsimulation waveforms
- Astrophysical prior for templates and HM mode amplitude ratios (generated via normalizing flows)

## Outputs

- Per-harmonic SNR time series (rho_22, rho_33, rho_44) for each template
- Coincident/background candidate triggers with veto-test results and coherent statistic values
- Background trigger counts at fixed thresholds per ranking statistic (e.g. background_trigger_comparison.png)

## How to apply

Pre-process strain with the pipeline's data-conditioning and use the band eraser to excise narrow time-varying noisy bands in time-frequency space without dropping whole time chunks. Matched-filter the conditioned data separately with (2,2), (3,3) and (4,4) harmonic templates generated with lal/lalsimulation waveforms (numpy/scipy filtering), yielding three SNR time series per template. This raises the matched-filtering cost by only a factor of 3 versus a (2,2)-only search. Feed the per-harmonic triggers into the coincidence module to collect coincident/background candidates, apply veto tests, and compute the coherent statistic; then rank triggers with the marginalized statistic over physical higher-mode SNR ratios rather than the naive summed-HM statistic (|rho_22|^2 + |rho_33|^2 + |rho_44|^2), which admits many more background triggers.

## Related tools

- **gwIAS-HM** (Search pipeline providing the triggering module (data pre-processing and matched filtering), band eraser, and coincidence module used for mode-by-mode filtering) — https://github.com/JayWadekar/gwIAS-HM
- **lal / lalsimulation** (Generate the (2,2), (3,3), (4,4) harmonic waveform templates used in matched filtering)
- **numpy/scipy** (Implement the filtering of conditioned strain against harmonic templates)
- **band eraser** (Excises narrow time-varying noisy bands in time-frequency space before matched filtering) — https://github.com/JayWadekar/gwIAS-HM
- **matplotlib** (Render the background trigger-count comparison across ranking statistics (background_trigger_comparison.png))

## Examples

```
jupyter lab Trig_Coin_test_with_injection.ipynb
```

## Evaluation signals

- Background trigger counts at fixed thresholds for the marginalized statistic are significantly lower than for the naive summed-HM statistic (|rho_22|^2 + |rho_33|^2 + |rho_44|^2), reproducing the reported Fig. 2 trend
- The marginalized ranking statistic preserves the sensitivity of the HM pipeline while reducing triggers collected and processed
- Matched-filtering cost stays within ~3x the cost of a (2,2)-only search despite filtering three harmonics
- Veto tests pass and the coherent statistic is computed for collected coincident/background candidates
- Pipeline tutorial run recovers the known candidate GWC190711 in the ~5 hour example data chunk

## Limitations

- Three modes (22,33,44) are hard-coded in a few places; arbitrary numbers of ell=m (or ell!=m) modes are not yet configurable
- Only Hanford and Livingston detectors are hard-coded; arbitrary detector networks are not supported yet
- Non-Gaussian transients require an empirical template-dependent background model in the ranking statistic; an injection module to compute (VT) sensitivity is still planned
- Candidates that fail veto tests are currently removed completely rather than re-included with a penalty

## Evidence

- [abstract] Mode-by-mode filtering raises filtering cost only ~3x: "This results in an increase in the matched-filtering cost by only a factor of 3 compared to that of a (2, 2)-only search."
- [abstract] Band eraser preprocessing: "a tool called band eraser which specifically excises narrow time-varying noisy bands in time-frequency space (without having to excise entire time chunks in the data)"
- [readme] Per-harmonic rather than combined-harmonic filtering: "Instead of making templates with combination of harmonics, we matched-filter data with different harmonic templates separately"
- [methods] Coincidence module duties: "(for collecting coincident/background candidates, performing veto tests and computing coherent statistic)"
