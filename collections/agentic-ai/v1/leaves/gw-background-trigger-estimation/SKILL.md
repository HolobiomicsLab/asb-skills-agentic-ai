---
name: gw-background-trigger-estimation
description: Use when comparing trigger-collection statistics for a gravitational-wave
  search — specifically when a naive summed-higher-mode SNR cut admits too many background
  triggers relative to a (2,2)-only cut and a marginalized ranking statistic must
  be evaluated instead.
when_to_use_negative:
- The goal is parameter estimation or sensitivity (VT) measurement via injections
  — the injection module for (VT) sensitivity is still listed as planned/incomplete
  in the pipeline
- Only a quadrupole-only (2,2) search is needed — the pipeline currently hard-codes
  the three modes (22,33,44) and is not yet general for an arbitrary number of modes
- Working with multi-detector data beyond Hanford and Livingston — only H1 and L1
  are currently hardcoded in the code
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3418
tools:
- name: gwIAS-HM
  role: Source pipeline providing the triggering module (data pre-processing and matched
    filtering with separate harmonic templates), coincidence module (collecting coincident/background
    candidates, veto tests, coherent statistic), and notebooks for collecting triggers
    based on the coherent marginalized score
  repo: https://github.com/JayWadekar/gwIAS-HM
- name: GW_higher_harmonics_search
  role: Repository of data products and codes for the two relevant modules (coherent
    marginalized score trigger collection, band eraser) and the candidates catalog
    containing above/below-threshold candidates
  repo: https://github.com/JayWadekar/GW_higher_harmonics_search
- name: band eraser
  role: Excises narrow time-varying noisy bands in time-frequency space during data
    conditioning, without removing entire time chunks of strain data
  repo: https://github.com/JayWadekar/gwIAS-HM
- name: numpy / scipy / matplotlib
  role: Matched filtering of conditioned strain against harmonic templates, statistic
    evaluation, and rendering background_trigger_comparison.png
- name: lal / lalsimulation
  role: Generation of the harmonic waveform templates used in the mode-by-mode matched
    filtering
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1103/physrevd.110.044063
    title: New search pipeline for gravitational waves with higher-order modes using
      mode-by-mode filtering
  source_content_hash: sha256:baeaed89f3de2c37fd7fe04c3d00cee2483340a484a7ee96b56c28d30c221924
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/JayWadekar/gwIAS-HM
  tools:
  - GW_higher_harmonics_search
  - IAS-HM search pipeline
  - band eraser
  - gwIAS-HM
  - lal
  - lal / lalsimulation
  - matplotlib
  - numpy
  - numpy / scipy / matplotlib
  - scipy
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3418
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8421052631578947
n_claims: 19
---

# gw-background-trigger-estimation

## Summary

Reproduces the IAS-HM pipeline's trigger-statistic comparison by collecting coincident/background triggers from public O3a strain and counting them at fixed thresholds under four ranking statistics: (2,2)-only SNR, naive summed-HM SNR, distance-to-physical-injection, and the marginalized HM statistic. It demonstrates that the marginalized statistic reduces background trigger load while preserving sensitivity.

## When to use

Use when comparing trigger-collection statistics for a gravitational-wave search — specifically when a naive summed-higher-mode SNR cut admits too many background triggers relative to a (2,2)-only cut and a marginalized ranking statistic must be evaluated instead.

## When NOT to use

- The goal is parameter estimation or sensitivity (VT) measurement via injections — the injection module for (VT) sensitivity is still listed as planned/incomplete in the pipeline
- Only a quadrupole-only (2,2) search is needed — the pipeline currently hard-codes the three modes (22,33,44) and is not yet general for an arbitrary number of modes
- Working with multi-detector data beyond Hanford and Livingston — only H1 and L1 are currently hardcoded in the code

## Inputs

- Public O3a GW strain data chunk from the Gravitational Wave Open Science Center (gw-openscience.org)
- (2,2), (3,3), (4,4) harmonic waveform templates from lal/lalsimulation
- Astrophysical prior for HM mode amplitude ratios (e.g. from normalizing flows per Astrophysical_prior.ipynb)

## Outputs

- Collected coincident/background trigger lists per ranking statistic
- Background trigger counts at fixed thresholds for each of the four statistics
- Comparison figure background_trigger_comparison.png rendered with matplotlib

## How to apply

Download a public O3a strain chunk from the Gravitational Wave Open Science Center and pre-process it with the pipeline's data-conditioning, using the band eraser to excise narrow time-varying noisy bands in time-frequency space without removing entire time chunks. Matched-filter the conditioned data separately with (2,2), (3,3) and (4,4) harmonic templates using the triggering module (lal/lalsimulation waveforms, numpy/scipy filtering). Run the coincidence module to collect coincident/background candidates, perform veto tests, and compute the coherent statistic. Then evaluate the four ranking statistics on the collected background triggers and count triggers at fixed thresholds for each statistic. The rationale is that mode-by-mode filtering with only three harmonics costs a factor of 3 over a (2,2)-only search, and the marginalized statistic optimally combines SNR time series while triggering only on physically allowed combinations, suppressing loud non-Gaussian transients via an empirical template-dependent background model.

## Related tools

- **gwIAS-HM** (Source pipeline providing the triggering module (data pre-processing and matched filtering with separate harmonic templates), coincidence module (collecting coincident/background candidates, veto tests, coherent statistic), and notebooks for collecting triggers based on the coherent marginalized score) — https://github.com/JayWadekar/gwIAS-HM
- **GW_higher_harmonics_search** (Repository of data products and codes for the two relevant modules (coherent marginalized score trigger collection, band eraser) and the candidates catalog containing above/below-threshold candidates) — https://github.com/JayWadekar/GW_higher_harmonics_search
- **band eraser** (Excises narrow time-varying noisy bands in time-frequency space during data conditioning, without removing entire time chunks of strain data) — https://github.com/JayWadekar/gwIAS-HM
- **numpy / scipy / matplotlib** (Matched filtering of conditioned strain against harmonic templates, statistic evaluation, and rendering background_trigger_comparison.png)
- **lal / lalsimulation** (Generation of the harmonic waveform templates used in the mode-by-mode matched filtering)

## Examples

```
python Trig_Coin_on_cluster.py --strain gwosc_o3a_chunk --modes 22 33 44; # then rank background triggers with the marginalized statistic and plot background_trigger_comparison.png
```

## Evaluation signals

- Reproduced background trigger counts qualitatively match the reported Fig. 2 trend: the marginalized statistic suppresses loud non-Gaussian background relative to the naive summed-HM statistic
- The marginalized statistic retains physically allowed higher-mode combinations (it triggers only on physically allowed combinations of harmonic SNRs)
- Number of collected background triggers under the marginalized statistic is significantly lower than under the naive summed-HM statistic (|rho_perp22|^2 + |rho_perp33|^2 + |rho_perp44|^2), reducing search cost
- Sensitivity of the HM pipeline is preserved under the marginalized ranking statistic (no loss relative to the HM pipeline)
- Band-erased conditioning removes only narrow time-varying noisy bands, not entire time chunks, so the ~5 hour analysis chunk retains usable livetime

## Limitations

- Three modes (22, 33, 44) are hard-coded in a few places; the code does not yet support an arbitrary number of ell=m modes or a quadrupole-only search via configuration
- Only Hanford and Livingston detectors are hardcoded; arbitrary detector sets are not yet supported
- The injection module to calculate the (VT) sensitivity of the search is not yet implemented, so sensitivity preservation is validated via the background-comparison trend rather than a VT calculation
- Candidates that fail veto tests are currently removed entirely rather than re-included with a penalty, which can discard useful candidates

## Evidence

- [abstract] the marginalized ranking-statistic both preserves the sensitivity of the HM pipeline and decreases the overall search cost by significantly reducing the number of triggers collected and processed: [evidence span withheld — anchor status: not_found]
- [readme] notebook showing coherent marginalized score trigger collection: [evidence span withheld — anchor status: external_doc]
