---
name: manpage-synopsis-value-extraction
description: Use when the input is Groff/roff man-page documentation for a CLI program
  and you need structured fields (name, description, synopsis, option descriptions)
  to drive LLM-based option-combination prediction or command assembly.
when_to_use_negative:
- The documentation is not in Groff/roff man-page format (e.g. HTML docs or Markdown
  READMEs) — the parser targets Groff documents specifically
- You already have structured option/constraint data (e.g. ProphetFuzz's assets/dataset
  groundtruth_for_20_programs.) and only need downstream prediction or assembly
- The target is a library or GUI application with no command-line synopsis to parse
edam_operation: http://edamontology.org/operation_0306
tools:
- name: ProphetFuzz
  role: Provides the manpage_parser module (parser.py, utils/groff_utils.py) that
    performs keyword-matching extraction of name, description, synopsis, and option
    descriptions from Groff man pages
  repo: github.com/NASP-THU/ProphetFuzz
- name: GPT-4 Turbo
  role: Downstream consumer of the parsed synopsis/option descriptions via the OpenAI
    API for constraint extraction, combination prediction, and command assembly (gpt-4-1106-preview,
    temperature 0.7)
- name: Python
  role: Implementation language of the manpage_parser and llm_interface components
    (~3k lines of Python code)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1145/3658644.3690231
    title: 'ProphetFuzz: Fully Automated Prediction and Fuzzing of High-Risk Option
      Combinations with Only Documentation via Large Language Model'
  source_content_hash: sha256:fa0b291c74267acfba29ca6372535a5a72ccf022db294fc18ca7162b2acd7ddd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GPT-4 Turbo
  - OpenAI API
  - ProphetFuzz
  - Python
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# manpage-synopsis-value-extraction

## Summary

Parses Groff man pages of command-line programs to extract the program name, description, usage synopsis, and per-option descriptions via keyword matching, producing the structured documentation that downstream LLM stages (constraint extraction, high-risk combination prediction, command assembly) consume.

## When to use

Use when the input is Groff/roff man-page documentation for a CLI program and you need structured fields (name, description, synopsis, option descriptions) to drive LLM-based option-combination prediction or command assembly.

## When NOT to use

- The documentation is not in Groff/roff man-page format (e.g. HTML docs or Markdown READMEs) — the parser targets Groff documents specifically
- You already have structured option/constraint data (e.g. ProphetFuzz's assets/dataset groundtruth_for_20_programs.) and only need downstream prediction or assembly
- The target is a library or GUI application with no command-line synopsis to parse

## Inputs

- Groff man-page document for a command-line program (e.g. from the manpage_parser/input directory)
- Target program name (e.g. bison, as used in run_all_in_one.sh)

## Outputs

- Structured parsed documentation containing program name, description, usage synopsis, and option descriptions (manpage_parser/output)
- Documentation text fed to LLM stages: constraint extraction, high-risk combination prediction, and command assembly

## How to apply

Run the manpage_parser component (manpage_parser/parser.py with manpage_parser/utils/groff_utils.py) on the Groff document, which ProphetFuzz treats as the standard documentation format for command-line programs. Use keyword matching to pull out the essential details: the program's name, its description, the usage synopsis, and each option's description. Emit the parsed content as structured documentation (manpage_parser/output) that the llm_interface stages consume: constraint extraction (llm_interface/constraint.py), high-risk combination prediction (llm_interface/predict.py), and command assembly (llm_interface/assembly.py). The synopsis and option descriptions matter because command assembly allocates option values consistent with the collective semantics of a combination and uses file0/fileN placeholders for input and auxiliary files, so malformed or missing synopsis fields will corrupt downstream value assignment. Validate by checking that every option listed in the synopsis appears in the parsed option descriptions and that no Groff markup artifacts remain in the extracted text.

## Related tools

- **ProphetFuzz** (Provides the manpage_parser module (parser.py, utils/groff_utils.py) that performs keyword-matching extraction of name, description, synopsis, and option descriptions from Groff man pages) — github.com/NASP-THU/ProphetFuzz
- **GPT-4 Turbo** (Downstream consumer of the parsed synopsis/option descriptions via the OpenAI API for constraint extraction, combination prediction, and command assembly (gpt-4-1106-preview, temperature 0.7))
- **Python** (Implementation language of the manpage_parser and llm_interface components (~3k lines of Python code))

## Examples

```
bash run_all_in_one.sh bison
```

## Evaluation signals

- Parsed output contains all four expected fields: program name, description, usage synopsis, and option descriptions
- Every option token appearing in the usage synopsis has a corresponding entry in the extracted option descriptions
- Extracted text is free of residual Groff/roff markup artifacts (no font macros or escape sequences leaking into option names)
- Downstream sanity check: the LLM command-assembly stage can bind file0/fileN placeholders to concrete files with consistent semantics, indicating the synopsis supplied enough structure
- For the 52-program / 40-package evaluation dataset, parsing succeeds for each program before constraint extraction runs

## Limitations

- Keyword matching is brittle for unusual man-page layouts; parsing quality directly gates constraint extraction and prediction precision
- LLM hallucination threatens constraint extraction even with correct parsing — ProphetFuzz needs a bidirectional-reasoning self-check to filter incorrect extractions
- Only command-line programs with Groff documentation are in scope; the parser was developed for that format

## Evidence

- [intro] Prophet Fuzz begins by extracting essential details such as the program's name, description, usage synopsis, and option descriptions through keyword matching: [evidence span withheld — anchor status: not_found]
- [intro] Prophet Fuzz initially guides the LLM to comprehend the collective semantics of the option combination and leverage this understanding to allocate option values and files: [evidence span withheld — anchor status: not_found]
