---
name: command-line-input-placeholder-identification
description: Use when converting LLM-assembled or manually written command lines for
  CLI programs into fuzzer-ready commands and you must decide which file argument
  is the primary input to mutate.
when_to_use_negative:
- The target program reads its primary input from stdin rather than a file argument
  — use afl-fuzz's stdin invocation form ('afl-fuzz -i testcase_dir -o findings_dir
  /path/to/program') instead of rewriting a placeholder as @@.
- The command contains no file placeholders at all (options and values only), so there
  is no mutable input target for afl-fuzz-style file fuzzing.
- You are doing option/value combination fuzzing where the file argument is a fixed
  auxiliary file (e.g., a config named via a non-input keyword) and the fuzzer should
  mutate option values instead.
tools:
- name: ProphetFuzz
  role: Source of the assembled commands and the input-file identification rules (default
    file0, input/test/source keywords, single-remaining-placeholder fallback) implemented
    in the fuzzing_handler pipeline
  repo: github.com/NASP-THU/ProphetFuzz
- name: AFL
  role: 'Defines the ''@@'' command-line placeholder convention: afl-fuzz replaces
    the ''@@'' position with the path of the mutated input file during fuzzing'
  repo: google/AFL
- name: Carpet Fuzz
  role: Upstream related work whose datasets contribute the target programs whose
    assembled commands are processed by this skill
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1145/3658644.3690231
    title: 'ProphetFuzz: Fully Automated Prediction and Fuzzing of High-Risk Option
      Combinations with Only Documentation via Large Language Model'
  source_content_hash: sha256:c182cd42a91dda579f512920c9e6c02c7b40dbdca42c34412ab3af32807466fa
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - AFL
  - Carpet Fuzz
  - ProphetFuzz
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# command-line-input-placeholder-identification

## Summary

Identifies which file placeholder in an assembled, executable command-line corresponds to the primary (fuzzer-mutable) input file and rewrites it as @@ so afl-fuzz-style fuzzers substitute mutated inputs at that position, following ProphetFuzz's input-file identification rules.

## When to use

Use when converting LLM-assembled or manually written command lines for CLI programs into fuzzer-ready commands and you must decide which file argument is the primary input to mutate.

## When NOT to use

- The target program reads its primary input from stdin rather than a file argument — use afl-fuzz's stdin invocation form ('afl-fuzz -i testcase_dir -o findings_dir /path/to/program') instead of rewriting a placeholder as @@.
- The command contains no file placeholders at all (options and values only), so there is no mutable input target for afl-fuzz-style file fuzzing.
- You are doing option/value combination fuzzing where the file argument is a fixed auxiliary file (e.g., a config named via a non-input keyword) and the fuzzer should mutate option values instead.

## Inputs

- Filtered executable commands with file placeholders (e.g., 'file0', 'file1', 'input.mp3') produced by ProphetFuzz's command assembly step (llm_interface/assembly.py generateCommands)
- Corresponding generated seed input files obtained from the sandboxed file-generation step (fuzzing_handler/generate_combination.py)

## Outputs

- fuzz_ready_commands.txt containing one command per line with exactly one '@@' placeholder
- argvs files for the fuzzer (e.g., fuzzing_handler/argvs/argvs_bison.txt) in the form '-- path/to/bison/bin/bison @@'
- Excluded-command list for commands where no input placeholder could be identified

## How to apply

Load the filtered executable commands produced by the command assembly step for each target program (ProphetFuzz's evaluation used 52 programs from 40 packages). For each command, apply the identification rule in order: (1) default to a placeholder named 'file0' when present, since most generated commands use this convention; (2) otherwise scan the remaining file placeholders for keywords LLMs commonly use to denote input files, such as 'input', 'test', or 'source'; (3) otherwise, if exactly one file placeholder remains, designate it as the input file. Rewrite the identified placeholder as '@@', the marker AFL-family fuzzers substitute at runtime with the mutated input file path, and exclude any command for which all three criteria fail because it has no mutable input target. Write retained commands to fuzz_ready_commands.txt in a format suitable for passing to the fuzzer, e.g. via an argvs file such as fuzzing_handler/argvs/argvs_bison.txt.

## Related tools

- **ProphetFuzz** (Source of the assembled commands and the input-file identification rules (default file0, input/test/source keywords, single-remaining-placeholder fallback) implemented in the fuzzing_handler pipeline) — github.com/NASP-THU/ProphetFuzz
- **AFL** (Defines the '@@' command-line placeholder convention: afl-fuzz replaces the '@@' position with the path of the mutated input file during fuzzing) — google/AFL
- **Carpet Fuzz** (Upstream related work whose datasets contribute the target programs whose assembled commands are processed by this skill)

## Examples

```
afl-fuzz -i testcase_dir -o findings_dir -- path/to/bison/bin/bison @@
```

## Evaluation signals

- Every retained command in fuzz_ready_commands.txt contains exactly one '@@' placeholder — commands with zero or multiple '@@' indicate a mis-identification or a non-file-argument command.
- Count of excluded commands matches the number of commands for which all three identification criteria (file0, input/test/source keywords, single remaining placeholder) failed.
- Documented invocation pattern holds, e.g. '-- path/to/bison/bin/bison @@', consistent with the argvs file format under fuzzing_handler/argvs/.
- Fuzzing with the resulting command (fuzzing_handler/run_fuzzing.sh runFuzzing) actually substitutes mutated input files at the '@@' position, i.e., afl-fuzz launches without 'no input file' errors.
- For known cases, the chosen placeholder matches the POWER-dataset convention, e.g. vim's initial command '-u NONE -X -Z -e -s -S @@' places the input file in the final argument position.

## Limitations

- Commands whose file placeholders lack the 'file0' default and any 'input'/'test'/'source' keyword and have more than one remaining placeholder cannot be resolved and are excluded entirely, losing potential fuzz targets.
- Keyword-based identification ('input', 'test', 'source') depends on LLM naming conventions during command assembly; unusual placeholder names can cause the wrong file argument to be marked as the fuzzer input.
- The '@@' mechanism only works for programs that accept input from a file argument; stdin-reading targets require a different afl-fuzz invocation and are not covered.

## Evidence

- [readme] AFL README: @@ marker convention: [evidence span withheld — anchor status: external_doc]
- [other] ProphetFuzz identification rule: [evidence span withheld — anchor status: not_found]
- [other] ProphetFuzz rewriting step: [evidence span withheld — anchor status: not_found]
- [methods] POWER dataset vim example with @@: "Analysis reveals that in the POWER dataset, the initial command for vim is “-u NONE -X -Z -e -s -S @@"
