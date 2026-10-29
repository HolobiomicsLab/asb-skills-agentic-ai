---
name: kspdg-observation-processing
description: Use when raw KSPDG pursuer-evader observations (pursuer and evader positions/velocities
  in the celestial-body reference frame) must be transformed into a vessel-frame prograde
  quantity before being rendered as an LLM observation prompt.
when_to_use_negative:
- The observations are already expressed in the vessel-local frame — applying R^-1
  again would double-transform the relative velocity.
- The scenario is not the 1-v-1 Pursuit-Evasion problem (e.g. the Lady-Bandit-Guard
  lbg1 or Sun-blocking sb1 scenarios in KSPDG), where a pursuer-minus-evader prograde
  vector is not the intended augmentation.
- The agent is not consuming textual observation prompts (e.g. a classical Lambert-MPC
  or navball-bot controller), so prompt augmentation of the observation space is not
  the mechanism of use.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: KSPDG
  role: Provides the pursuer-evader environment whose observation space (positions/velocities
    in the celestial-body reference frame) is augmented with the prograde vector
  repo: https://github.com/mit-ll/spacegym-kspdg/tree main
- name: Kerbal Space Program (KSP)
  role: Game engine running the differential-game simulation whose real-time state
    feeds the observation augmentation
  repo: https://www.kerbalspaceprogram.com/
- name: kspdg (ARCLab-MIT agents repository)
  role: Source of sample KSPDG pursuer-evader observations and the agents folder where
    prompt-engineering agents live
  repo: https://github.com/ARCLab-MIT/kspdg/tree/main/arclab_mit/agents
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:256f9f3ef90164ee37d73f41fa724a8c9fd536e7fcce823ccf74c7a4279c9cdf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KSPDG
  - Kerbal Space Program (KSP)
  - github.com/ARCLab-MIT/kspdg
  - kspdg (ARCLab-MIT agents repository)
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# kspdg-observation-processing

## Summary

Augments the KSPDG Pursuer-Evader observation space by computing the unit-norm prograde vector p = R^-1 (v_p - v_e)/||v_p - v_e||, i.e. the normalized pursuer-minus-evader relative velocity rotated from the celestial-body reference frame into the vessel frame, which is added to the LLM agent's textual prompt.

## When to use

Use when raw KSPDG pursuer-evader observations (pursuer and evader positions/velocities in the celestial-body reference frame) must be transformed into a vessel-frame prograde quantity before being rendered as an LLM observation prompt.

## When NOT to use

- The observations are already expressed in the vessel-local frame — applying R^-1 again would double-transform the relative velocity.
- The scenario is not the 1-v-1 Pursuit-Evasion problem (e.g. the Lady-Bandit-Guard lbg1 or Sun-blocking sb1 scenarios in KSPDG), where a pursuer-minus-evader prograde vector is not the intended augmentation.
- The agent is not consuming textual observation prompts (e.g. a classical Lambert-MPC or navball-bot controller), so prompt augmentation of the observation space is not the mechanism of use.

## Inputs

- KSPDG pursuer-evader observation records: pursuer and evader positions and velocities in the celestial-body reference frame (from the kspdg python package, e.g. the pe1 scenario environment)
- Vessel orientation data sufficient to build unit vectors e_r (right), e_f (towards the target), e_u (up) along the pitch, roll, and yaw axes

## Outputs

- Normalized prograde vector p = R^-1 (v_p - v_e)/||v_p - v_e||, expressed in the vessel frame, one per observation step
- prograde_vector.py script emitting the prograde vector for each provided sample observation, suitable for insertion into the LLM prompt augmentation

## How to apply

Load sample pursuer-evader observations (positions and velocities of pursuer and evader in the celestial-body reference frame) from the kspdg package. Construct the vessel-local unit vectors e_r (right), e_f (towards the target) and e_u (up) matching the paper's vessel frame where x points right, y towards the target and z upwards, and assemble the rotation matrix R from these columns. Compute the pursuer-evader relative velocity (v_p - v_e), normalize it, and apply the inverse rotation to obtain p = R^-1 (v_p - v_e)/||v_p - v_e||, transforming from the celestial-body frame into the vessel frame. Wrap the computation in a Python script (prograde_vector.py) that emits the prograde vector for each sample observation, then add this vector to the observation space as prompt augmentation feeding the classic RL loop where real-time observations are transformed into textual user prompts.

## Related tools

- **KSPDG** (Provides the pursuer-evader environment whose observation space (positions/velocities in the celestial-body reference frame) is augmented with the prograde vector) — https://github.com/mit-ll/spacegym-kspdg/tree main
- **Kerbal Space Program (KSP)** (Game engine running the differential-game simulation whose real-time state feeds the observation augmentation) — https://www.kerbalspaceprogram.com/
- **kspdg (ARCLab-MIT agents repository)** (Source of sample KSPDG pursuer-evader observations and the agents folder where prompt-engineering agents live) — https://github.com/ARCLab-MIT/kspdg/tree/main/arclab_mit/agents

## Examples

```
python prograde_vector.py --input sample_observations.json
```

## Evaluation signals

- Each emitted prograde vector is unit-norm (||p|| = 1).
- Each output agrees with hand-computed reference values computed manually for the sample observations.
- The vector is expressed in the vessel frame (x right, y towards the target, z up), verifiable by checking it changes under vessel rotation while the celestial-frame relative velocity direction stays fixed.
- The un-rotated direction matches the normalized pursuer-evader velocity difference (v_p - v_e)/||v_p - v_e|| in the celestial-body frame.
- The augmented prompt, when fed to the LLM agent in the KSPDG RL loop, is parseable within the real-time (non-blocking) step-function decision window.

## Limitations

- KSPDG game time does not pause while agents make decisions (non-blocking step function), so the prograde computation must be fast enough not to penalize the policy.
- The prograde augmentation was designed for the basic pursuit-evasion environment (pe1); other KSPDG scenario groups (lbg1, sb1) have different observation spaces.
- KSP physics-range extension beyond ~100 km can cause phantom forces and vessel shaking, which can corrupt the underlying velocity observations used in the computation.

## Evidence

- [methods] The prograde vector is computed as p = R^-1 (v_p - v_e) / ||v_p - v_e||: the pursuer-minus-evader velocity difference, normalized, and transformed by the inverse of the rotation matrix R: [evidence span withheld — anchor status: not_found]
- [intro] RL loop with observations as textual prompts: "we design the classic RL loop by interfacing the simulation environment (KSDPG) with a LLM, transforming the real-time observations (or state) of the mission as textual user prompts"
- [readme] KSPDG provides differential-game environments and non-blocking step function: [evidence span withheld — anchor status: not_found]
