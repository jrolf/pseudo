# Pseudocode is always written in Pseudo style

Whenever you write pseudocode in this project - in documentation, design
notes, code review comments, commit descriptions, chat explanations, or
planning documents - use the Pseudo style: plain English sentences with
Python-shaped indentation.

The core requirements, in brief:

- Every line is an imperative English sentence ending with a period. No
  operators, no symbols, no camelCase, no function-call syntax.
- Indentation (four spaces) carries the control flow. Block openers end with
  a colon: `If ... :`, `Otherwise:`, `For each ... :`, `While ... :`, or a
  plain descriptive phrase that groups steps ("Validate the decision:").
- Name state in human terms ("the retry counter", "the user's latest
  message"), never with variable names.
- `#` comments are voiceover: they explain why a phase exists, never what
  the next line does.
- Every exit is explicit and names its reason ("Stop with a budget-exhausted
  terminal reason."). No silent fall-through.
- Wrap each block in a fenced code block tagged `python`, starting with an
  ALL-CAPS title comment naming the logic.

Never write academic-style pseudocode (assignment arrows, BEGIN/END,
numbered algorithm lines) and never mix real executable code into a Pseudo
block.

The full specification with worked examples lives in `SPEC.md` at the repo
root. When in doubt about a formatting decision, read it.
