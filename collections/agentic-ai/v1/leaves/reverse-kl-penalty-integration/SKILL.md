---
name: reverse-kl-penalty-integration
description: Use when implementing or porting sequential-update trust-region MARL
  actor losses (HAPPO/HATRPO) where the KL trust-region penalty must be taken in reverse
  order — new policy as reference — alongside a clipped joint importance ratio.
when_to_use_negative:
- When the algorithm is not a sequential-update trust-region method with independent
  actors (e.g. centralized-critic methods like MAPPO) — the reverse-KL + min(R,1)
  structure of Algorithm 1 does not transfer
- 'When a separate clipping hyperparameter is desired: the paper deliberately fixes
  the clipping threshold at 1 for all agents, which the authors note may be pessimistic
  in environments with less severe variance issues'
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: clip-HAPPO
  role: 'Target algorithm: HAPPO actor update augmented with the min(R,1) clipped
    ratio and reverse-order KL penalty from Algorithm 1'
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: clip-HATRPO
  role: 'Target algorithm: HATRPO surrogate with the same clipped ratio, penalized
    KL in expectation of the discounted marginal state distribution, and reverse-order
    KL (new policy as reference)'
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: pip
  role: Install the codebase in editable mode with pip install -e . before modifying
    actor-update code
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s10458-025-09695-8
    title: Low variance trust region optimization with independent actors and sequential
      updates in cooperative multi-agent reinforcement learning
  source_content_hash: sha256:36b2a955aa061293b8c88d080db9ef3e99799509a4982cf55ead5d4e85eeedc9
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
yes_rate: 0.9230769230769231
n_claims: 13
---

# reverse-kl-penalty-integration

## Summary

Implements the reverse-order KL divergence penalty (new policy as reference) required by Algorithm 1 of the low-variance trust-region MARL paper, combined with the min(R,1)-clipped importance ratio, to yield the clip-HAPPO and clip-HATRPO actor updates with bounded advantage variance.

## When to use

Use when implementing or porting sequential-update trust-region MARL actor losses (HAPPO/HATRPO) where the KL trust-region penalty must be taken in reverse order — new policy as reference — alongside a clipped joint importance ratio.

## When NOT to use

- When the algorithm is not a sequential-update trust-region method with independent actors (e.g. centralized-critic methods like MAPPO) — the reverse-KL + min(R,1) structure of Algorithm 1 does not transfer
- When a separate clipping hyperparameter is desired: the paper deliberately fixes the clipping threshold at 1 for all agents, which the authors note may be pessimistic in environments with less severe variance issues

## Inputs

- HAPPO/HATRPO actor-update source code from giangbang/Low-Variance-Trust-Region-MARL
- Per-agent sequential-update joint importance ratios R_{1:m-1} and advantages A_pi
- Training config YAML files from run_configs (e.g. for SMAC, MPE, MAMuJoCo environments)

## Outputs

- clip_objective.py implementing the clipped surrogate loss with min(R,1) ratio and reverse-order KL penalty
- clip-HAPPO and clip-HATRPO actor updates usable as drop-in replacements in HARL
- Training runs reproducible via python -m examples.train --load_config

## How to apply

Locate the HAPPO/HATRPO actor-update code in the paper's repository (giangbang/Low-Variance-Trust-Region-MARL, install with `pip install -e .`). For each agent m in the sequential update, wrap the joint importance ratio of the first m-1 agents in min(R, 1) per Definition 4, keeping the clipping threshold fixed at 1 so no new hyperparameter is introduced, and implement the surrogate E[(pi_hat_m/pi_m - 1) * min(R_{1:m-1}, 1) * A_pi] as a drop-in replacement (e.g. in clip_objective.py). For the trust-region variant (clip-HATRPO), enforce the two structural details of Algorithm 1: the KL divergence is computed in reverse order, with the NEW policy as the reference distribution, and the KL term is penalized in expectation over the discounted marginal state distribution rather than the trajectory distribution. The rationale is that the min(R,1) clip bounds Var[min(R,1)A] <= E[A^2] <= epsilon^2 independent of the number of agents, while the reverse KL keeps the update inside the trust region defined by the updated (newer) policy. Validate by importing clip_objective.py and running a short training invocation via `python -m examples.train --load_config <CONFIG PATH>` from run_configs.

## Related tools

- **clip-HAPPO** (Target algorithm: HAPPO actor update augmented with the min(R,1) clipped ratio and reverse-order KL penalty from Algorithm 1) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **clip-HATRPO** (Target algorithm: HATRPO surrogate with the same clipped ratio, penalized KL in expectation of the discounted marginal state distribution, and reverse-order KL (new policy as reference)) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **pip** (Install the codebase in editable mode with pip install -e . before modifying actor-update code) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL

## Examples

```
pip install -e . && python -m examples.train --load_config run_configs/<CONFIG PATH>
```

## Evaluation signals

- The implemented objective contains the min(R,1) clipped joint importance ratio with the clipping threshold fixed at 1 (no new hyperparameter)
- The KL divergence in the trust-region term is in reverse order: the new (updated) policy serves as the reference distribution
- The KL penalty is taken in expectation of the discounted marginal state distribution, matching Algorithm 1's two structural details
- Variance check: Var[min(R,1)A] <= E[A^2] <= epsilon^2 holds and does not grow with the number of agents
- Smoke test: `python -m examples.train --load_config <CONFIG PATH>` runs without error after the modification

## Limitations

- Fixing the clipping threshold to 1 for all agents may be pessimistic in environments with less severe variance issues; a natural extension is to vary the clipping threshold
- The clipped objective could potentially dampen the exploration signal within the joint advantage, and estimating the exploration effect is challenging in practice
- The exponential variance growth analysis is only a pessimistic upper bound; in practice growth depends on environment types and hyperparameters (HATRPO is less susceptible than HAPPO)

## Evidence

- [other] Reverse-order KL is a structural requirement of Algorithm 1 for clip-HATRPO: [evidence span withheld — anchor status: not_found]
- [methods] Variance bounding by the clipped ratio with threshold 1: [evidence span withheld — anchor status: not_found]
- [other] Clipping operator definition with fixed threshold introducing no hyperparameter: [evidence span withheld — anchor status: not_found]
