---
name: template-bank-waveform-generation
description: Use when building template banks for a matched-filter GW search that
  filters each harmonic (l,|m|) separately and needs per-mode normalized templates
  plus stored mode SNR-ratio samples rather than combined-harmonic templates.
when_to_use_negative:
- You need arbitrary or ell!=m modes beyond the three hard-coded (22, 33, 44) modes
  — the current code hard-codes these modes in several places
- You are building combined multi-harmonic templates rather than filtering each harmonic
  separately (IAS-HM deliberately avoids combination templates)
- The bank targets precessing or eccentric signals — IMRPhenomXHM harmonic generation
  here covers aligned-spin ell=m harmonics
tools:
- name: lalsimulation
  role: Generates the IMRPhenomXHM (2,2), (3,3), (4,4) frequency-domain harmonic waveforms
    for each bank parameter point
- name: cogwheel
  role: Waveform infrastructure used to wrap the lalsimulation IMRPhenomXHM calls
    inside the pipeline
  repo: https://github.com/jroulet/cogwheel
- name: gwIAS-HM pipeline
  role: Hosts the Template_banks.ipynb and Astrophysical_prior.ipynb notebooks that
    produce the bank file and the observable-volume weights consumed here
  repo: https://github.com/JayWadekar/gwIAS-HM
- name: numpy
  role: Array backend for whitening/normalizing templates against the PSD and computing
    inner products for the R_lm ratios
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1103/physrevd.110.044063
    title: New search pipeline for gravitational waves with higher-order modes using
      mode-by-mode filtering
  source_content_hash: sha256:ae839007453ac946a786af5d779ea0fe77f6cb9a621e93798aa7afe9bb7f41a8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jroulet/cogwheel
  tools:
  - astropy
  - cogwheel
  - gwIAS-HM pipeline
  - lalsimulation
  - numba
  - numpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# template-bank-waveform-generation

## Summary

Generates whitened, unit-normalized per-harmonic frequency-domain templates ((2,2), (3,3), (4,4)) from IMRPhenomXHM for each subbank parameter point in the IAS-HM mode-by-mode search, together with physically allowed mode-SNR ratio samples R_33/R_44 used to combine harmonic SNR time series.

## When to use

Use when building template banks for a matched-filter GW search that filters each harmonic (l,|m|) separately and needs per-mode normalized templates plus stored mode SNR-ratio samples rather than combined-harmonic templates.

## When NOT to use

- You need arbitrary or ell!=m modes beyond the three hard-coded (22, 33, 44) modes — the current code hard-codes these modes in several places
- You are building combined multi-harmonic templates rather than filtering each harmonic separately (IAS-HM deliberately avoids combination templates)
- The bank targets precessing or eccentric signals — IMRPhenomXHM harmonic generation here covers aligned-spin ell=m harmonics

## Inputs

- Template bank file of subbank parameter points (masses, spins, orientation) from Template_banks.ipynb
- Detector noise PSD for the target detector (e.g. Hanford/Livingston)
- Astrophysical prior / observable-volume weights for HM mode amplitude ratios from Astrophysical_prior.ipynb (normalizing flows)

## Outputs

- templates_subbank.h5 containing unit-normalized (2,2), (3,3), (4,4) harmonic templates for every subbank point
- Stored R33/R44 mode SNR ratio samples at ι = π/2 with observable-volume weights

## How to apply

Load the subbank parameter points (masses, spins, orientation) from the bank file produced by the Template_banks notebook. For each point, generate the (2,2), (3,3), and (4,4) frequency-domain waveforms with IMRPhenomXHM via lalsimulation, using the cogwheel waveform infrastructure. Whiten and normalize each harmonic template against the detector noise PSD so the per-mode matched-filter SNR is unity for unit signal amplitude. Compute R_33 and R_44 as the per-mode inner product divided by that of the (2,2) mode at inclination ι = π/2, weighting the fractional-SNR ratios with the astrophysical prior (observable-volume weights) from the normalizing-flow prior notebook. Store the normalized templates and the R33/R44 ratio table in templates_subbank.h5; the mode-by-mode design keeps the per-subbank waveform count at 3x a quadrupole-only bank, matching the reported factor-of-3 matched-filtering cost increase.

## Related tools

- **lalsimulation** (Generates the IMRPhenomXHM (2,2), (3,3), (4,4) frequency-domain harmonic waveforms for each bank parameter point)
- **cogwheel** (Waveform infrastructure used to wrap the lalsimulation IMRPhenomXHM calls inside the pipeline) — https://github.com/jroulet/cogwheel
- **gwIAS-HM pipeline** (Hosts the Template_banks.ipynb and Astrophysical_prior.ipynb notebooks that produce the bank file and the observable-volume weights consumed here) — https://github.com/JayWadekar/gwIAS-HM
- **numpy** (Array backend for whitening/normalizing templates against the PSD and computing inner products for the R_lm ratios)

## Examples

```
jupyter nbconvert --to notebook --execute Template_banks.ipynb
```

## Evaluation signals

- templates_subbank.h5 exists and contains exactly the three hard-coded modes (22, 33, 44) for every subbank parameter point
- Each stored harmonic template is unit-normalized (per-mode matched-filter SNR equals 1 for unit signal amplitude against the detector PSD)
- R33/R44 values fall within the physically allowed fractional-SNR range computed at ι = π/2 from IMRPhenomXHM waveforms
- Number of templates per subbank equals 3x the corresponding quadrupole-only bank, consistent with the factor-of-3 matched-filtering cost

## Limitations

- Only three ell=m modes (22, 33, 44) are hard-coded in several places; generalization to an arbitrary number of modes or a quadrupole-only comparison search is planned but not implemented
- The pipeline currently hard-codes only Hanford and Livingston detectors rather than supporting an arbitrary detector set
- No injection module yet exists to measure the search's (VT) sensitivity, so template-bank sensitivity gains are not directly benchmarked in-repo

## Evidence

- [abstract] Mode-by-mode filtering with three harmonics: [evidence span withheld — anchor status: not_found]
