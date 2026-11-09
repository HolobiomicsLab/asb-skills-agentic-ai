---
name: executable-command-validation-for-fuzzing
description: Use when a pipeline has predicted high-risk option combinations (e.g.,
  ProphetFuzz's 1748 combos) and needs them converted into unique, executable fuzzing
  commands with semantically consistent option values and input files.
when_to_use_negative:
- Option combinations have not passed constraint extraction and self-check, so predicted
  combos may violate option conflicts or dependencies (LLM hallucination risk, challenge
  C1)
- The target program consumes stdin directly with no input-file arguments, so file0/fileN
  placeholder replacement and file generation do not apply
- You need to fuzz a single fixed configuration rather than explore predicted high-risk
  option combinations from documentation
edam_operation: http://edamontology.org/operation_2409
tools:
- name: ProphetFuzz
  role: Implements command assembly (llm_interface/assembly.py, generateCommands)
    and file generation/validation (fuzzing_handler) over parsed man pages and predicted
    combinations
  repo: https://github.com/NASP-THU/ProphetFuzz
- name: GPT-4 Turbo
  role: LLM prompted at temperature 0.7 to comprehend combination semantics, allocate
    option values, and emit exclusive single commands with file placeholders
- name: Python
  role: ~3k-line prototype language; generates and sandbox-executes the file-creation
    scripts and validates/deduplicates assembled commands
  repo: https://github.com/NASP-THU/ProphetFuzz
- name: AFL++
  role: Fuzzer that receives the validated commands and generated files to fuzz the
    high-risk option combinations
  repo: https://github.com/google/AFL
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1145/3658644.3690231
    title: 'ProphetFuzz: Fully Automated Prediction and Fuzzing of High-Risk Option
      Combinations with Only Documentation via Large Language Model'
  source_content_hash: sha256:93ba6496a4abb296d9a229045cefffe7f2032ac107d7fcd1ad28e819744b54f0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NASP-THU/ProphetFuzz
  tools:
  - AFL++
  - GPT-4 Turbo
  - OpenAI API
  - ProphetFuzz
  - Python
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# executable-command-validation-for-fuzzing

## Summary

Assembles and validates LLM-generated executable fuzzing commands from predicted high-risk option combinations, enforcing exclusivity (no '&&' chaining), resolved file0/fileN placeholders, and file-size limits before handoff to a fuzzer.

## When to use

Use when a pipeline has predicted high-risk option combinations (e.g., ProphetFuzz's 1748 combos) and needs them converted into unique, executable fuzzing commands with semantically consistent option values and input files.

## When NOT to use

- Option combinations have not passed constraint extraction and self-check, so predicted combos may violate option conflicts or dependencies (LLM hallucination risk, challenge C1)
- The target program consumes stdin directly with no input-file arguments, so file0/fileN placeholder replacement and file generation do not apply
- You need to fuzz a single fixed configuration rather than explore predicted high-risk option combinations from documentation

## Inputs

- Parsed program documentation (program name, description, usage synopsis, option descriptions extracted from Groff man pages)
- Predicted high-risk option combinations from the upstream LLM prediction stage
- OpenAI API access to GPT-4 Turbo (gpt-4-1106-preview) with configured API key
- Target program binaries for which fuzzing commands will be constructed

## Outputs

- Set of unique, executable fuzzing commands per program (assembled_commands file, valid JSON), each a single exclusive command with no '&&' chaining
- Concrete file references replacing all file0/fileN placeholders, with required files saved under exact given names and file0 under 1KB
- Python file-generation scripts executed in a sandbox to produce the required input/auxiliary files
- Deduplicated, executable-only command set ready to feed into the fuzzer (e.g., 7614 unique executable commands across 52 programs)

## How to apply

Load the parsed program documentation (Groff man pages parsed for name, description, synopsis, option descriptions) and the predicted high-risk option combinations. Prompt GPT-4 Turbo via the OpenAI API (temperature 0.7, the value the paper uses for diverse-outcome tasks like constraint extraction and command assembly) to comprehend the collective semantics of each option combination and allocate option values and files with consistent semantics. Instruct the LLM to emit one exclusive single command per combination using file0/fileN placeholders, with no '&&' chaining, all placeholders replaced with concrete file references, and file0 kept under 1KB (n = 3 × N inferences per combination, where N is the number of options needing assignment, to diversify outputs). Generate and execute Python file-generation scripts within a sandbox environment to obtain the required files. Finally, filter and deduplicate the generated commands, keeping only executable ones that satisfy the constraints, and save them per program (the paper reports 7614 unique executable commands assembled for the 52-program dataset).

## Related tools

- **ProphetFuzz** (Implements command assembly (llm_interface/assembly.py, generateCommands) and file generation/validation (fuzzing_handler) over parsed man pages and predicted combinations) — https://github.com/NASP-THU/ProphetFuzz
- **GPT-4 Turbo** (LLM prompted at temperature 0.7 to comprehend combination semantics, allocate option values, and emit exclusive single commands with file placeholders)
- **Python** (~3k-line prototype language; generates and sandbox-executes the file-creation scripts and validates/deduplicates assembled commands) — https://github.com/NASP-THU/ProphetFuzz
- **AFL++** (Fuzzer that receives the validated commands and generated files to fuzz the high-risk option combinations) — https://github.com/google/AFL

## Examples

```
bash run_all_in_one.sh bison
```

## Evaluation signals

- Output file exists and is valid JSON with one command per predicted combination
- No command contains '&&' chaining or unresolved file0/fileN placeholders
- All commands are unique (paper benchmark: 7614 unique executable commands for the 52-program, 40-package dataset)
- All required files exist under the exact names given in the commands, with file0 kept under 1KB
- Commands are executable in form and were deduplicated after the n = 3 × N diverse-outcome inferences

## Limitations

- LLM hallucination threatens constraint-extraction precision (C1): combinations that violate option conflicts/dependencies can surface as high-risk and produce invalid commands if constraints are not injected first
- Temperature 0.7 maximizes diverse outcomes but requires downstream filtering and deduplication, since redundant or non-executable commands are generated
- Semantic mismatch among options, values, and files (L2) is only mitigated, not eliminated — option values and files must be checked for consistent collective semantics before fuzzing

## Evidence

- [intro] Prophet Fuzz initially guides the LLM to comprehend the collective semantics of the option combination and leverage this understanding to allocate option values and files: [evidence span withheld — anchor status: not_found]
- [readme] Keep the files small. Under 1 kB is ideal, although not strictly necessary.: [evidence span withheld — anchor status: external_doc]
