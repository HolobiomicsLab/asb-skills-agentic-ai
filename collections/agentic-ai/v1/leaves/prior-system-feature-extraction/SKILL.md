---
name: prior-system-feature-extraction
description: Use when reproducing a feature comparison matrix (e.g., Table 1) that
  contrasts a new framework's capabilities — module management, editing, understanding,
  generation, batch input — against cited prior agent systems.
when_to_use_negative:
- The goal is to benchmark runtime performance or audio quality of the systems rather
  than reproduce the paper's static feature/capability comparison table.
- The prior systems' capabilities are not recorded in a check/cross pattern in a cited
  table — the extracted AudioGPT/WavCraft/WavJourney cells would then have to be invented
  rather than traced.
tools:
- name: AudioFab
  role: Subject system whose module-management (MCPs) and capability cells (editing,
    understanding, generation, batch input via Markdown/TXT/JSON) are verified against
    the README
  repo: https://github.com/SmileHnu/AudioFab
- name: AudioGPT
  role: Prior system whose LangChain module management and ✗ ✗ ✓ ✓ ✓ capability pattern
    are extracted from the introduction comparison table
- name: WavCraft
  role: Prior system with no module management ('None') and ✗ ✓ ✗ ✓ ✗ capability pattern
    extracted from the comparison table
- name: WavJourney
  role: Prior system with no module management ('None') and ✗ ✓ ✗ ✓ ✗ capability pattern
    extracted from the comparison table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1145/3746027.3756869
    title: 'AudioFab: Building A General and Intelligent Audio Factory through Tool
      Learning'
  source_content_hash: sha256:fd9179ea205432cce212bbe34eeaf06b7e6478b0bd27eb8e56e65050c3432716
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/SmileHnu/AudioFab
  tools:
  - AudioFab
  - AudioGPT
  - WavCraft
  - WavJourney
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# prior-system-feature-extraction

## Summary

Extracts feature/capability entries for prior systems (AudioGPT, WavCraft, WavJourney) from the paper's introduction comparison table and assembles a five-column feature matrix against AudioFab, whose cells are verified against the README's MCP-based architecture documentation.

## When to use

Use when reproducing a feature comparison matrix (e.g., Table 1) that contrasts a new framework's capabilities — module management, editing, understanding, generation, batch input — against cited prior agent systems.

## When NOT to use

- The goal is to benchmark runtime performance or audio quality of the systems rather than reproduce the paper's static feature/capability comparison table.
- The prior systems' capabilities are not recorded in a check/cross pattern in a cited table — the extracted AudioGPT/WavCraft/WavJourney cells would then have to be invented rather than traced.

## Inputs

- Paper's introduction comparison table with rows for AudioGPT, WavCraft, WavJourney and capability columns (module management, editing, understanding, generation, batch input)
- AudioFab repository README (SmileHnu/AudioFab) with MCP Server service descriptions

## Outputs

- comparison_table.md — a five-column feature matrix with four rows (AudioFab, AudioGPT, WavCraft, WavJourney) filled with ✓/✗/mechanism entries

## How to apply

First, read the ✓/✗ pattern per capability column for each prior system directly from the paper's comparison table in the introduction: AudioGPT uses LangChain with pattern ✗ ✗ ✓ ✓ ✓, while WavCraft and WavJourney have no module management ('None') with pattern ✗ ✓ ✗ ✓ ✗. Second, fill AudioFab's module-management cell from the repository README's documented 'Independent Tool Management Architecture based on Model-Context-Protocols (MCPs)', which gives each tool a dedicated dependency environment. Third, verify AudioFab's editing, understanding, and generation cells against the README's FunTTS, Music, and Audio Separator MCP Server service descriptions (Speech Editing, Speech/Sound/Music Understanding, Text2Speech, Music Generation, Audio Separation). Fourth, verify AudioFab's batch-input cell from the Markdown Servers service, which handles Markdown, TXT, and JSON text formats. Finally, assemble the five-column table (module management, editing, understanding, generation, batch input) with rows for AudioFab, AudioGPT, WavCraft, and WavJourney and save it as comparison_table.md.

## Related tools

- **AudioFab** (Subject system whose module-management (MCPs) and capability cells (editing, understanding, generation, batch input via Markdown/TXT/JSON) are verified against the README) — https://github.com/SmileHnu/AudioFab
- **AudioGPT** (Prior system whose LangChain module management and ✗ ✗ ✓ ✓ ✓ capability pattern are extracted from the introduction comparison table)
- **WavCraft** (Prior system with no module management ('None') and ✗ ✓ ✗ ✓ ✗ capability pattern extracted from the comparison table)
- **WavJourney** (Prior system with no module management ('None') and ✗ ✓ ✗ ✓ ✗ capability pattern extracted from the comparison table)

## Examples

```
python -c "open('comparison_table.md','w').write('| System | Module mgmt | Editing | Understanding | Generation | Batch input |\n| AudioFab | MCPs | ✓ | ✓ | ✓ | ✓ |\n| AudioGPT | LangChain | ✗ | ✗ | ✓ | ✓ |\n| WavCraft | None | ✗ | ✓ | ✗ | ✓ |\n| WavJourney | None | ✗ | ✓ | ✗ | ✓ |')"
```

## Evaluation signals

- All 20 cells (4 systems × 5 columns) are filled with values consistent with the paper's recorded patterns: AudioGPT = ✗ ✗ ✓ ✓ ✓; WavCraft and WavJourney = ✗ ✓ ✗ ✓ ✗.
- AudioFab's module-management cell reads 'MCPs', cross-checked against the README's independent tool management architecture description.
- AudioFab's editing/understanding/generation cells are each backed by a named README service (FunTTS, Music, Audio Separator MCP Servers).
- AudioFab's batch-input cell cites the Markdown Servers handling of Markdown, TXT, and JSON formats.
- Output file comparison_table.md exists and the table structure matches the five columns defined in the workflow.

## Limitations

- The README's local-deployment caveat means AudioFab's FunTTS, Music, and Audio Separator capabilities may be unavailable unless models are deployed locally or accessed via API; the table records documented capability, not guaranteed availability.
- AudioFab is released under CC BY-NC 4.0, so reuse of extracted content may be restricted to non-commercial contexts.
- The paper link was not yet available in the official revised version, so only the introduction table pattern (✗/✗ entries) can be traced.

## Evidence
