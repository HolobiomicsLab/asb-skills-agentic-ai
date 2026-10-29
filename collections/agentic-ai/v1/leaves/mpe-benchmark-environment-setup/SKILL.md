---
name: mpe-benchmark-environment-setup
description: Use when reproducing MARL experiments on the MPE benchmark with the Low-Variance-Trust-Region-MARL
  repo and the PettingZoo/SuperSuit environment stack must be installed with exact
  pinned versions.
when_to_use_negative:
- The experiment targets SMAC (StarCraft Multi-Agent Challenge) instead — SMAC requires
  following the official oxwhirl/smac installation instructions and downloading StarCraft
  II and SMAC Maps, not the pettingzoo/supersuit stack.
- The experiment targets MuJoCo or MAMuJoCo benchmarks — those require downloading
  MuJoCo binaries to ~/.mujoco, setting LD_LIBRARY_PATH, and installing mujoco-py/Jinja2/glfw/Cython
  pins.
- The task is a discrete or non-continuous MPE scenario outside the three continuous
  tasks (simple_spread_v2, simple_reference_v2, simple_speaker_listener_v3) used in
  the paper's Figure-4-style evaluation.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: pip
  role: Package manager used to install the codebase ('pip install -e .') and the
    pinned environment dependencies (pettingzoo==1.22.2, supersuit==3.7.0, gym==0.21.0,
    pyglet==1.5.0, importlib-metadata==4.13.0).
- name: pettingzoo 1.22.2
  role: Provides the MPE multi-agent environment API through which the three continuous
    benchmark tasks are instantiated.
- name: supersuit 3.7.0
  role: PettingZoo environment wrapper library pinned alongside pettingzoo to make
    MPE scenarios compatible with the training runner.
- name: Low-Variance-Trust-Region-MARL
  role: The codebase being installed; supplies examples.train entrypoint, RUNNER_REGISTRY,
    and run_configs folder used to validate the MPE setup.
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s10458-025-09695-8
    title: Low variance trust region optimization with independent actors and sequential
      updates in cooperative multi-agent reinforcement learning
  source_content_hash: sha256:b4cc899804148b576ddb8c8cc196d08684eb538b61337d8249033ade088d5fec
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
  tools:
  - HAPPO
  - HATRPO
  - Low-Variance-Trust-Region-MARL
  - MAPPO
  - clip-HAPPO
  - clip-HATRPO
  - pettingzoo
  - pettingzoo 1.22.2
  - pip
  - supersuit
  - supersuit 3.7.0
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# mpe-benchmark-environment-setup

## Summary

Installs the pinned Python dependencies (pettingzoo==1.22.2, supersuit==3.7.0, gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0) needed to run the three continuous MPE tasks (simple_spread_v2, simple_reference_v2, simple_speaker_listener_v3) from the Low-Variance-Trust-Region-MARL codebase.

## When to use

Use when reproducing MARL experiments on the MPE benchmark with the Low-Variance-Trust-Region-MARL repo and the PettingZoo/SuperSuit environment stack must be installed with exact pinned versions.

## When NOT to use

- The experiment targets SMAC (StarCraft Multi-Agent Challenge) instead — SMAC requires following the official oxwhirl/smac installation instructions and downloading StarCraft II and SMAC Maps, not the pettingzoo/supersuit stack.
- The experiment targets MuJoCo or MAMuJoCo benchmarks — those require downloading MuJoCo binaries to ~/.mujoco, setting LD_LIBRARY_PATH, and installing mujoco-py/Jinja2/glfw/Cython pins.
- The task is a discrete or non-continuous MPE scenario outside the three continuous tasks (simple_spread_v2, simple_reference_v2, simple_speaker_listener_v3) used in the paper's Figure-4-style evaluation.

## Inputs

- Cloned Low-Variance-Trust-Region-MARL repository (https://github.com/giangbang/Low-Variance-Trust-Region-MARL)
- Python environment with pip
- Config files from the repo's run_configs folder for MPE tasks (e.g. simple_spread_v2, simple_reference_v2, simple_speaker_listener_v3)

## Outputs

- Working MPE environment stack (pettingzoo 1.22.2 + supersuit 3.7.0 + gym 0.21.0) importable by the training entrypoint
- Successful launch of a training run producing episode-reward logs for one algorithm-task pair

## How to apply

First install the codebase itself with 'pip install -e .' from the cloned Low-Variance-Trust-Region-MARL repository. Then install the MPE environment stack with the exact pinned versions: pip install pettingzoo==1.22.2 and pip install supersuit==3.7.0. After that, resolve remaining dependency conflicts with pip install gym==0.21.0, pip install pyglet==1.5.0, and pip install importlib-metadata==4.13.0, as the MPE scenarios load through PettingZoo wrappers that require these older pins. Verify the setup by launching a training run via 'python -m examples.train --load_config <CONFIG PATH>' using a config from the run_configs folder for one of the three continuous MPE tasks, which constructs the runner through RUNNER_REGISTRY[args["algo"]] and executes runner.run() until the training horizon completes.

## Related tools

- **pip** (Package manager used to install the codebase ('pip install -e .') and the pinned environment dependencies (pettingzoo==1.22.2, supersuit==3.7.0, gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0).)
- **pettingzoo 1.22.2** (Provides the MPE multi-agent environment API through which the three continuous benchmark tasks are instantiated.)
- **supersuit 3.7.0** (PettingZoo environment wrapper library pinned alongside pettingzoo to make MPE scenarios compatible with the training runner.)
- **Low-Variance-Trust-Region-MARL** (The codebase being installed; supplies examples.train entrypoint, RUNNER_REGISTRY, and run_configs folder used to validate the MPE setup.) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL

## Examples

```
pip install -e . && pip install pettingzoo==1.22.2 && pip install supersuit==3.7.0 && pip install gym==0.21.0 && pip install pyglet==1.5.0 && pip install importlib-metadata==4.13.0
```

## Evaluation signals

- pip install -e . completes without errors and the package is importable.
- pip install pettingzoo==1.22.2 and pip install supersuit==3.7.0 resolve to exactly those pinned versions (verify with pip show pettingzoo / pip show supersuit).
- The dependency-resolution pins (gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0) install cleanly, indicating no version conflicts remain.
- 'python -m examples.train --load_config <MPE CONFIG PATH>' starts a run: RUNNER_REGISTRY[args["algo"]] constructs the runner and runner.run() begins producing episode-reward logs for the chosen MPE task.
- For the reproduction goal, training runs across five algorithms (MAPPO, HATRPO, HAPPO, clip-HATRPO, clip-HAPPO) on all three continuous MPE tasks complete and yield five reward-versus-iteration curves per task.

## Limitations

- The exact version pins (pettingzoo==1.22.2, supersuit==3.7.0, gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0) are required; newer or unpinned versions may break the MPE environment loading.
- This setup covers only the MPE benchmark; SMAC and MuJoCo/MAMuJoCo benchmarks in the same repo require entirely different, benchmark-specific installation procedures.
- Version pins imply an older Python ecosystem; the gym==0.21.0 pin is known to be sensitive to pip/setuptools versions in practice (installation may need 'pip install --upgrade pip').

## Evidence
