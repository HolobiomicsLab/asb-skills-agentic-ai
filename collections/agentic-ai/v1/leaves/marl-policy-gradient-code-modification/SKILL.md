---
name: marl-policy-gradient-code-modification
description: Use when a sequential-update MARL codebase with independent actors shows
  high/unstable advantage variance, and you need to replace the unclipped importance-sampled
  advantage with a variance-bounded clipped objective.
when_to_use_negative:
- The environment already uses centralized critics (e.g. MAPPO-style joint policy
  optimization) rather than independent actors with sequential updates — the variance
  analysis and clipping operator are derived specifically for the independent-actor
  sequential update setting.
- The variance issue stems from training/trust-region setup rather than the advantage
  estimator — the paper notes clipping 'is not the only way to reduce variance' and
  that HATRPO is less susceptible to high variance than HAPPO without modification.
- The target codebase requires a tunable per-agent clipping threshold — the proposed
  method fixes the threshold at 1 for all agents, which may be pessimistic in environments
  with less severe variance issues.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: HAPPO
  role: Baseline sequential-update PPO-style algorithm whose advantage recomputation
    step is modified with the min(R,1) clipped ratio to produce clip-HAPPO
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: HATRPO
  role: Baseline trust-region algorithm whose surrogate objective receives the same
    clipped ratio plus reverse-order KL penalty to produce clip-HATRPO
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: clip-HAPPO
  role: 'Derived algorithm produced by this skill: clipped surrogate loss implemented
    in clip_objective.py'
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: clip-HATRPO
  role: 'Derived algorithm produced by this skill: clipped ratio applied to HATRPO''s
    surrogate with KL penalized in expectation of the discounted marginal state distribution'
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: pip
  role: Installs the codebase in editable mode (`pip install -e .`) and pinned dependencies
    (gym==0.21.0, pettingzoo==1.22.2, supersuit==3.7.0)
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s10458-025-09695-8
    title: Low variance trust region optimization with independent actors and sequential
      updates in cooperative multi-agent reinforcement learning
  source_content_hash: sha256:0ec377c2577583f6dd300e75052220533468db797a1cf30b98b77c6b751a3241
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# marl-policy-gradient-code-modification

## Summary

A skill for modifying sequential-update trust-region MARL actor code (HAPPO/HATRPO) by wrapping the joint importance ratio in the clipping operator min(R, 1), yielding the low-variance clip-HAPPO and clip-HATRPO objectives as drop-in replacements.

## When to use

Use when a sequential-update MARL codebase with independent actors shows high/unstable advantage variance, and you need to replace the unclipped importance-sampled advantage with a variance-bounded clipped objective.

## When NOT to use

- The environment already uses centralized critics (e.g. MAPPO-style joint policy optimization) rather than independent actors with sequential updates — the variance analysis and clipping operator are derived specifically for the independent-actor sequential update setting.
- The variance issue stems from training/trust-region setup rather than the advantage estimator — the paper notes clipping 'is not the only way to reduce variance' and that HATRPO is less susceptible to high variance than HAPPO without modification.
- The target codebase requires a tunable per-agent clipping threshold — the proposed method fixes the threshold at 1 for all agents, which may be pessimistic in environments with less severe variance issues.

## Inputs

- HAPPO/HATRPO actor-update source code from the Low-Variance-Trust-Region-MARL repository (pip install -e .)
- Sequential-update trust region objective with unclipped joint importance ratio R_{1:m-1}
- Training config file from run_configs (e.g. for SMAC, MPE, or MAMuJoCo benchmarks)

## Outputs

- clip_objective.py implementing E[(pi_hat_m/pi_m - 1) * min(R_{1:m-1}, 1) * A_pi] as a drop-in surrogate loss
- clip-HAPPO and clip-HATRPO actor updates with the min(R,1) clipped ratio and reverse-order KL penalty
- A runnable training invocation via python -m examples.train --load_config <CONFIG PATH>

## How to apply

Install the paper's repository (https://github.com/giangbang/Low-Variance-Trust-Region-MARL) with `pip install -e .` and locate the HAPPO/HATRPO actor-update code implementing the sequential update trust region objective. In HAPPO's advantage recomputation step, wrap the joint importance ratio of the first m-1 agents in min(R, 1) per Definition 4, keeping the clipping threshold fixed at 1 so no new hyperparameter is introduced. Implement the clipped surrogate E[(pi_hat_m/pi_m - 1) * min(R_{1:m-1}, 1) * A_pi] in clip_objective.py as a drop-in replacement for the unclipped ratio term (clip-HAPPO), then apply the same clipped ratio to HATRPO's surrogate while enforcing Algorithm 1's two structural details: the KL trust-region term penalized in expectation of the discounted marginal state distribution, and reverse-order KL (new policy as reference) to obtain clip-HATRPO. The rationale is that clipping caps the advantage fluctuation upper bound across agents' sequential updates, so Var[min(R,1)A] <= E[A^2] <= epsilon^2, independent of the number of agents.

## Related tools

- **HAPPO** (Baseline sequential-update PPO-style algorithm whose advantage recomputation step is modified with the min(R,1) clipped ratio to produce clip-HAPPO) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **HATRPO** (Baseline trust-region algorithm whose surrogate objective receives the same clipped ratio plus reverse-order KL penalty to produce clip-HATRPO) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **clip-HAPPO** (Derived algorithm produced by this skill: clipped surrogate loss implemented in clip_objective.py) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **clip-HATRPO** (Derived algorithm produced by this skill: clipped ratio applied to HATRPO's surrogate with KL penalized in expectation of the discounted marginal state distribution) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **pip** (Installs the codebase in editable mode (`pip install -e .`) and pinned dependencies (gym==0.21.0, pettingzoo==1.22.2, supersuit==3.7.0)) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL

## Examples

```
pip install -e . ; python -m examples.train --load_config <CONFIG PATH>
```

## Evaluation signals

- The implemented objective contains the min(R, 1) clipped joint importance ratio with the clipping threshold fixed at 1 (no new hyperparameter).
- Var[min(R,1)A] <= E[A^2] <= epsilon^2 holds for any state and does not grow with the number of agents.
- HATRPO variant uses reverse-order KL (new policy as reference) and penalizes the KL term in expectation of the discounted marginal state distribution, matching Algorithm 1.
- A smoke test importing clip_objective.py succeeds and a short training invocation `python -m examples.train --load_config <CONFIG PATH>` runs without error.
- Training shows stable convergence and low advantage variance estimation relative to the unclipped HAPPO baseline across benchmark settings.

## Limitations

- The exponential variance growth derived in the paper is only an upper bound (pessimistic); in practice growth depends on environment types and hyperparameters and does not always follow the worst case.
- Fixing the clipping threshold to 1 for all agents may be pessimistic in environments with less severe variance issues, and the clipped objective could potentially dampen the exploration signal within the joint advantage.
- The clipping objective is not the only way to reduce variance; training and trust region setups also matter, which is why HATRPO is less susceptible to high variance than HAPPO.

## Evidence

- [intro] clipped surrogate loss formula: "we derive two new algorithms clip-HAPPO and clip-HATRPO"
- [methods] variance bound: [evidence span withheld — anchor status: not_found]
