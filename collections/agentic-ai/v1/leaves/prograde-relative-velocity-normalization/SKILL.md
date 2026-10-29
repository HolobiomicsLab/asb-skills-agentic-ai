---
name: prograde-relative-velocity-normalization
description: Use when an LLM agent receives raw pursuer/evader positions and velocities
  in the celestial-body reference frame in the KSPDG pursuit-evasion environment,
  and the goal is to augment the textual observation with a vessel-frame prograde-direction
  quantity to help the model achieve its mission.
when_to_use_negative:
- Observations are already expressed in the vessel frame, so the R^-1 transform is
  unnecessary or would double-rotate the relative velocity
- The scenario is not the pursuit-evasion task (e.g. KSPDG Lady-Bandit-Guard or sun-blocking
  problems), where the pursuer-minus-evader relative velocity is not the relevant
  augmentation quantity
- The goal is high-fidelity astrodynamical trajectory analysis rather than prompt
  augmentation for an LLM agent — use Lambert solvers or orbital propagation instead
edam_operation: http://edamontology.org/operation_3438
tools:
- name: kspdg
  role: Python package providing the KSPDG pursuer-evader environment observations
    (positions and velocities of pursuer and evader) that the prograde computation
    consumes
  repo: https://github.com/ARCLab-MIT/kspdg
- name: KSPDG
  role: Suite of differential game environments (pursuit-evasion) in Kerbal Space
    Program whose observation space is being augmented with the prograde vector
  repo: https://github.com/mit-ll/spacegym-kspdg
- name: Kerbal Space Program
  role: Game engine hosting the simulation whose vessel reference frame (x right,
    y towards target, z up) defines the rotation matrix R
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:bceae4399856711f015d23951d49d6a983c5cf8f7ce89c935be1d974febfaf53
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ARCLab-MIT/kspdg
  tools:
  - KSPDG
  - Kerbal Space Program
  - kspdg
  edam_operation: http://edamontology.org/operation_3438
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# prograde-relative-velocity-normalization

## Summary

Augments an LLM agent's observation space for the KSPDG Pursuer-Evader task by computing a normalized prograde vector p = R^-1 (v_p - v_e)/||v_p - v_e||, i.e. the pursuer-minus-evader velocity difference, normalized and rotated from the celestial-body reference frame into the vessel frame, then added to the prompt.

## When to use

Use when an LLM agent receives raw pursuer/evader positions and velocities in the celestial-body reference frame in the KSPDG pursuit-evasion environment, and the goal is to augment the textual observation with a vessel-frame prograde-direction quantity to help the model achieve its mission.

## When NOT to use

- Observations are already expressed in the vessel frame, so the R^-1 transform is unnecessary or would double-rotate the relative velocity
- The scenario is not the pursuit-evasion task (e.g. KSPDG Lady-Bandit-Guard or sun-blocking problems), where the pursuer-minus-evader relative velocity is not the relevant augmentation quantity
- The goal is high-fidelity astrodynamical trajectory analysis rather than prompt augmentation for an LLM agent — use Lambert solvers or orbital propagation instead

## Inputs

- Pursuer and evader positions and velocities in the celestial body reference frame (KSPDG observation tuples from the kspdg package)
- Vessel-local unit vectors e_r (right), e_f (towards target), e_u (up) used to build the rotation matrix R

## Outputs

- Normalized prograde vector p = R^-1 (v_p - v_e)/||v_p - v_e|| in the vessel frame (3 components, unit norm)
- Prograde vector text added to the LLM agent's observation prompt for each timestep

## How to apply

Load sample KSPDG pursuer-evader observations (positions and velocities of pursuer and evader in the celestial body reference frame) from the kspdg package. Construct the vessel-local unit vectors e_r (right), e_f (towards the target), and e_u (up) matching the paper's vessel reference frame (x right, y towards the target, z upwards), and assemble the rotation matrix R from these columns. Compute the pursuer-evader relative velocity v_p - v_e, normalize it, and apply the inverse rotation to obtain the prograde vector p = R^-1 (v_p - v_e)/||v_p - v_e||. Wrap the computation in a Python script (prograde_vector.py) that emits the prograde vector for each provided sample observation, and inject it into the observation space as prompt augmentation. The rationale is that the LLM, which consumes only natural-language telemetry, cannot easily infer the pursuit direction from raw inertial-frame state vectors, so this feature makes the prograde direction explicit.

## Related tools

- **kspdg** (Python package providing the KSPDG pursuer-evader environment observations (positions and velocities of pursuer and evader) that the prograde computation consumes) — https://github.com/ARCLab-MIT/kspdg
- **KSPDG** (Suite of differential game environments (pursuit-evasion) in Kerbal Space Program whose observation space is being augmented with the prograde vector) — https://github.com/mit-ll/spacegym-kspdg
- **Kerbal Space Program** (Game engine hosting the simulation whose vessel reference frame (x right, y towards target, z up) defines the rotation matrix R)

## Examples

```
import numpy as np; p = np.linalg.inv(R) @ ((v_p - v_e) / np.linalg.norm(v_p - v_e))
```

## Evaluation signals

- Each output prograde vector must be unit-norm (||p|| = 1 within floating-point tolerance)
- Each output prograde vector agrees with hand-computed reference values computed manually for the sample observations
- Sanity check: when R is the identity matrix, p equals the normalized pursuer-minus-evader velocity difference (v_p - v_e)/||v_p - v_e||
- The rotation matrix R is built from vessel unit vectors consistent with the paper's convention (x right, y towards the target, z upwards)
- The script emits a prograde vector for every provided sample observation, none missing

## Limitations

- The transform assumes observations are given in the celestial-body reference frame; the rotation matrix R must be constructed from the vessel's unit vectors along pitch/roll/yaw axes matching the paper's x-right, y-target, z-up convention — any mismatch in axis convention invalidates the result
- KSPDG provides evaluation environments, not training environments, and game time does not pause while agents make decisions, so the prograde computation must be fast enough for a non-blocking control loop
- This augmentation addresses only the prograde-direction quantity for pursuit-evasion; other KSPDG scenarios (lady-bandit-guard, sun-blocking) are not covered by this feature

## Evidence

- [methods] prograde definition: [evidence span withheld — anchor status: not_found]
- [methods] augmentation purpose: [evidence span withheld — anchor status: not_found]
- [methods] vessel frame convention: [evidence span withheld — anchor status: not_found]
- [methods] validation: [evidence span withheld — anchor status: not_found]
- [readme] environment type: [evidence span withheld — anchor status: not_found]
- [readme] real-time constraint: [evidence span withheld — anchor status: not_found]
