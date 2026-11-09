---
name: prefix-matching-file-correction
description: Use when an automated pipeline assembles command lines with file placeholders
  (e.g. '@@' positions) and LLM-generated files may arrive with names that deviate
  from the placeholders, requiring resolution before fuzzing or execution.
when_to_use_negative:
- When all generated files already exactly match their command placeholders, so no
  name correction is needed
- When commands do not reference file placeholders at all (e.g. stdin-only targets),
  since there is nothing to resolve
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_3377
tools:
- name: ProphetFuzz
  role: Tool implementing the file-resolution filter and prefix-matching correction
    (fuzzing_handler/cmd_fixer.py) as Section 3.5 of the pipeline
  repo: https://github.com/NASP-THU/ProphetFuzz
- name: Python
  role: Language of the ~3k-line prototype and the LLM-generated file-generation scripts
    executed in the sandbox
- name: Docker
  role: Containerized sandbox environment (Ubuntu 20.04 based, built from the Dockerfile)
    used to run the file-generation scripts
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1145/3658644.3690231
    title: 'ProphetFuzz: Fully Automated Prediction and Fuzzing of High-Risk Option
      Combinations with Only Documentation via Large Language Model'
  source_content_hash: sha256:ee9cd11f2d6668abfcb00d50124e0b6b0b89a1fc82f13a552a4f9b9cf8876f04
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
  - http://edamontology.org/topic_3377
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Prefix-matching file correction

## Summary

A file-resolution filter from ProphetFuzz (Section 3.5) that compares files produced by LLM-generated Python scripts in a Docker sandbox against the file placeholders in assembled commands, remapping unexpectedly named files via prefix matching and excluding commands whose crucial input files cannot be resolved.

## When to use

Use when an automated pipeline assembles command lines with file placeholders (e.g. '@@' positions) and LLM-generated files may arrive with names that deviate from the placeholders, requiring resolution before fuzzing or execution.

## When NOT to use

- When all generated files already exactly match their command placeholders, so no name correction is needed
- When commands do not reference file placeholders at all (e.g. stdin-only targets), since there is nothing to resolve

## Inputs

- LLM-generated Python file-generation scripts (from Command Assembly output, e.g. llm_interface/assembly.py generateCommands)
- Assembled commands containing file placeholders referencing expected input files
- Files produced by running the generation scripts in the Docker/virtualenv sandbox

## Outputs

- resolved_commands: set of executable commands with every file placeholder bound to an existing generated file via a verified file_exists mapping
- Excluded command set: commands referencing missing crucial files or uncorrectable file names

## How to apply

First execute each LLM-generated Python file-generation script inside a Docker-based Python sandbox built with virtualenv, which pre-installs 33 commonly used third-party libraries and 36 command-line tools for subprocess calls. Collect all files produced by the sandboxed scripts and compare them against the file placeholders referenced in the assembled commands. Exclude any command whose crucial input files were not successfully generated, since a missing crucial file makes the command non-executable. For produced files whose names deviate from the expected placeholders, apply a prefix-matching heuristic to remap them to the placeholder that expects them (e.g. via the cmd_fixer logic in fuzzing_handler), rather than discarding the command; exclude commands that still cannot be corrected. Save the remaining executable commands with their resolved file mappings to resolved_commands.

## Related tools

- **ProphetFuzz** (Tool implementing the file-resolution filter and prefix-matching correction (fuzzing_handler/cmd_fixer.py) as Section 3.5 of the pipeline) — https://github.com/NASP-THU/ProphetFuzz
- **Python** (Language of the ~3k-line prototype and the LLM-generated file-generation scripts executed in the sandbox)
- **Docker** (Containerized sandbox environment (Ubuntu 20.04 based, built from the Dockerfile) used to run the file-generation scripts)

## Examples

```
docker run -it --privileged=true prophetfuzz bash  # then: python fuzzing_handler/generate_combination.py  (File Generation, Section 3.5)
```

## Evaluation signals

- Every command retained in resolved_commands has all of its file placeholders bound to an existing generated file (verified with a file_exists check per mapping)
- No command referencing a missing crucial file appears in the output
- Every remapped file assignment is justified by a prefix match between the produced file name and the placeholder's expected name
- Commands excluded during resolution correspond to generation scripts that failed or produced fewer files than placeholders

## Limitations

- Commands with genuinely missing crucial files cannot be recovered and are excluded, reducing the fuzzing command set
- Prefix matching is a heuristic: produced files whose names do not share a prefix with any placeholder cannot be remapped and their commands are dropped

## Evidence

- [intro] L2- Semantic Mismatch: Every component of an executable command, such as options, values, and files, is interconnected: [evidence span withheld — anchor status: not_found]
