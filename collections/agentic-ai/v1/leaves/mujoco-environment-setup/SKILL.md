---
name: mujoco-environment-setup
description: Use when a reproduction task requires running the trust region MARL codebase
  on MuJoCo/MAMuJoCo multi-agent physics environments (e.g. Ant-v2-8x1, Humanoid-v2-17x1)
  and the environment binaries and Python bindings are not yet installed.
when_to_use_negative:
- The target environments are SMAC (StarCraft Multi-Agent Challenge) or MPE — those
  need the oxwhirl/smac instructions or pettingzoo==1.22.2 / supersuit==3.7.0 instead
  of MuJoCo setup.
- The workflow does not require physics simulation (e.g. only analyzing logged reward/variance
  curves from prior runs).
- Working on Windows, since mujoco-py has deprecated and removed Windows support.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: MuJoCo
  role: Physics engine providing the simulation backend for MAMuJoCo multi-agent scenarios;
    extracted to ~/.mujoco with LD_LIBRARY_PATH set
  repo: https://github.com/deepmind/mujoco
- name: mujoco-py
  role: Deprecated Python bindings compatible with MuJoCo 2.1, installed at >=2.1.2.14
    for MAMuJoCo
  repo: https://github.com/openai/mujoco-py
- name: pip
  role: Used for 'pip install -e .' of the codebase and for all pinned dependency
    installs (gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0, mujoco-py>=2.1.2.14)
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: MAMuJoCo
  role: Multi-agent MuJoCo benchmark whose dependencies are installed after base MuJoCo
    setup
provenance:
  source_task_ids:
  - task_007
  - task_001
  - task_008
  - task_003
  source_papers:
  - doi: 10.1007/s10458-025-09695-8
    title: Low variance trust region optimization with independent actors and sequential
      updates in cooperative multi-agent reinforcement learning
  source_content_hash: sha256:b047f7b5a4396a1927807a93e0d71634d6a2eade38b5e8cd2ebd9365cc50ad33
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/deepmind/mujoco
  tools:
  - HAPPO
  - HATRPO
  - MAMuJoCo
  - MAPPO
  - MuJoCo
  - clip-HAPPO
  - clip-HATRPO
  - mujoco-py
  - pip
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# mujoco-environment-setup

## Summary

Sets up the MuJoCo physics engine and its MAMuJoCo dependencies (mujoco-py>=2.1.2.14, Jinja2>=3.0.3, glfw>=2.5.1, Cython>=0.29.28) so the Low-Variance-Trust-Region-MARL codebase can train HAPPO/clip-HAPPO on MAMuJoCo scenarios.

## When to use

Use when a reproduction task requires running the trust region MARL codebase on MuJoCo/MAMuJoCo multi-agent physics environments (e.g. Ant-v2-8x1, Humanoid-v2-17x1) and the environment binaries and Python bindings are not yet installed.

## When NOT to use

- The target environments are SMAC (StarCraft Multi-Agent Challenge) or MPE — those need the oxwhirl/smac instructions or pettingzoo==1.22.2 / supersuit==3.7.0 instead of MuJoCo setup.
- The workflow does not require physics simulation (e.g. only analyzing logged reward/variance curves from prior runs).
- Working on Windows, since mujoco-py has deprecated and removed Windows support.

## Inputs

- Cloned repository giangbang/Low-Variance-Trust-Region-MARL
- MuJoCo engine archive (.tar.gz/.zip) from roboti.us or deepmind/mujoco
- Linux shell with sudo apt available

## Outputs

- Working MuJoCo installation under ~/.mujoco with LD_LIBRARY_PATH configured
- pip-installed Python environment with mujoco-py>=2.1.2.14, gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0
- Runnable 'python -m examples.train --load_config <CONFIG PATH>' entry point

## How to apply

Install the cloned repository with 'pip install -e .', then install MuJoCo by downloading the engine from deepmind/mujoco or roboti.us, extracting the archive into ~/.mujoco, and exporting LD_LIBRARY_PATH pointing to the extracted bin folder (e.g. mujoco210). Next install the MAMuJoCo-specific dependencies via pinned pip versions: 'mujoco-py>=2.1.2.14', 'Jinja2>=3.0.3', 'glfw>=2.5.1', 'Cython>=0.29.28', plus system libraries libosmesa6-dev, libgl1-mesa-glx, libglfw3 and patchelf. Finally resolve the remaining pinned dependencies gym==0.21.0, pyglet==1.5.0, and importlib-metadata==4.13.0, which the README mandates to avoid version conflicts. The order matters: MuJoCo must be extracted and LD_LIBRARY_PATH set before mujoco-py, since mujoco-py compiles against the engine at import time.

## Related tools

- **MuJoCo** (Physics engine providing the simulation backend for MAMuJoCo multi-agent scenarios; extracted to ~/.mujoco with LD_LIBRARY_PATH set) — https://github.com/deepmind/mujoco
- **mujoco-py** (Deprecated Python bindings compatible with MuJoCo 2.1, installed at >=2.1.2.14 for MAMuJoCo) — https://github.com/openai/mujoco-py
- **pip** (Used for 'pip install -e .' of the codebase and for all pinned dependency installs (gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0, mujoco-py>=2.1.2.14)) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **MAMuJoCo** (Multi-agent MuJoCo benchmark whose dependencies are installed after base MuJoCo setup)

## Examples

```
pip install -e . && pip install "mujoco-py>=2.1.2.14" "Jinja2>=3.0.3" "glfw>=2.5.1" "Cython>=0.29.28" && pip install gym==0.21.0 pyglet==1.5.0 importlib-metadata==4.13.0
```

## Evaluation signals

- LD_LIBRARY_PATH exports the bin directory of the extracted MuJoCo folder (e.g. ~/.mujoco/mujoco210/bin) before any mujoco-py import.
- All four MAMuJoCo pip constraints succeed: mujoco-py>=2.1.2.14, Jinja2>=3.0.3, glfw>=2.5.1, Cython>=0.29.28.
- Final pinned dependency resolution succeeds without conflicts: gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0.
- 'python -m examples.train --load_config <CONFIG PATH>' starts a training run (runner.run() reached) on a MAMuJoCo scenario without ImportError or mujoco-py compilation failure.
- System packages libosmesa6-dev, libgl1-mesa-glx, libglfw3, and patchelf are installed, as the README requires for rendering/patching.

## Limitations

- mujoco-py is deprecated and does not support MuJoCo versions after 2.1.0; new users are pointed to the official DeepMind Python bindings, but this codebase pins mujoco-py>=2.1.2.14.
- The setup requires downloading the correct MuJoCo version manually from roboti.us or deepmind/mujoco; a wrong engine version breaks the pinned mujoco-py build.
- Windows support was removed from mujoco-py (deprecated in 2.0.2.0), so the workflow assumes Linux with apt and sudo.

## Evidence

pip install "Jinja2>=3.0.3"
pip install "glfw>=2.5.1"
pip install "Cython>=0.29.28""
