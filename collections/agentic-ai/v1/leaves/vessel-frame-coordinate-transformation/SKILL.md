---
name: vessel-frame-coordinate-transformation
description: Use when augmenting KSPDG pursuer-evader observations (positions and
  velocities of pursuer and evader in the celestial-body reference frame) into textual
  prompts for an LLM agent that must capture the evader spacecraft.
when_to_use_negative:
- Observations are already expressed in the vessel-local frame or the agent consumes
  raw numeric state via a non-LLM controller (e.g. the Lambert-MPC or Navball bot
  baselines), so the frame transform is unnecessary
- The task is not the KSPDG Pursuer-Evader scenario (e.g. Lady-Bandit-Guard or Sun-blocking
  scenarios), where a prograde-direction augmentation is not defined by the paper
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: KSPDG
  role: Provides the pursuer-evader environment and observation vectors (pursuer/evader
    positions and velocities) that the prograde transform is computed on
  repo: https://github.com/mit-ll/spacegym-kspdg
- name: kspdg python package
  role: Python package used to load sample pursuer-evader observations for the prograde-vector
    computation
  repo: https://github.com/ARCLab-MIT/kspdg
- name: Kerbal Space Program
  role: Game engine whose vessel state supplies the vessel-local unit vectors (right,
    towards target, up) used to assemble the rotation matrix R
  repo: https://www.kerbalspaceprogram.com/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:562a826080c0dc136508b53aa7bbab0549364a3355db926b8722493928d71f85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mit-ll/spacegym-kspdg
  tools:
  - KSPDG
  - Kerbal Space Program
  - kspdg python package
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# vessel-frame-coordinate-transformation

## Summary

Reconstructs the paper's prograde-direction observation augmentation for the KSPDG Pursuer-Evader task: compute the normalized pursuer-minus-evader relative velocity and rotate it from the celestial-body reference frame into the vessel-local frame via the inverse of the rotation matrix R built from the vessel's right/target/up unit vectors, then append the resulting vector to the LLM agent's prompt observation space.

## When to use

Use when augmenting KSPDG pursuer-evader observations (positions and velocities of pursuer and evader in the celestial-body reference frame) into textual prompts for an LLM agent that must capture the evader spacecraft.

## When NOT to use

- Observations are already expressed in the vessel-local frame or the agent consumes raw numeric state via a non-LLM controller (e.g. the Lambert-MPC or Navball bot baselines), so the frame transform is unnecessary
- The task is not the KSPDG Pursuer-Evader scenario (e.g. Lady-Bandit-Guard or Sun-blocking scenarios), where a prograde-direction augmentation is not defined by the paper

## Inputs

- KSPDG pursuer-evader observation vectors (positions and velocities of pursuer and evader in the celestial-body reference frame) from the kspdg package
- Vessel-local unit vectors e_r (right), e_f (towards the target), e_u (up) as reported by the KSP vessel state

## Outputs

- Unit-norm prograde vector p = R^-1 (v_p - v_e) / ||v_p - v_e|| expressed in the vessel frame
- Prograde vector strings appended to the LLM agent's textual observation prompt

## How to apply

Load sample pursuer-evader observations (positions and velocities in the celestial-body reference frame) from the kspdg package. Construct the vessel-local unit vectors e_r (right, x axis), e_f (towards the target, y axis) and e_u (up, z axis) and assemble the rotation matrix R with these as columns. Compute the pursuer-evader relative velocity, normalize it, and apply the inverse rotation to obtain the prograde vector p = R^-1 (v_p - v_e) / ||v_p - v_e||. The rationale is that LLM agents relying solely on natural-language telemetry cannot reason about raw frame coordinates, so the paper adds this prograde quantity to the observation space as prompt augmentation; wrap the computation in a script (prograde_vector.py) that emits one prograde vector per provided sample observation.

## Related tools

- **KSPDG** (Provides the pursuer-evader environment and observation vectors (pursuer/evader positions and velocities) that the prograde transform is computed on) — https://github.com/mit-ll/spacegym-kspdg
- **kspdg python package** (Python package used to load sample pursuer-evader observations for the prograde-vector computation) — https://github.com/ARCLab-MIT/kspdg
- **Kerbal Space Program** (Game engine whose vessel state supplies the vessel-local unit vectors (right, towards target, up) used to assemble the rotation matrix R) — https://www.kerbalspaceprogram.com/

## Examples

```
python prograde_vector.py --input sample_observations.  # emits unit-norm prograde vector p = R^-1 (v_p - v_e)/||v_p - v_e|| per observation
```

## Evaluation signals

- Every emitted prograde vector has unit norm (||p|| = 1)
- Each output prograde vector agrees with hand-computed reference values computed manually for the sample observations
- The rotation matrix R, when composed with its inverse, recovers the identity (orthonormal column vectors e_r, e_f, e_u)
- Applying R (not R^-1) to the output vector recovers the normalized celestial-frame relative velocity (v_p - v_e)/||v_p - v_e||
- The augmented prompt for each observation contains a prograde-vector component for every sample observation provided

## Limitations

- KSPDG game time does not pause while agents make decisions, a non-blocking step function that penalizes slow computation of the augmented observation and control policy
- KSPDG provides evaluation environments, NOT training environments for reinforcement learning, so the augmented observations must be evaluated rather than trained on in the simulation
- KSP physics behavior is brittle when the physics range is extended > 100 km (vessel shaking, phantom forces, etc.), which can corrupt the vessel state feeding the transform

## Evidence

- [methods] prograde computation formula: [evidence span withheld — anchor status: not_found]
- [methods] prograde added to observation space: [evidence span withheld — anchor status: not_found]
- [other] vessel reference frame axes: [evidence span withheld — anchor status: not_found]
- [readme] KSPDG purpose: [evidence span withheld — anchor status: not_found]
