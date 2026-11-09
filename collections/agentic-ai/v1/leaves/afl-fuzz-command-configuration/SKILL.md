---
name: afl-fuzz-command-configuration
description: Use when preparing executable commands (e.g., from ProphetFuzz's command
  assembly output) for afl-fuzz-style fuzzing and the primary input-file placeholder
  must be located and rewritten as '@@'.
when_to_use_negative:
- The command reads its primary input from stdin rather than a file argument (afl-fuzz
  uses the plain command-line syntax without '@@' in that case).
- All three identification criteria fail (no 'file0' default, no 'input'/'test'/'source'
  keyword, and more than one placeholder remains) — the command has no mutable input
  target for afl-fuzz-style fuzzing.
- The goal is option-combination prediction or constraint extraction rather than preparing
  commands for the fuzzing stage.
tools:
- name: AFL (afl-fuzz)
  role: Fuzzer that consumes commands containing the '@@' placeholder, substituting
    the mutated input file at the marked position at runtime
  repo: google/AFL
- name: ProphetFuzz
  role: Framework whose fuzzing_handler stage rewrites assembled commands with the
    '@@' placeholder and writes argvs files for fuzzing
  repo: github.com/NASP-THU/ProphetFuzz
- name: Python
  role: Implementation language for the command-rewriting and identification logic
    (~3k lines of Python in ProphetFuzz)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1145/3658644.3690231
    title: 'ProphetFuzz: Fully Automated Prediction and Fuzzing of High-Risk Option
      Combinations with Only Documentation via Large Language Model'
  source_content_hash: sha256:899c3aba15cb0b88a47620b9908fa635e9f1b842ce1de84bfcbc126c84a5f36b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/google/AFL
  tools:
  - AFL
  - AFL (afl-fuzz)
  - Carpet Fuzz
  - ProphetFuzz
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# afl-fuzz-command-configuration

## Summary

Identifies which file placeholder in a filtered, executable command corresponds to the primary input file and rewrites it as '@@' so that afl-fuzz substitutes the mutated input at that position at runtime.

## When to use

Use when preparing executable commands (e.g., from ProphetFuzz's command assembly output) for afl-fuzz-style fuzzing and the primary input-file placeholder must be located and rewritten as '@@'.

## When NOT to use

- The command reads its primary input from stdin rather than a file argument (afl-fuzz uses the plain command-line syntax without '@@' in that case).
- All three identification criteria fail (no 'file0' default, no 'input'/'test'/'source' keyword, and more than one placeholder remains) — the command has no mutable input target for afl-fuzz-style fuzzing.
- The goal is option-combination prediction or constraint extraction rather than preparing commands for the fuzzing stage.

## Inputs

- Filtered executable commands produced by the ProphetFuzz command assembly step (llm_interface/assembly.py generateCommands) for the 52 target programs
- Placeholder lists per command (e.g., file0, file1, ...) generated during command assembly

## Outputs

- fuzz_ready_commands.txt listing retained commands with exactly one '@@' placeholder
- argvs files for the fuzzer (e.g., fuzzing_handler/argvs/argvs_bison.txt)

## How to apply

For each filtered executable command from the upstream command assembly step (e.g., for the 52 target programs), apply the identification rule in order: first check for the default 'file0' placeholder; otherwise scan remaining placeholders for keywords LLMs commonly use to denote input files, such as 'input', 'test', or 'source'; otherwise, if exactly one placeholder remains, designate it as the input file. Rewrite the identified placeholder as '@@', the marker AFL's afl-fuzz uses to mark where the input file name belongs on the target's command line. Exclude any command for which all three criteria fail, since it has no mutable input target and cannot be fuzzed in afl-fuzz style. Write retained commands to a fuzzer-ready format (e.g., fuzz_ready_commands.txt or an argvs file such as fuzzing_handler/argvs/argvs_bison.txt) in patterns like '-- path/to/bison/bin/bison @@'.

## Related tools

- **AFL (afl-fuzz)** (Fuzzer that consumes commands containing the '@@' placeholder, substituting the mutated input file at the marked position at runtime) — google/AFL
- **ProphetFuzz** (Framework whose fuzzing_handler stage rewrites assembled commands with the '@@' placeholder and writes argvs files for fuzzing) — github.com/NASP-THU/ProphetFuzz
- **Python** (Implementation language for the command-rewriting and identification logic (~3k lines of Python in ProphetFuzz))

## Examples

```
echo '-- path/to/bison/bin/bison @@' > fuzzing_handler/argvs/argvs_bison.txt
```

## Evaluation signals

- Every retained command in the output file contains exactly one '@@' placeholder.
- The '@@' position matches the identified primary input placeholder ('file0' by default, or a keyword-matched or sole remaining placeholder).
- Commands failing all three identification criteria are excluded from the output rather than emitted with a guessed placeholder.
- Output file (e.g., fuzz_ready_commands.txt or fuzzing_handler/argvs/argvs_bison.txt) exists and follows the documented invocation pattern '-- path/to/bison/bin/bison @@'.
- afl-fuzz accepts the command without complaining about a missing '@@' marker for file-input targets.

## Limitations

- Commands whose input-file placeholder cannot be identified by any of the three criteria (default 'file0', keywords like 'input'/'test'/'source', or a single remaining placeholder) must be dropped entirely.
- The keyword heuristic ('input', 'test', 'source') depends on LLM naming conventions during command assembly and may miss unconventionally named placeholders.
- Commands where the target reads from stdin need the afl-fuzz invocation without '@@', which this skill does not produce.

## Evidence

- [readme] AFL @@ placeholder marker: [evidence span withheld — anchor status: external_doc]
- [intro] ProphetFuzz feeds assembled commands to the fuzzer: "Finally, Prophet Fuzz inputs all generated commands and corresponding files into the fuzzer to carry out fuzz testing on the high-risk option combinations."
- [readme] fuzzing_handler stage role: "fuzzing_handler: Scripts for preparing and conducting fuzzing."
