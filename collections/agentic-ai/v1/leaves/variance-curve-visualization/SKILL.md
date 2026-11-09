---
name: variance-curve-visualization
description: Use when comparing sequential-update MARL algorithms (HAPPO vs clip-HAPPO/HATRPO)
  and you need to visualize whether advantage-estimate variance grows with agent update
  position across environments like MAMuJoCo tasks.
when_to_use_negative:
- When the analysis goal is return/reward learning curves rather than advantage-estimate
  variance diagnostics
- When the algorithm uses simultaneous (non-sequential) agent updates, where per-agent
  update-order variance is not defined
- When environments have very few agents (e.g. 2-3), where exponential growth is hard
  to distinguish from noise and the curves are less informative
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: HAPPO
  role: Baseline sequential-update algorithm whose unclipped ratio-product advantage
    estimates are logged and plotted to show exponential variance growth
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: clip-HAPPO
  role: Clipped alternative whose advantage estimates min(ℜ_{1:m-1}, 1) * A^π(s,a)
    are logged and plotted to show variance bounded near ϵ²
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: pip
  role: Installs the Low-Variance-Trust-Region-MARL codebase (pip install -e .) and
    pinned dependencies (gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0) needed
    to run the training that produces the variance logs
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s10458-025-09695-8
    title: Low variance trust region optimization with independent actors and sequential
      updates in cooperative multi-agent reinforcement learning
  source_content_hash: sha256:1db2d459325e12009af885927f48cc12dca706257b4b59bdd6f0b0e0c001e306
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
  tools:
  - HAPPO
  - clip-HAPPO
  - pip
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# variance-curve-visualization

## Summary

Plot per-agent advantage-estimate variance against update order to expose the exponential variance growth of sequential-update trust region methods (HAPPO) versus the flat, bounded variance of a clipped alternative (clip-HAPPO).

## When to use

Use when comparing sequential-update MARL algorithms (HAPPO vs clip-HAPPO/HATRPO) and you need to visualize whether advantage-estimate variance grows with agent update position across environments like MAMuJoCo tasks.

## When NOT to use

- When the analysis goal is return/reward learning curves rather than advantage-estimate variance diagnostics
- When the algorithm uses simultaneous (non-sequential) agent updates, where per-agent update-order variance is not defined
- When environments have very few agents (e.g. 2-3), where exponential growth is hard to distinguish from noise and the curves are less informative

## Inputs

- Sequential per-agent advantage estimates logged during HAPPO and clip-HAPPO training runs (per agent m, per timestep)
- Training configs from the run_configs folder (YAML files) matching each MAMuJoCo environment
- Update-order metadata mapping each logged advantage to its agent position in the sequential update

## Outputs

- Per-agent mean and maximum variance statistics over the update order for HAPPO and clip-HAPPO
- Variance-vs-agent-index curves per environment (reproducing Figures 5-6), showing HAPPO growth to ~1e1-1e2 and clip-HAPPO flat near ~1

## How to apply

During sequential per-agent updates, log the advantage estimate for each agent m computed under the partially updated joint policy: min(ℜ_{1:m-1}, 1) * A^π(s,a) for clip-HAPPO versus the unclipped ratio product for HAPPO. Aggregate the logged values into per-agent mean and maximum variance statistics over the update order for both algorithms in each environment (e.g. Ant-v2-8x1, Half Cheetah-v2-6x1, Manyagent Swimmer-v2-10x2, Walker2d-v2-6x1). Then plot variance versus agent index/update order for both algorithms, reproducing Figures 5-6 of the paper where HAPPO variance grows to the ~1e1-1e2 range (reaching ~1e6 for Humanoid-v2-17x1) while clip-HAPPO stays near 1. The rationale is that the clipped objective bounds Var ≤ ϵ², so its variance curve should be approximately flat, providing direct empirical evidence for the exponential upper-bound analysis.

## Related tools

- **HAPPO** (Baseline sequential-update algorithm whose unclipped ratio-product advantage estimates are logged and plotted to show exponential variance growth) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **clip-HAPPO** (Clipped alternative whose advantage estimates min(ℜ_{1:m-1}, 1) * A^π(s,a) are logged and plotted to show variance bounded near ϵ²) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **pip** (Installs the Low-Variance-Trust-Region-MARL codebase (pip install -e .) and pinned dependencies (gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0) needed to run the training that produces the variance logs) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL

## Examples

```
python -m examples.train --load_config run_configs/mamujoco/happo_Ant-v2-8x1.yaml
```

## Evaluation signals

- clip-HAPPO variance curves remain approximately flat near 1, consistent with the theoretical bound Var ≤ ϵ²
- HAPPO variance reaches the ~1e1-1e2 range as agent update position increases, matching reported figure trends
- For Humanoid-v2-17x1, HAPPO advantage variance reaches values as high as 1e6 during training
- Variance is monotonically increasing (or nearly so) with agent index for HAPPO across all four MAMuJoCo environments
- Per-agent statistics are aggregated separately for mean and maximum variance so both trend and worst case are visible

## Limitations

- The exponential growth derived in the paper is only an upper bound and a pessimistic point of view; in practice growth depends on environment types and hyperparameters and does not always follow the worst case
- Fixing the clipping threshold to 1 for all agents may appear pessimistic in environments with less severe variance issues
- The clipped objective could potentially dampen the exploration signal within the joint advantage, complicating interpretation of low variance as purely beneficial

## Evidence

- [methods] clip-HAPPO variance bound: [evidence span withheld — anchor status: not_found]
