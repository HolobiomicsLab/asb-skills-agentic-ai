---
name: reference-frame-rotation-matrix-construction
description: Use when raw pursuer/evader state vectors are given in the celestial-body
  reference frame but the downstream agent consumes vessel-frame quantities, e.g.
  building the prograde prompt augmentation for KSPDG LLM agents.
when_to_use_negative:
- State vectors are already expressed in the vessel frame — applying R^-1 again would
  double-transform the relative velocity
- The task is raw KSPDG RL evaluation (e.g. Lambert-MPC or Navball bot baselines)
  where the agent consumes the default observation space without natural-language
  prompt augmentation
edam_operation: http://edamontology.org/operation_2990
tools:
- name: KSPDG
  role: Provides the pursuer-evasion environment whose observations (positions, velocities)
    supply the raw vectors for the rotation and prograde computation
  repo: https://github.com/mit-ll/spacegym-kspdg
- name: kspdg (ARCLab-MIT)
  role: Python package used to load sample pursuer-evader observations that the prograde
    computation is applied to
  repo: https://github.com/ARCLab-MIT/kspdg
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:acda6b28e19188278b05806e6f3128a0eb42bf2e3c3a46979b92afa1beb94064
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mit-ll/spacegym-kspdg
  tools:
  - KSPDG
  - kspdg (ARCLab-MIT)
  edam_operation: http://edamontology.org/operation_2990
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# reference-frame-rotation-matrix-construction

## Summary

Constructs a vessel-local rotation matrix R from unit vectors along the pitch, roll, and yaw axes (right, target, up) of the KSPDG pursuer-evader environment, and applies its inverse to transform vectors from the celestial-body reference frame into the vessel frame — e.g. to compute the normalized prograde vector p = R^-1 (v_p - v_e)/||v_p - v_e|| used as LLM observation augmentation.

## When to use

Use when raw pursuer/evader state vectors are given in the celestial-body reference frame but the downstream agent consumes vessel-frame quantities, e.g. building the prograde prompt augmentation for KSPDG LLM agents.

## When NOT to use

- State vectors are already expressed in the vessel frame — applying R^-1 again would double-transform the relative velocity
- The task is raw KSPDG RL evaluation (e.g. Lambert-MPC or Navball bot baselines) where the agent consumes the default observation space without natural-language prompt augmentation

## Inputs

- Pursuer and evader positions and velocities in the celestial-body reference frame (KSPDG pe1 pursuer-evasion observations from the kspdg package)
- Vessel-local unit vectors e_r (right), e_f (towards the target), e_u (up) built from vessel attitude

## Outputs

- 3x3 rotation matrix R mapping celestial-body frame to vessel frame (and its inverse R^-1)
- Unit-norm prograde vector p in the vessel frame, emitted per sample observation and added to the LLM agent's observation prompt

## How to apply

Load pursuer and evader positions and velocities from kspdg observations in the celestial-body reference frame. Assemble the rotation matrix R from the vessel-local unit vectors e_r (right), e_f (towards the target), and e_u (up), matching the paper's convention where the x axis points right, the y axis towards the target, and the z axis upwards. Compute the relative velocity v_p - v_e, normalize it, and apply the inverse rotation p = R^-1 (v_p - v_e)/||v_p - v_e||. Wrap the computation in a script (prograde_vector.py) emitting the prograde vector per sample observation, then append it to the observation space as prompt augmentation for the LLM agent.

## Related tools

- **KSPDG** (Provides the pursuer-evasion environment whose observations (positions, velocities) supply the raw vectors for the rotation and prograde computation) — https://github.com/mit-ll/spacegym-kspdg
- **kspdg (ARCLab-MIT)** (Python package used to load sample pursuer-evader observations that the prograde computation is applied to) — https://github.com/ARCLab-MIT/kspdg

## Examples

```
python prograde_vector.py  # loads sample kspdg pursuer-evader observations and emits R^-1(v_p - v_e)/||v_p - v_e|| for each
```

## Evaluation signals

- Each emitted prograde vector must be unit-norm (||p|| = 1 within numerical tolerance)
- Output prograde vectors must agree with hand-computed reference values for the sample observations
- R must be orthonormal: R * R^-1 = I and columns are mutually orthogonal unit vectors
- p is invariant to the normalization order: R^-1(v_p - v_e)/||v_p - v_e|| equals R^-1((v_p - v_e)/||v_p - v_e||)
- Augmented prompt observations must still fit within the LLM token budget (paper uses tiktoken for tokenization)

## Limitations

- Vessel-local unit vectors depend on the paper's frame convention (x right, y towards target, z up); mismatching conventions will silently flip sign or axis of the prograde vector
- KSPDG game time does not pause during agent decisions, so the rotation/prograde computation must complete quickly to remain valid
- The prograde augmentation was designed for the KSPDG Pursuer-Evader (pe1) task and is not automatically applicable to the LBG1 or Sun-blocking scenarios

## Evidence

- [methods] prograde definition: [evidence span withheld — anchor status: not_found]
- [methods] vessel frame convention: [evidence span withheld — anchor status: not_found]
- [methods] observation augmentation: [evidence span withheld — anchor status: not_found]
- [readme] kRPC control: [evidence span withheld — anchor status: not_found]
