---
name: command-placeholder-resolution
description: Use when an LLM has assembled command lines containing named file placeholders
  and you must verify each command is executable before fuzzing by checking that sandbox-generated
  files exist for every placeholder.
when_to_use_negative:
- Commands contain no file placeholders (pure option/value combinations) — nothing
  to resolve and the filter would not apply
- You lack a sandbox or the generated scripts make network/system calls beyond the
  sandbox's 33 pre-installed libraries and 36 command-line tools, so execution results
  would be misleading
- You are validating the LLM's constraint extraction or combination prediction (ProphetFuzz
  Sections 3.2–3.3) rather than command executability
tools:
- name: ProphetFuzz
  role: Implements the file-resolution filter; the cmd_fixer.py and code_checker.py
    modules in fuzzing_handler perform command repair and file/placeholder checking
    before fuzzing
  repo: github.com/NASP-THU/ProphetFuzz
- name: Python
  role: Language of the ~3k-line prototype and of the LLM-generated file-generation
    scripts executed in the sandbox
- name: Docker
  role: Provides the sandboxed execution environment (Dockerfile tested on Ubuntu
    20.04) in which file-generation scripts are run to produce the input files
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1145/3658644.3690231
    title: 'ProphetFuzz: Fully Automated Prediction and Fuzzing of High-Risk Option
      Combinations with Only Documentation via Large Language Model'
  source_content_hash: sha256:2de16c0ebeec9f63b1f6a75b874269a9d76d2114479d7dfa960ec95df2acff4d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Docker
  - ProphetFuzz
  - Python
  - virtualenv
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# command-placeholder-resolution

## Summary

A file-resolution filter from ProphetFuzz (Section 3.5) that validates assembled option-combination commands by executing LLM-generated Python file-generation scripts in a Docker sandbox and binding the produced files to the commands' file placeholders, dropping or repairing commands whose files cannot be resolved.

## When to use

Use when an LLM has assembled command lines containing named file placeholders and you must verify each command is executable before fuzzing by checking that sandbox-generated files exist for every placeholder.

## When NOT to use

- Commands contain no file placeholders (pure option/value combinations) — nothing to resolve and the filter would not apply
- You lack a sandbox or the generated scripts make network/system calls beyond the sandbox's 33 pre-installed libraries and 36 command-line tools, so execution results would be misleading
- You are validating the LLM's constraint extraction or combination prediction (ProphetFuzz Sections 3.2–3.3) rather than command executability

## Inputs

- Assembled option-combination commands containing file placeholders (e.g. '@@' style input-file markers)
- LLM-generated Python file-generation scripts
- Docker sandbox environment (Python + virtualenv, pre-installed third-party libraries and command-line tools)

## Outputs

- resolved_commands: executable commands whose placeholders are all bound to existing generated files
- File mappings from each command's placeholders to actual sandbox-generated file paths
- Excluded command set (commands referencing missing crucial files or uncorrectable placeholder names)

## How to apply

Run each LLM-generated Python file-generation script inside a Docker-based Python sandbox built with virtualenv that pre-installs 33 commonly used third-party libraries and 36 command-line tools for subprocess calls. Collect the files produced by the sandboxed scripts and compare them against the file placeholders referenced in the assembled commands; exclude any command whose crucial input files were not generated (fewer produced files than placeholders indicates a missing crucial file). For unexpectedly named produced files, apply a prefix-matching heuristic to remap them to the placeholders that expect them instead of discarding the commands, mirroring ProphetFuzz's cmd_fixer.py stage; only remap prefixes that plausibly correspond. Save the surviving commands with their resolved file mappings to resolved_commands, then hand commands and files to the fuzzer.

## Related tools

- **ProphetFuzz** (Implements the file-resolution filter; the cmd_fixer.py and code_checker.py modules in fuzzing_handler perform command repair and file/placeholder checking before fuzzing) — github.com/NASP-THU/ProphetFuzz
- **Python** (Language of the ~3k-line prototype and of the LLM-generated file-generation scripts executed in the sandbox)
- **Docker** (Provides the sandboxed execution environment (Dockerfile tested on Ubuntu 20.04) in which file-generation scripts are run to produce the input files)

## Examples

```
bash run_all_in_one.sh bison
```

## Evaluation signals

- Every command retained in resolved_commands has all of its file placeholders bound to an existing generated file (verified with a file_exists check per mapping)
- No command referencing a missing crucial file appears in the output (commands with fewer produced files than placeholders are excluded)
- Each prefix-mapped file name corresponds to exactly one placeholder; uncorrectable mappings are dropped rather than guessed
- Retained commands execute in the fuzzer without file-not-found failures, confirming the sandbox actually produced the bound files

## Limitations

- Commands are excluded when fewer files than placeholders are produced, so partial file generation forfeits the whole command even if most inputs exist
- The prefix-matching heuristic is heuristic: unexpectedly named files may be remapped incorrectly or, if uncorrectable, cause the command to be dropped
- The sandbox's fixed pre-installed library/tool set (33 libraries, 36 CLI tools) bounds what file-generation scripts can run; scripts needing other dependencies fail
- The full fuzzing pipeline (seed generation, command repair, fuzzing) requires a privileged Docker container ('--privileged=true'), limiting where it can run

## Evidence

- [intro] Sandbox execution rationale: "executes these scripts within a sandbox environment to obtain the required files (C3)"
- [methods] Filter behavior: [evidence span withheld — anchor status: not_found]
- [methods] Prefix remapping: [evidence span withheld — anchor status: not_found]
- [supplementary] Sandbox composition: [evidence span withheld — anchor status: not_found]
- [readme] Command repair module: "fuzzing_handler: Scripts for preparing and conducting fuzzing."
- [readme] Full pipeline entry point: "Execute the script to start the automated fuzzing process:
   
   ```bash
   bash run_all_in_one.sh bison"
