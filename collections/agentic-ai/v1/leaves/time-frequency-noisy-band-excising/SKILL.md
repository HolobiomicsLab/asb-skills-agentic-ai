---
name: time-frequency-noisy-band-excising
description: Use when conditioning public LVK/GWOSC strain chunks for matched filtering
  and specific narrow, possibly time-varying noisy bands would otherwise force excision
  of whole time chunks and lose analyzable data.
when_to_use_negative:
- The noise contaminating the data occupies entire time chunks (e.g. full data-quality
  vetted periods), so chunk-level vetoes, not band excision, are appropriate
- The goal is parameter estimation on a candidate event (cogwheel) rather than search
  pre-processing — band excision is part of the triggering module's data-conditioning,
  not PE
tools:
- name: band eraser
  role: the pipeline's own data-conditioning tool that excises narrow time-varying
    noisy bands in time-frequency space
  repo: https://github.com/JayWadekar/gwIAS-HM
- name: IAS-HM search pipeline
  role: host pipeline whose triggering module (data pre-processing and matched filtering)
    applies the band eraser to strain data
  repo: https://github.com/JayWadekar/gwIAS-HM
- name: GW_higher_harmonics_search notebook
  role: tutorial notebook demonstrating use of the band eraser module on raw LVK strain
  repo: https://github.com/JayWadekar/GW_higher_harmonics_search
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1103/physrevd.110.044063
    title: New search pipeline for gravitational waves with higher-order modes using
      mode-by-mode filtering
  source_content_hash: sha256:493d7c8296178e154943376fa6bdfdbb9721e222f1886af2b49a3fcce19b0eca
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/JayWadekar/gwIAS-HM
  tools:
  - GW_higher_harmonics_search notebook
  - IAS-HM search pipeline
  - band eraser
  - lal
  - matplotlib
  - numpy
  - scipy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# time-frequency-noisy-band-excising

## Summary

Excises narrow, time-varying noisy spectral bands from GW strain data in time-frequency space using the pipeline's 'band eraser' tool, without discarding entire time chunks — preserving usable coincident data during data-conditioning of LVK observing-run strain.

## When to use

Use when conditioning public LVK/GWOSC strain chunks for matched filtering and specific narrow, possibly time-varying noisy bands would otherwise force excision of whole time chunks and lose analyzable data.

## When NOT to use

- The noise contaminating the data occupies entire time chunks (e.g. full data-quality vetted periods), so chunk-level vetoes, not band excision, are appropriate
- The goal is parameter estimation on a candidate event (cogwheel) rather than search pre-processing — band excision is part of the triggering module's data-conditioning, not PE

## Inputs

- public LVK/GWOSC strain data chunk (e.g. an O3a ~5 hour chunk) from gw-openscience.org
- identified noisy bands (time-varying, narrow) in the strain's time-frequency representation

## Outputs

- conditioned strain data with narrow time-varying noisy bands excised, retaining otherwise-usable time chunks for matched filtering

## How to apply

After downloading a public O3a strain chunk from gw-openscience.org, run the pipeline's data-conditioning step and apply the band eraser as part of the triggering module's pre-processing. The tool identifies narrow noisy bands in time-frequency space and removes only those time-frequency regions that are actually noisy at a given time, rather than deleting entire time chunks that merely intersect a noisy frequency band. This maximizes the surviving data volume available to the (2,2), (3,3) and (4,4) harmonic matched filters. The band-eraser workflow is distributed as a tutorial notebook in the Pipeline_modules folder of the GW_higher_harmonics_search repository, so reproduce it on public LVK strain data by following that notebook before running matched filtering and the coincidence module.

## Related tools

- **band eraser** (the pipeline's own data-conditioning tool that excises narrow time-varying noisy bands in time-frequency space) — https://github.com/JayWadekar/gwIAS-HM
- **IAS-HM search pipeline** (host pipeline whose triggering module (data pre-processing and matched filtering) applies the band eraser to strain data) — https://github.com/JayWadekar/gwIAS-HM
- **GW_higher_harmonics_search notebook** (tutorial notebook demonstrating use of the band eraser module on raw LVK strain) — https://github.com/JayWadekar/GW_higher_harmonics_search

## Evaluation signals

- Surviving data volume: time chunks that only partially intersect a noisy band should be retained, not fully excised
- Conditioned strain should feed into the triggering module's matched filtering with (2,2), (3,3) and (4,4) harmonics without chunk-loss artifacts at the excised bands
- Background trigger behavior should match the paper's rationale: excising only narrow bands avoids removing time chunks, which would reduce background statistics and search sensitivity

## Limitations

- The tool addresses narrow time-varying noisy bands; it is not a substitute for removing entire bad-quality time chunks flagged by data-quality vetoes
- The pipeline's modules are still being cleaned up and documented (README: 'We are working on cleaning up already included modules and adding more documentation')

## Evidence

- [readme] band eraser module notebook: [evidence span withheld — anchor status: external_doc]
