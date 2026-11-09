---
name: service-registration-configuration
description: 'Use when deploying AudioFab or adding a new tool: after configuring
  per-service environments, each MCP service must be registered in servers_config.
  before scripts/check.sh validation and launch.'
when_to_use_negative:
- If a tool/service requiring local model deployment (FunTTS_mcp_servers, music_mcp_servers,
  Audioseparator_mcp_servers) has not been locally deployed — its registration must
  be removed instead of configured
- If you only want a quick API-based experience — configure MD_TOKEN and HF_TOKEN
  in servers/API_servers.py rather than deploying per-service local registrations
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3489
tools:
- name: AudioFab
  role: Framework whose MCP services are registered and launched; provides scripts/check.sh
    and scripts/start_all.py
  repo: https://github.com/SmileHnu/AudioFab
- name: Conda
  role: Creates the isolated per-service virtual environments whose interpreter paths
    are referenced as the 'command' for each registered service
- name: Python
  role: Interpreter invoked per service via the 'command' field; runs the service
    scripts listed in args
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1145/3746027.3756869
    title: 'AudioFab: Building A General and Intelligent Audio Factory through Tool
      Learning'
  source_content_hash: sha256:4030c1f4b7b579e5569fcd6a9f45fed931618b238facbeafb312ca1e8571655d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/SmileHnu/AudioFab
  tools:
  - AudioFab
  - Conda
  - Python
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3489
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# Service Registration Configuration

## Summary

Register each AudioFab MCP service in mcp_servers/servers_config. by specifying the per-service Python interpreter command, the service script path in args, and the PYTHONPATH environment variable, so every tool runs in its own isolated dependency environment (Conda or Docker) without conflicts.

## When to use

Use when deploying AudioFab or adding a new tool: after configuring per-service environments, each MCP service must be registered in servers_config. before scripts/check.sh validation and launch.

## When NOT to use

- If a tool/service requiring local model deployment (FunTTS_mcp_servers, music_mcp_servers, Audioseparator_mcp_servers) has not been locally deployed — its registration must be removed instead of configured
- If you only want a quick API-based experience — configure MD_TOKEN and HF_TOKEN in servers/API_servers.py rather than deploying per-service local registrations

## Inputs

- mcp_servers/servers_config. configuration file
- Per-service Conda environments created from environment.yml or environment-lock.yml
- Python interpreter paths for each isolated service environment
- Service scripts under mcp_servers/servers/ (e.g. markdown_servers.py)
- .env file with LLM_API_KEY, LLM_BASE_URL, LLM_MODEL_NAME, folder paths

## Outputs

- A valid mcp_servers/servers_config. registering each MCP service
- scripts/check.sh reporting the configuration is correct
- All 9 registered MCP services started and executing successfully via scripts/start_all.py

## How to apply

Edit mcp_servers/servers_config. and, for each service (e.g. markdown_servers), set 'command' to the local Python interpreter path of its dedicated Conda environment, list the service script (e.g. mcp_servers/servers/markdown_servers.py) under 'args', and set the 'env' PYTHONPATH to '.'. This per-service interpreter indirection is what isolates dependencies, eliminating conflicts that would otherwise arise from AudioFab's mixed PyTorch, audio-processing-library, and web-framework stack. If local deployment of FunTTS_mcp_servers, music_mcp_servers, or Audioseparator_mcp_servers was not completed, remove those registrations from servers_config. so AudioFab skips locally deployed models. Validate the configuration with scripts/check.sh, then launch all registered MCP services (client plus server managing the framework's 9 services) with python scripts/start_all.py after conda activate AudioFab.

## Related tools

- **AudioFab** (Framework whose MCP services are registered and launched; provides scripts/check.sh and scripts/start_all.py) — https://github.com/SmileHnu/AudioFab
- **Conda** (Creates the isolated per-service virtual environments whose interpreter paths are referenced as the 'command' for each registered service)
- **Python** (Interpreter invoked per service via the 'command' field; runs the service scripts listed in args)

## Examples

```
conda activate AudioFab; python scripts/start_all.py
```

## Evaluation signals

- scripts/check.sh reports the configuration is correct
- All registered MCP services (client plus server managing the framework's 9 services) start and execute successfully via python scripts/start_all.py
- Each service entry in servers_config. has a command (interpreter path), args (script path), and env with PYTHONPATH set to '.'
- Services not locally deployed (FunTTS_mcp_servers, music_mcp_servers, Audioseparator_mcp_servers) are absent from the registration when their models are missing

## Limitations

- Running models requiring local deployment consumes substantial computing resources; API use is recommended for a quicker experience
- If registrations for locally deployed model services are left in place without deployment, AudioFab will fail to use those models
- Dependencies include PyTorch, audio processing libraries, and web frameworks, so a single shared environment risks dependency conflicts — isolation is mandatory

## Evidence

- [readme] Per-service interpreter and script registration: [evidence span withheld — anchor status: not_found]
