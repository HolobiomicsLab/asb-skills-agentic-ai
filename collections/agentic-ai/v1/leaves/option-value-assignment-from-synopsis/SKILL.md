---
name: option-value-assignment-from-synopsis
description: Use when you have parsed program documentation (name, description, synopsis,
  option descriptions) plus predicted high-risk option combinations and must convert
  each combination into a single executable fuzzing command with semantically consistent
  option values and input files.
when_to_use_negative:
- The task is predicting which option combinations are high-risk or extracting inter-option
  constraints — those are upstream stages (constraint.py, predict.py), not command
  assembly.
- No parsed documentation (synopsis/option descriptions) is available; assembly depends
  on the LLM comprehending the program synopsis, not raw binaries or source code.
- Commands may legitimately contain chained sub-commands ('&&') — the skill enforces
  exclusive single commands by design.
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: ProphetFuzz
  role: Tool whose command-assembly component (llm_interface/assembly.py, generateCommands)
    performs option value allocation and command emission
  repo: https://github.com/NASP-THU/ProphetFuzz
- name: GPT-4 Turbo
  role: LLM prompted (temperature 0.7, n = 3 × N inferences) to comprehend combination
    semantics and allocate option values and files
- name: Python
  role: Implementation language of ProphetFuzz (~3k lines) and the generated file-generation
    scripts executed in a sandbox
  repo: https://github.com/NASP-THU/ProphetFuzz
- name: AFL
  role: Fuzzer whose seed-file guidance (initial test cases under 1 kB) motivates
    the file0 < 1KB constraint in assembly
  repo: https://github.com/google/AFL
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1145/3658644.3690231
    title: 'ProphetFuzz: Fully Automated Prediction and Fuzzing of High-Risk Option
      Combinations with Only Documentation via Large Language Model'
  source_content_hash: sha256:e8225c9be896157f2864a9514bb1446ccdb1aa1c739926fb60e80e5e7dde5f6d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NASP-THU/ProphetFuzz
  tools:
  - AFL
  - GPT-4 Turbo
  - OpenAI API
  - ProphetFuzz
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Option value assignment from synopsis via LLM command assembly

## Summary

ProphetFuzz's command-assembly stage: prompt GPT-4 Turbo to comprehend the collective semantics of each predicted high-risk option combination, allocate concrete option values and file0/fileN placeholders, and emit one exclusive executable command per combination from parsed Groff man-page synopsis alone.

## When to use

Use when you have parsed program documentation (name, description, synopsis, option descriptions) plus predicted high-risk option combinations and must convert each combination into a single executable fuzzing command with semantically consistent option values and input files.

## When NOT to use

- The task is predicting which option combinations are high-risk or extracting inter-option constraints — those are upstream stages (constraint.py, predict.py), not command assembly.
- No parsed documentation (synopsis/option descriptions) is available; assembly depends on the LLM comprehending the program synopsis, not raw binaries or source code.
- Commands may legitimately contain chained sub-commands ('&&') — the skill enforces exclusive single commands by design.

## Inputs

- Parsed program documentation (program name, description, usage synopsis, option descriptions from Groff man pages)
- Predicted high-risk option combinations from the High-Risk Combination Prediction stage (predictCombinations)
- OpenAI API key configured in llm_interface/config/.env

## Outputs

- Assembled executable fuzzing commands per program (assembled_commands), one exclusive command per predicted combination with file0/fileN placeholders replaced
- File-generation Python scripts whose execution in a sandbox yields the required input and auxiliary files

## How to apply

Load the parsed Groff man-page content and the predicted high-risk option combinations from the upstream prediction stage (predictCombinations), then call the assembly module (llm_interface/assembly.py, generateCommands). Prompt GPT-4 Turbo (gpt-4-1106-preview) via the OpenAI API with temperature 0.7 — the setting ProphetFuzz uses for diverse-outcome tasks like command assembly — to comprehend the collective semantics of each option combination and allocate option values, running n = 3 × N inferences (N = number of options needing assignment) for diversity. Require one exclusive single command per combination using file0/fileN placeholders for input and auxiliary files, with no '&&' chaining, all placeholders resolved to concrete file references, required files saved under exact given names, and file0 kept under 1KB (consistent with AFL seed-size guidance). Filter and deduplicate the LLM outputs, keeping only executable commands satisfying all constraints, and save the final set per program as assembled_commands. Semantic consistency between values and files matters because ProphetFuzz's motivation (L2) is that command components — options, values, files — are interconnected and mismatched pairs break fuzzing.

## Related tools

- **ProphetFuzz** (Tool whose command-assembly component (llm_interface/assembly.py, generateCommands) performs option value allocation and command emission) — https://github.com/NASP-THU/ProphetFuzz
- **GPT-4 Turbo** (LLM prompted (temperature 0.7, n = 3 × N inferences) to comprehend combination semantics and allocate option values and files)
- **Python** (Implementation language of ProphetFuzz (~3k lines) and the generated file-generation scripts executed in a sandbox) — https://github.com/NASP-THU/ProphetFuzz
- **AFL** (Fuzzer whose seed-file guidance (initial test cases under 1 kB) motivates the file0 < 1KB constraint in assembly) — https://github.com/google/AFL

## Examples

```
bash run_all_in_one.sh bison
```

## Evaluation signals

- Output file assembled_commands exists and is valid JSON.
- No command contains '&&' command chaining and no command contains unresolved file0/fileN placeholders.
- All commands are unique (deduplicated) and executable in form, consistent with the paper's 7614 unique executable commands assembled for the 52-program dataset.
- Each predicted combination yields exactly one exclusive command; required files are saved under the exact names the LLM specified and file0 stays under 1KB.
- Generated file-generation scripts execute successfully in the sandbox and produce the declared files (C3).

## Limitations

- LLM hallucination threatens precision: the LLM may allocate semantically inconsistent values or violate exclusivity/constraint rules, so filtering and deduplication of generated commands is required.
- Cost and diversity trade-offs: temperature 0.7 and n = 3 × N inferences increase token cost (ProphetFuzz averaged $8.69 per program overall); fewer inferences reduce output diversity.
- Assembly quality is bounded by documentation parsing: only Groff man-page-derived name, description, synopsis, and option descriptions are available, so semantic mismatches can occur for poorly documented options.

## Evidence

- [intro] Prophet Fuzz initially guides the LLM to comprehend the collective semantics of the option combination and leverage this understanding to allocate option values and files: [evidence span withheld — anchor status: not_found]
- [readme] AFL seed size guidance underpinning the file0 < 1KB constraint: [evidence span withheld — anchor status: external_doc]
