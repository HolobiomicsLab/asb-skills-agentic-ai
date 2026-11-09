---
name: command-semantic-alignment-assembly
description: Use when predicted option combinations must be turned into concrete,
  exclusive, executable command lines with option values and file0/fileN placeholders
  before fuzzing can start.
when_to_use_negative:
- Input combinations have not been filtered for constraint violations (conflicts/dependencies)
  by the upstream constraint-extraction and self-check stages — the LLM tends to identify
  constraint-violating combinations as high-risk (C1)
- The goal is fuzzing setup for binaries where the LLM should not be involved, e.g.
  plain AFL-style seed and '@@' placement without option-combination semantics
edam_operation: http://edamontology.org/operation_3096
tools:
- name: ProphetFuzz
  role: Tool whose Command Assembly component (llm_interface/assembly.py, function
    generateCommands) implements the semantic-alignment command assembly
  repo: https://github.com/NASP-THU/ProphetFuzz
- name: GPT-4 Turbo
  role: LLM prompted at temperature 0.7 to comprehend collective option-combination
    semantics and allocate values and files
- name: Python
  role: Implementation language of the ~3k-line ProphetFuzz prototype, including assembly
    and file-generation scripts
  repo: https://github.com/NASP-THU/ProphetFuzz
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1145/3658644.3690231
    title: 'ProphetFuzz: Fully Automated Prediction and Fuzzing of High-Risk Option
      Combinations with Only Documentation via Large Language Model'
  source_content_hash: sha256:9f4ba20664e77103b3dda42bd00ec7fde07f340256873d1d0f44eefb0aedfc55
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NASP-THU/ProphetFuzz
  tools:
  - GPT-4 Turbo
  - OpenAI API
  - ProphetFuzz
  - Python
  edam_operation: http://edamontology.org/operation_3096
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# command-semantic-alignment-assembly

## Summary

Converts predicted high-risk option combinations into executable fuzzing commands by prompting GPT-4 Turbo to comprehend the collective semantics of each option combination and allocate semantically consistent option values and file placeholders, producing 7614 unique executable commands across the 52-program dataset.

## When to use

Use when predicted option combinations must be turned into concrete, exclusive, executable command lines with option values and file0/fileN placeholders before fuzzing can start.

## When NOT to use

- Input combinations have not been filtered for constraint violations (conflicts/dependencies) by the upstream constraint-extraction and self-check stages — the LLM tends to identify constraint-violating combinations as high-risk (C1)
- The goal is fuzzing setup for binaries where the LLM should not be involved, e.g. plain AFL-style seed and '@@' placement without option-combination semantics

## Inputs

- Parsed program documentation (program name, description, usage synopsis, option descriptions from Groff man pages)
- Predicted high-risk option combinations from the upstream prediction task
- LLM prompt template enforcing exclusivity, placeholder replacement, and file-size constraints

## Outputs

- One exclusive executable command per option combination with concrete option values and resolved file references (no '&&' chaining, no unresolved placeholders)
- assembled_commands file per program (7614 unique executable commands across the 52-program dataset)
- File requirements (file0 under 1KB, auxiliary files under exact given names) passed to the File Generation stage

## How to apply

Load the parsed program synopsis and the predicted high-risk combinations from the upstream prediction stage, then prompt GPT-4 Turbo (gpt-4-1106-preview) via the OpenAI API with temperature 0.7 — the setting ProphetFuzz reserves for tasks needing diverse outcomes like command assembly. Instruct the LLM to allocate option values using n = 3 × N inferences (N = number of options needing assignment) to ensure diverse, non-redundant outputs, and to emit one exclusive single command per combination, using file0/fileN placeholders for input and auxiliary files. Enforce prompt constraints during assembly: no '&&' chaining of multiple commands, all placeholders replaced with concrete file references, required files saved under their exact given names, and file0 kept under 1KB. Filter and deduplicate the LLM-generated commands, keeping only executable ones that satisfy the constraints, then save the final set per program as assembled_commands. The rationale is ProphetFuzz's limitation L2 — components of an executable command (options, values, files) are semantically interconnected, so value/file allocation must follow the combination's collective semantics rather than random assignment.

## Related tools

- **ProphetFuzz** (Tool whose Command Assembly component (llm_interface/assembly.py, function generateCommands) implements the semantic-alignment command assembly) — https://github.com/NASP-THU/ProphetFuzz
- **GPT-4 Turbo** (LLM prompted at temperature 0.7 to comprehend collective option-combination semantics and allocate values and files)
- **Python** (Implementation language of the ~3k-line ProphetFuzz prototype, including assembly and file-generation scripts) — https://github.com/NASP-THU/ProphetFuzz

## Examples

```
bash run_all_in_one.sh bison
```

## Evaluation signals

- No assembled command contains '&&' chaining of multiple commands (exclusivity constraint)
- No command contains unresolved file placeholders — every file0/fileN is replaced with a concrete file reference
- All commands are unique after deduplication and executable in form, consistent with 7614 unique executable commands for 52 programs
- Required files are saved under the exact names given in the command and file0 is kept under 1KB
- The output file assembled_commands exists and is valid JSON

## Limitations

- LLM hallucination threatens precision — LLMs tend to mark constraint-violating combinations as high-risk, so constraint information must be provided upstream before assembly
- LLM outputs are non-deterministic; temperature 0.7 is used deliberately for diverse outcomes, requiring n = 3 × N value-assignment inferences and deduplication filtering
- Generated Python file-generation scripts must be executed in a sandbox (C3) because their content is LLM-produced and untrusted

## Evidence

- [intro] Prophet Fuzz initially guides the LLM to comprehend the collective semantics of the option combination and leverage this understanding to allocate option values and files: [evidence span withheld — anchor status: not_found]
- [methods] 0.7 for tasks needing diverse outcomes, like constraint extraction and command assembly: "0.7 for tasks needing diverse outcomes, like constraint extraction and command assembly"
- [readme] | 3.4 | Command Assembly | [llm_interface/assembly.py](llm_interface/assembly.py) | generateCommands |: "| 3.4 | Command Assembly | [llm_interface/assembly.py](llm_interface/assembly.py) | generateCommands |"
- [intro] executes these scripts within a sandbox environment to obtain the required files (C3): "executes these scripts within a sandbox environment to obtain the required files (C3)"
