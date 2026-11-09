---
name: mcp-service-coverage-mapping
description: 'Use when reconstructing or verifying a feature comparison table for
  AudioFab or similar MCP-based agent frameworks: you need to trace each advertised
  capability (speech/music/sound editing, understanding, generation, batch text input)
  back to the specific MCP service that provides it, or fill in.'
when_to_use_negative:
- Input is a functional-audio processing task itself (e.g., speech emotion recognition
  or music separation) — this skill is a documentation/comparison exercise, not an
  audio pipeline
- The framework under evaluation does not expose service-level capability descriptions
  (e.g., a monolithic tool without MCP-style services), making per-service capability
  tracing impossible
- Only a single system's capabilities need listing without comparison against prior
  agent frameworks such as AudioGPT, WavCraft, or WavJourney
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3345
tools:
- name: AudioFab
  role: Framework whose MCP Server services (FunTTS, Music, Audio Separator, Markdown
    servers) are mapped to capability cells; its MCP-based module management distinguishes
    it from prior platforms
  repo: https://github.com/SmileHnu/AudioFab
- name: AudioGPT
  role: Prior audio agent framework used as a comparison row (LangChain module management;
    ✗ ✗ ✓ ✓ ✓ pattern)
- name: WavCraft
  role: Prior audio agent framework used as a comparison row (None module management;
    ✗ ✓ ✗ ✓ ✗ pattern)
- name: WavJourney
  role: Prior audio agent framework used as a comparison row (None module management;
    ✗ ✓ ✗ ✓ ✗ pattern)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1145/3746027.3756869
    title: 'AudioFab: Building A General and Intelligent Audio Factory through Tool
      Learning'
  source_content_hash: sha256:4380bc30e6a9e82ccf69feb81c0ff718da80f3fca8156a6eeb7f260a5e517788
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/SmileHnu/AudioFab
  tools:
  - AudioFab
  - AudioGPT
  - WavCraft
  - WavJourney
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3345
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# mcp-service-coverage-mapping

## Summary

Maps the capabilities of AudioFab's nine MCP Server services (Markdown, Tensor, Tool Query, DSP, Audio, FunTTS, Music, Audio Separator servers) onto a structured capability matrix (editing, understanding, generation, batch input) so that a comparison against prior audio agent frameworks like AudioGPT, WavCraft, and WavJourney can be assembled and validated cell-by-cell.

## When to use

Use when reconstructing or verifying a feature comparison table for AudioFab or similar MCP-based agent frameworks: you need to trace each advertised capability (speech/music/sound editing, understanding, generation, batch text input) back to the specific MCP service that provides it, or fill in check/cross cells for AudioFab versus prior platforms.

## When NOT to use

- Input is a functional-audio processing task itself (e.g., speech emotion recognition or music separation) — this skill is a documentation/comparison exercise, not an audio pipeline
- The framework under evaluation does not expose service-level capability descriptions (e.g., a monolithic tool without MCP-style services), making per-service capability tracing impossible
- Only a single system's capabilities need listing without comparison against prior agent frameworks such as AudioGPT, WavCraft, or WavJourney

## Inputs

- Feature comparison table from the paper's introduction (rows: AudioFab, AudioGPT, WavCraft, WavJourney; columns: module management, editing, understanding, generation, batch input)
- Repository README service descriptions for MCP Server services (FunTTS, Music, Audio Separator, Markdown, Tensor, Tool Query, DSP, Audio servers)
- AudioFab repository (https://github.com/SmileHnu/AudioFab) including mcp_servers/servers_config.

## Outputs

- comparison_table.md — a five-column capability matrix with rows for AudioFab, AudioGPT, WavCraft, and WavJourney
- Service-to-capability mapping: each capability cell annotated with the MCP service that provides it

## How to apply

First, extract the check/cross entries from the paper's introduction comparison table, which records AudioGPT with LangChain and per-capability patterns (AudioGPT: ✗ ✗ ✓ ✓ ✓; WavCraft and WavJourney: ✗ ✓ ✗ ✓ ✗) with 'None' as module management for prior systems. Second, anchor AudioFab's module-management cell to the README's independent tool management architecture based on Model-Context-Protocols (MCPs), where each tool runs in its own dedicated dependency environment and is integrated via simple registration. Third, verify the editing, understanding, and generation cells against the three advanced functional services: FunTTS MCP Servers (Speech Editing, Speech/Sound/Music Understanding, Text2Speech, Voice Conversion), Music MCP Servers (Text2Audio, Text2Music, Lyrics2Song, etc.), and Audio Separator MCP Servers (Audio Separation, Music Separation, Music Mix Track). Fourth, verify the batch-input cell from the Markdown Servers service, which handles text formats such as Markdown, TXT, and JSON. Finally, assemble the five-column matrix (module management, editing, understanding, generation, batch input) and cross-check all 20 cells against README-documented capabilities and cited table entries before saving (e.g., as comparison_table.md).

## Related tools

- **AudioFab** (Framework whose MCP Server services (FunTTS, Music, Audio Separator, Markdown servers) are mapped to capability cells; its MCP-based module management distinguishes it from prior platforms) — https://github.com/SmileHnu/AudioFab
- **AudioGPT** (Prior audio agent framework used as a comparison row (LangChain module management; ✗ ✗ ✓ ✓ ✓ pattern))
- **WavCraft** (Prior audio agent framework used as a comparison row (None module management; ✗ ✓ ✗ ✓ ✗ pattern))
- **WavJourney** (Prior audio agent framework used as a comparison row (None module management; ✗ ✓ ✗ ✓ ✗ pattern))

## Examples

```
git clone https://github.com/SmileHnu/AudioFab.git && cd AudioFab && conda env create -f environment.yml && conda activate AudioFab && bash scripts/check.sh
```

## Evaluation signals

- All 20 cells of the five-column matrix (4 systems × 5 capabilities) are filled with values consistent with the paper's recorded patterns (AudioGPT: ✗ ✗ ✓ ✓ ✓; WavCraft/WavJourney: ✗ ✓ ✗ ✓ ✗)
- Every AudioFab capability cell traces to a named MCP service: editing/understanding/generation to FunTTS, Music, and Audio Separator MCP Servers; batch input to Markdown Servers (Markdown, TXT, JSON)
- AudioFab's module-management cell equals 'MCPs', matching the README's independent tool management architecture description
- Prior-system cells match their cited introduction-table entries exactly (e.g., AudioGPT paired with LangChain, WavCraft and WavJourney with None)
- Output file comparison_table.md exists and its header order matches the five-column schema (module management, editing, understanding, generation, batch input)

## Limitations

- AudioFab's paper link was not yet available at extraction time; citation-dependent verification must rely on the repository README and the introduction table text
- No changelog was found in the repository, so service capability coverage may change between the README snapshot and later versions (e.g., the dev1.1 branch updated retrieval model and ranking algorithm)
- FunTTS, Music, and Audio Separator MCP Servers require local model deployment or API tokens (MD_TOKEN, HF_TOKEN); if these services are unregistered, AudioFab will not exercise those capabilities, and the coverage map cannot be validated end-to-end

## Evidence

- [readme] batch input capability: [evidence span withheld — anchor status: not_found]
