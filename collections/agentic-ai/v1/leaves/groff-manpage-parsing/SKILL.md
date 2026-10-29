---
name: groff-manpage-parsing
description: Use when raw Groff/man-page documentation of CLI programs must be converted
  into structured per-option descriptions (program name via .TH, sections via .SH,
  options via .TP/.PP/.RS/.sp) before LLM processing.
when_to_use_negative:
- Input documentation is not in Groff/man-page format (e.g., HTML-only docs, PDFs,
  or texinfo) — the keyword matching targets Groff macros like .TH, .SH, .TP
- The goal is direct fuzzing of binaries without documentation-based constraint extraction
  — downstream stages (prediction, assembly, fuzzing) would be unnecessary overhead
edam_operation: http://edamontology.org/operation_0306
tools:
- name: ProphetFuzz
  role: Hosts the manpage_parser component (parser.py, utils/groff_utils.py) and restruct_manpage.py
    used to parse Groff documents into structured option descriptions
  repo: https://github.com/NASP-THU/ProphetFuzz
- name: Python
  role: Implementation language of the ~3k-line ProphetFuzz prototype, including the
    parsing scripts
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1145/3658644.3690231
    title: 'ProphetFuzz: Fully Automated Prediction and Fuzzing of High-Risk Option
      Combinations with Only Documentation via Large Language Model'
  source_content_hash: sha256:0e5453c3180e4cf860f3855463763600e75a2fa684a73c41304dd94faa223838
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NASP-THU/ProphetFuzz
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
yes_rate: 0.7777777777777778
n_claims: 9
---

# groff-manpage-parsing

## Summary

Parses Groff-format man pages of command-line programs into structured records (program name, description, synopsis, option descriptions) that downstream LLM modules consume for constraint extraction. It is the first stage of ProphetFuzz's documentation-only pipeline and determines the quality of everything the LLM sees.

## When to use

Use when raw Groff/man-page documentation of CLI programs must be converted into structured per-option descriptions (program name via .TH, sections via .SH, options via .TP/.PP/.RS/.sp) before LLM processing.

## When NOT to use

- Input documentation is not in Groff/man-page format (e.g., HTML-only docs, PDFs, or texinfo) — the keyword matching targets Groff macros like .TH, .SH, .TP
- The goal is direct fuzzing of binaries without documentation-based constraint extraction — downstream stages (prediction, assembly, fuzzing) would be unnecessary overhead

## Inputs

- Groff man-page documents (.TH/.SH/.TP/.PP/.RS/.sp markup) for command-line programs
- Evaluation dataset of 52 open-source programs from 40 packages, collected from three related studies

## Outputs

- Structured per-program option tables: program name, description, usage synopsis, and per-option descriptions
- Restructured man-page records (via restruct_manpage.py) ready for LLM constraint extraction

## How to apply

Run the manpage_parser package (parser.py with groff_utils.py) on the Groff documents of each target program, extracting the program name from .TH directives, section content from .SH directives, and option descriptions from .TP/.PP/.RS/.sp blocks via keyword matching. Restructure the output (restruct_manpage.py in llm_interface) so each option carries its name and description text, which becomes the input for extractRelationships-based constraint extraction with GPT-4 Turbo (temperature 0.7, n=10 inferences per call). Rationale: ProphetFuzz uses only documentation as input, so faithful parsing of Groff markup — not crawling binaries or source — supplies all semantic context; parsing errors directly corrupt the 633-constraint extraction whose precision is measured at 94.00% overall. The few-shot examples (manpage_htmldoc., manpage_jhead., etc. under llm_interface/few-shot) show the expected structured format for downstream prompts.

## Related tools

- **ProphetFuzz** (Hosts the manpage_parser component (parser.py, utils/groff_utils.py) and restruct_manpage.py used to parse Groff documents into structured option descriptions) — https://github.com/NASP-THU/ProphetFuzz
- **Python** (Implementation language of the ~3k-line ProphetFuzz prototype, including the parsing scripts)

## Examples

```
bash run_all_in_one.sh bison  # full pipeline starting from man-page parsing; LLM-only mode: docker run -it 4ugustus/prophetfuzz_base bash
```

## Evaluation signals

- Parsed output must contain the program name (from .TH) and the expected sections (from .SH) for all 52 programs in the evaluation dataset
- Option description counts per program should match the man page contents (e.g., Image Magick has 305 options); missing options indicate incomplete .TP/.PP/.RS/.sp parsing
- Downstream invariant: constraint extraction on parsed records should reach ~94.00% overall precision and ~92.48% average per-program precision on the 52-program set
- For the 20-program annotated subset, parsed inputs must support extraction comparable to the 180 ground-truth constraints in assets/dataset/groundtruth_for_20_programs.
- Structured few-shot records (manpage_htmldoc., manpage_jhead., etc.) should be format-consistent with newly parsed programs

## Limitations

- Keyword matching on Groff macros can miss option descriptions with non-standard formatting, silently reducing recall of downstream constraint extraction
- Quality depends on documentation completeness — programs with sparse or ambiguous man pages yield fewer usable option descriptions
- Parsing is validated only on the 52 programs from 40 packages collected from three related studies; other documentation styles are untested

## Evidence

- [intro] Prophet Fuzz begins by extracting essential details such as the program's name, description, usage synopsis, and option descriptions through keyword matching: [evidence span withheld — anchor status: not_found]
