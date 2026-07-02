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
- Named callables from the source open with `Define "name", given [params]:`
  using the real identifier and parameter names, followed by one
  `Where <param> is ...` sentence per parameter whose meaning is not
  obvious. Classes open with `Define the "ClassName" class:`, put the
  object's big idea in a voiceover comment, and state starting state and
  invariant inside the constructor, translated like any other method.
- Name state in human terms ("the retry counter", "the user's latest
  message"), never with variable names. Two sanctioned exceptions: the
  `Define` opener, and an optional trailing bracketed anchor quoting the
  source expression (`Take the cheapest entry off the frontier.
  [frontier.pop]`) - the sentence must be complete with the anchor deleted.
- `#` comments are voiceover: they explain why a phase exists, never what
  the next line does.
- Every exit is explicit and names its reason ("Stop with a budget-exhausted
  terminal reason."). No silent fall-through.
- Wrap each block in a fenced code block tagged `python`, starting with an
  ALL-CAPS title comment naming the logic.
- Non-control-flow ideas may use the auxiliary registers, each in a `text`
  fence: aligned label-lists for enumerating a design space, equation
  lines (`=` and `+` only) for naming a thing's parts, and arrow lines
  (`STATE -> NEXT | OTHER`) for summarizing a state machine alongside -
  never instead of - its narrative block. Registers switch between
  blocks, never within a line.

Never write academic-style pseudocode (assignment arrows, BEGIN/END,
numbered algorithm lines) and never mix real executable code into a Pseudo
block.

The full specification with worked examples lives at
https://github.com/jrolf/pseudo/blob/main/SPEC.md. When in doubt about a
formatting decision, read it.
