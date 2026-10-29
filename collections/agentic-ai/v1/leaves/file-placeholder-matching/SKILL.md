---
name: file-placeholder-matching
description: Use when LLM-generated file-generation scripts have produced files whose
  names may not exactly match the file placeholders referenced in assembled command
  lines, and you must decide which commands are executable.
when_to_use_negative:
- Commands contain no file placeholders (e.g. options and values only) — no file resolution
  is needed.
- The produced files exactly match placeholder names and no remapping or exclusion
  logic is required.
- You are fuzzing binaries without input-file arguments (e.g. stdin-driven targets),
  where placeholder resolution does not apply.
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: ProphetFuzz
  role: 'Implements the file-resolution filter: sandbox execution of generation scripts,
    placeholder comparison, prefix-based remapping, and writing resolved_commands'
  repo: https://github.com/NASP-THU/ProphetFuzz
- name: Docker
  role: Provides the isolated sandbox environment (Dockerfile, Ubuntu 20.04) in which
    Python file-generation scripts are executed
- name: Python
  role: Language of the file-generation scripts (~3k lines) and the filter implementation
    (fuzzing_handler/generate_combination.py, main)
- name: virtualenv
  role: Sandbox environment builder with pre-installed third-party libraries for the
    generation scripts
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1145/3658644.3690231
    title: 'ProphetFuzz: Fully Automated Prediction and Fuzzing of High-Risk Option
      Combinations with Only Documentation via Large Language Model'
  source_content_hash: sha256:438996886d91b175de39e0a1922b462d08d1985751089919311426df8025ddfc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NASP-THU/ProphetFuzz
  tools:
  - Docker
  - ProphetFuzz
  - Python
  - virtualenv
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# file-placeholder-matching

## Summary

ProphetFuzz's file-resolution filter (Section 3.5): compare sandbox-generated files against file placeholders in LLM-assembled commands, exclude commands whose crucial input files were not generated, and apply prefix-matching to remap unexpectedly named files to placeholders before fuzzing.

## When to use

Use when LLM-generated file-generation scripts have produced files whose names may not exactly match the file placeholders referenced in assembled command lines, and you must decide which commands are executable.

## When NOT to use

- Commands contain no file placeholders (e.g. options and values only) — no file resolution is needed.
- The produced files exactly match placeholder names and no remapping or exclusion logic is required.
- You are fuzzing binaries without input-file arguments (e.g. stdin-driven targets), where placeholder resolution does not apply.

## Inputs

- LLM-generated Python file-generation scripts (e.g. from fuzzing_handler/generate_combination.py, function main)
- Assembled commands from the Command Assembly step (llm_interface/assembly.py, generateCommands) containing file placeholders such as '@@'
- Set of files produced by executing the scripts in the Docker/virtualenv sandbox

## Outputs

- resolved_commands: executable commands with every file placeholder bound to an existing generated file (verified via file_exists check per mapping)
- Set of excluded commands whose crucial input files were missing or could not be remapped

## How to apply

First execute each LLM-generated Python file-generation script inside a Docker-based Python sandbox (built with virtualenv, pre-installing 33 third-party libraries and 36 command-line tools for subprocess calls) and collect the produced files. Compare these files against the file placeholders referenced in the assembled commands: any command whose crucial input file was not successfully generated is excluded, since missing crucial files make the command non-executable. For produced files whose names differ from the placeholders, apply a prefix-matching heuristic to remap them to the placeholders that expect them, rather than discarding the associated commands; commands that cannot be corrected this way are excluded. Persist the surviving commands, together with their resolved file mappings, to resolved_commands so downstream fuzzing only receives commands with all placeholders bound to existing files.

## Related tools

- **ProphetFuzz** (Implements the file-resolution filter: sandbox execution of generation scripts, placeholder comparison, prefix-based remapping, and writing resolved_commands) — https://github.com/NASP-THU/ProphetFuzz
- **Docker** (Provides the isolated sandbox environment (Dockerfile, Ubuntu 20.04) in which Python file-generation scripts are executed)
- **Python** (Language of the file-generation scripts (~3k lines) and the filter implementation (fuzzing_handler/generate_combination.py, main))
- **virtualenv** (Sandbox environment builder with pre-installed third-party libraries for the generation scripts)

## Examples

```
bash run_all_in_one.sh bison
```

## Evaluation signals

- Every command retained in resolved_commands has all file placeholders bound to an existing generated file (file_exists check per mapping passes).
- No command referencing a missing crucial input file appears in the output — such commands are excluded before fuzzing.
- Prefix-matching remaps only files whose names share a prefix with the expected placeholder; unmatched files are not force-bound to placeholders.
- Commands excluded at this stage are precisely those whose file-generation scripts failed or produced fewer files than placeholders.
- Downstream fuzzing (run_fuzzing.sh) receives no commands with unresolved '@@'-style placeholders or dangling file references.

## Limitations

- Prefix-matching is a heuristic: files whose names share no prefix with the placeholder cannot be remapped, and their commands are excluded even if semantically equivalent files exist.
- Fewer produced files than placeholders is treated as missing crucial files, which can exclude commands even when a subset of inputs suffices.
- The sandbox's fixed pre-installed set (33 third-party libraries, 36 command-line tools) may not cover all generation-script dependencies; Docker full image must be built for seed generation, command repair, and fuzzing, and fuzzing_handler/config. may need manual path adjustment outside Docker.

## Evidence

- [supplementary] Exclusion of commands with missing crucial files: [evidence span withheld — anchor status: not_found]
- [supplementary] Prefix-matching remapping rationale: [evidence span withheld — anchor status: not_found]
