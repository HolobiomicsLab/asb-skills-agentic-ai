---
name: trust-region-surrogate-loss-implementation
description: Use when a sequential-update trust-region MARL trainer (HAPPO/HATRPO
  with independent actors) shows unstable convergence or exponentially growing advantage-estimator
  variance as agent count increases, and you need a low-variance clipped replacement
  for the unclipped importance ratio in the actor.
when_to_use_negative:
- The environment/training setup already keeps advantage-estimate variance low (the
  paper notes HATRPO is less susceptible to high variance than HAPPO, and the fixed
  threshold of 1 may be pessimistic in less severe settings).
- You need to preserve the exploration signal within the joint advantage — the authors
  caution the clipped objective could potentially dampen exploration and suggest varying
  the clipping threshold as future work.
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: HAPPO
  role: Baseline sequential-update trust-region algorithm whose advantage recomputation
    is modified with the min(R,1) clipped ratio to produce clip-HAPPO
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: HATRPO
  role: Baseline whose surrogate objective receives the same clipped ratio plus reverse-order
    KL penalty and KL penalized in expectation over the discounted marginal state
    distribution, yielding clip-HATRPO
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: pip
  role: Used to install the codebase (`pip install -e .`) and pin dependency versions
    (gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s10458-025-09695-8
    title: Low variance trust region optimization with independent actors and sequential
      updates in cooperative multi-agent reinforcement learning
  source_content_hash: sha256:a3ee8c6489c4a554433bcd5818ad1f54ed13e87d7d2207ccc3d141d6c1883a15
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
  tools:
  - HAPPO
  - HATRPO
  - clip-HAPPO
  - clip-HATRPO
  - pip
  edam_operation: http://edamontology.org/operation_3946
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# trust-region-surrogate-loss-implementation

## Summary

Implement the clipped sequential trust-region surrogate loss (clip-HAPPO and clip-HATRPO) as a drop-in HARL actor update that wraps the joint importance ratio in min(R, 1), bounding the advantage-estimator variance by epsilon squared instead of letting it grow exponentially with the number of agents.

## When to use

Use when a sequential-update trust-region MARL trainer (HAPPO/HATRPO with independent actors) shows unstable convergence or exponentially growing advantage-estimator variance as agent count increases, and you need a low-variance clipped replacement for the unclipped importance ratio in the actor update.

## When NOT to use

- The environment/training setup already keeps advantage-estimate variance low (the paper notes HATRPO is less susceptible to high variance than HAPPO, and the fixed threshold of 1 may be pessimistic in less severe settings).
- You need to preserve the exploration signal within the joint advantage — the authors caution the clipped objective could potentially dampen exploration and suggest varying the clipping threshold as future work.

## Inputs

- HAPPO actor-update code from the Low-Variance-Trust-Region-MARL repository (HARL-style)
- HATRPO actor-update code from the same repository
- An experiment config file from run_configs (e.g. for SMAC, MPE, or MAMuJoCo)

## Outputs

- clip_objective.py implementing the clipped surrogate E[(pi_hat_m/pi_m - 1) * min(R_{1:m-1}, 1) * A_pi]
- clip-HAPPO and clip-HATRPO actor updates with the min(R,1) clipped importance ratio and reverse-order KL penalty
- A short training run completing without error via examples.train

## How to apply

Install the paper's repository (https://github.com/giangbang/Low-Variance-Trust-Region-MARL) with `pip install -e .` and locate the HAPPO/HATRPO actor-update code implementing the sequential update trust-region objective. In HAPPO's advantage recomputation step, wrap the joint importance ratio of the first m-1 agents in the clipping operator min(R, 1) per Definition 4, keeping the clipping threshold fixed at 1 so no new hyperparameter is introduced. Implement the clipped surrogate loss E[(pi_hat_m/pi_m - 1) * min(R_{1:m-1}, 1) * A_pi] as a drop-in replacement for the unclipped ratio term to yield clip-HAPPO; apply the same clipped ratio to HATRPO's surrogate objective and enforce Algorithm 1's two structural details — the trust-region KL term is penalized in expectation over the discounted marginal state distribution and the KL is in reverse order (new policy as reference) — to yield clip-HATRPO. Rationale: clipping with min(R, 1) gives Var[min(R,1)A] <= E[A^2] <= epsilon^2 independent of the number of agents, and the paper reports sub-linear convergence to epsilon-Nash equilibria and stable convergence on SMAC, MPE, and MAMuJoCo benchmarks.

## Related tools

- **HAPPO** (Baseline sequential-update trust-region algorithm whose advantage recomputation is modified with the min(R,1) clipped ratio to produce clip-HAPPO) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **HATRPO** (Baseline whose surrogate objective receives the same clipped ratio plus reverse-order KL penalty and KL penalized in expectation over the discounted marginal state distribution, yielding clip-HATRPO) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **pip** (Used to install the codebase (`pip install -e .`) and pin dependency versions (gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0))

## Examples

```
pip install -e . && python -m examples.train --load_config <CONFIG PATH>
```

## Evaluation signals

- The implemented objective contains the min(R,1) clipped importance ratio of the first m-1 agents' joint policy, not the raw ratio R_{1:m-1}.
- The trust-region KL term is computed in reverse order (new policy as reference) and penalized in expectation over the discounted marginal state distribution, per Algorithm 1.
- Variance check: Var[min(R,1)A] <= E[A^2] <= epsilon^2 holds for any state and does not grow with the number of agents.
- Smoke test: `python -m examples.train --load_config <CONFIG PATH>` from run_configs runs without error after the change.

## Limitations

- The exponential variance growth derived in the paper is only an upper bound and pessimistic; actual growth depends on environment types and hyperparameters and does not always follow the worst case.
- Fixing the clipping threshold at 1 for all agents may appear pessimistic in environments with less severe variance issues; varying the threshold is proposed as a natural extension.
- The clipped objective could potentially dampen the exploration signal within the joint advantage, and estimating the exploration effect is challenging in practice.

## Evidence

- [methods] clipped surrogate loss in clip_objective.py: [evidence span withheld — anchor status: not_found]
- [methods] variance bound: [evidence span withheld — anchor status: not_found]
