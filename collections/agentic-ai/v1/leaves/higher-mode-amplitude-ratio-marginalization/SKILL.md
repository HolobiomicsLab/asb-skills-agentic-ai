---
name: higher-mode-amplitude-ratio-marginalization
description: Use when you have per-mode complex SNR timeseries from mode-by-mode matched
  filtering and need a computationally cheap single-detector score that penalizes
  unphysical harmonic-SNR combinations before coincidence.
when_to_use_negative:
- You are doing full parameter estimation rather than cheap trigger collection — use
  cogwheel's likelihood marginalization over extrinsic parameters instead
- Your waveform modes are not the hard-coded (2,2), (3,3), (4,4) set — the pipeline
  currently hardcodes these three modes and does not yet support arbitrary ell=m modes
- You need coherent multi-detector ranking with FARs — this is a single-detector statistic
  applied before the coincidence module
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: numpy
  role: Vectorized evaluation of the per-sample, phi0-maximized mode-SNR combinations
    so the statistic stays cheap on full SNR timeseries
- name: scipy
  role: Scientific routines supporting the score evaluation alongside numpy
- name: gwIAS-HM pipeline notebooks
  role: Astrophysical_prior.ipynb generates the mode-amplitude-ratio prior; Trig_Coin_test_with_injection.ipynb
    provides the worked-example validation of the score
  repo: https://github.com/JayWadekar/gwIAS-HM
- name: cogwheel
  role: Related marginalization machinery for waveforms with higher modes (extrinsic-parameter
    marginalization) referenced by the pipeline
  repo: https://github.com/jroulet/cogwheel
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1103/physrevd.110.044063
    title: New search pipeline for gravitational waves with higher-order modes using
      mode-by-mode filtering
  source_content_hash: sha256:c237a309d957e1d4def9d4034d46af2fb89e863bc66f248c202b14057ce06195
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/JayWadekar/gwIAS-HM
  tools:
  - cogwheel
  - gwIAS-HM pipeline notebooks
  - lal
  - matplotlib
  - numpy
  - scipy
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# higher-mode-amplitude-ratio-marginalization

## Summary

A single-detector trigger-collection statistic (Eq. A5 of the IAS-HM pipeline) that combines per-harmonic complex SNR timeseries from the (2,2), (3,3), and (4,4) modes by marginalizing over astrophysical prior samples of HM mode amplitude ratios, so only physically plausible mode-SNR combinations are collected cheaply at the trigger stage.

## When to use

Use when you have per-mode complex SNR timeseries from mode-by-mode matched filtering and need a computationally cheap single-detector score that penalizes unphysical harmonic-SNR combinations before coincidence.

## When NOT to use

- You are doing full parameter estimation rather than cheap trigger collection — use cogwheel's likelihood marginalization over extrinsic parameters instead
- Your waveform modes are not the hard-coded (2,2), (3,3), (4,4) set — the pipeline currently hardcodes these three modes and does not yet support arbitrary ell=m modes
- You need coherent multi-detector ranking with FARs — this is a single-detector statistic applied before the coincidence module

## Inputs

- Per-mode complex SNR timeseries for the (2,2), (3,3), and (4,4) harmonics from the mode-by-mode matched-filtering module
- Precomputed physical mode-ratio samples {r33, r44, w} from the normalizing-flow astrophysical prior

## Outputs

- Marginalized single-detector score timeseries over the full SNR timeseries, suitable for trigger collection and downstream coincidence/ranking

## How to apply

Load the per-mode complex SNR timeseries for the (2,2), (3,3), and (4,4) harmonics produced by the mode-by-mode matched-filtering module, together with precomputed physical mode-ratio samples {r33, r44, w} generated from the astrophysical prior (normalizing-flow-based HM mode amplitude ratios, e.g. from Astrophysical_prior.ipynb). For each prior sample, compute the analytically phi0-maximized combination of the three mode SNR timeseries that realizes that physical amplitude-ratio realization's log-likelihood contribution. Marginalize by averaging the per-sample scores weighted by w, which downweights mode-SNR combinations inconsistent with any physical template. Vectorize the evaluation with numpy/scipy so the statistic remains cheap enough to evaluate on the full SNR timeseries, then reproduce the worked example in Trig_Coin_test_with_injection.ipynb to validate.

## Related tools

- **numpy** (Vectorized evaluation of the per-sample, phi0-maximized mode-SNR combinations so the statistic stays cheap on full SNR timeseries)
- **scipy** (Scientific routines supporting the score evaluation alongside numpy)
- **gwIAS-HM pipeline notebooks** (Astrophysical_prior.ipynb generates the mode-amplitude-ratio prior; Trig_Coin_test_with_injection.ipynb provides the worked-example validation of the score) — https://github.com/JayWadekar/gwIAS-HM
- **cogwheel** (Related marginalization machinery for waveforms with higher modes (extrinsic-parameter marginalization) referenced by the pipeline) — https://github.com/jroulet/cogwheel

## Examples

```
score = np.average([phi0_maximized_combination(r33, r44, snr22, snr33, snr44) for r33, r44, w in prior_samples], weights=w)  # Eq. A5, see Trig_Coin_test_with_injection.ipynb
```

## Evaluation signals

- The computed score timeseries reproduces the Trig_Coin_test_with_injection.ipynb worked example within notebook-tolerant agreement
- The score peaks at the injection time for an injected signal
- The score is suppressed for unphysical mode-SNR combinations of the harmonic SNR timeseries
- Vectorized evaluation remains computationally cheap on the full SNR timeseries
- Per-sample scores are correctly weighted by the prior weights w in the marginalization average

## Limitations

- Three modes (22, 33, 44) are hard-coded in several places; the code does not yet let users specify an arbitrary number of ell=m modes or run a quadrupole-only comparison search
- Only Hanford and Livingston detectors are currently supported; arbitrary detector configurations are a planned feature
- The prior for mode amplitude ratios depends on a normalizing-flow astrophysical prior; non-Gaussian transients are handled by an empirical template-dependent background model in the broader ranking statistic, not by this single-detector statistic alone

## Evidence

- [abstract] mode-by-mode filtering: [evidence span withheld — anchor status: not_found]
