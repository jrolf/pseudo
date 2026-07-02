---
description: Translate code into Pseudo (plain-English, Python-shaped pseudocode)
argument-hint: [file, function, class, or description of what to translate]
---

Translate the following target into Pseudo: $ARGUMENTS

If no target was given, translate the code most recently discussed in this
conversation, or ask one focused question to identify it.

## What Pseudo is

Pseudo is pseudocode written as plain English sentences on a Python-shaped
skeleton. English carries the meaning; indentation carries the control flow;
nothing else is allowed in. The output must satisfy two bars at once: a
non-programmer can read it without stumbling, and an engineer could
reimplement the behavior from it.

## Procedure

1. Read the target code fully, including anything it calls that materially
   shapes its control flow. Never translate code you have not read. If a
   load-bearing dependency is unavailable, say so rather than guessing.
2. Identify the skeleton before writing: the loops, the branches, the state
   that changes, the side effects, and every way the logic can end.
3. Produce the translation as a fenced code block tagged `python` (the tag
   is a rendering trick for indentation; the content is never real Python).
4. Check the result against the quality checklist below and revise.
5. Keep surrounding prose to a few sentences at most, reserved for what the
   block cannot carry - a bug you noticed, a caveat, missing context.

## The rules

- Start the block with an ALL-CAPS title comment naming the logic and,
  when possible, its point ("DEBOUNCE: COALESCE A BURST OF CALLS INTO ONE").
- Every line is an imperative English sentence ending with a period. No
  operators, symbols, camelCase, indexing, or function-call syntax. Literal
  numbers and quoted values are fine when they are the point.
- Four spaces per indent level; indentation is the control flow and must
  be trustworthy. Block openers end with a colon: `If ... :`,
  `Otherwise if ... :` (or `elif` in long chains), `Otherwise:`,
  `For each ... :`, `While ... :`, or a descriptive grouping phrase
  ("Validate the input:").
- Named functions and methods from the source open with
  `Define "name", given [params]:` - the real identifier in quotes, the
  real parameter names in brackets (omit `, given [...]` when there are
  none). This is the one place source spelling is welcome; inside the
  body, everything goes back to human phrases. Use
  `To <do something> ... :` only for operations with no source
  identifier, such as design sketches.
- Directly under each `Define` opener, add one `Where <param> is ...`
  sentence per parameter a smart outsider could not confidently picture,
  then a blank line before the body. Explain the role the parameter
  plays, never its source type. When in doubt, write the clause.
- Classes open with `Define the "ClassName" class:`. The object's big
  idea goes in a voiceover comment under the opener; its starting state
  and invariant are stated in the constructor, translated like any other
  method under its real name (`__init__`, `constructor`, `new`). Then
  one `Define` per public operation.
- Name state in human terms and keep names consistent for the whole block:
  "the retry counter", "the user's latest message", "the frontier". Prefer
  the role something plays over the type it has.
- `#` comments are voiceover: they explain why a phase exists, what ghost
  it guards against, or what invisible constraint it honors. Never restate
  the line below. Spend the longest comment on the line most likely to
  confuse.
- Make every exit explicit and name its reason ("Stop with a
  budget-exhausted terminal reason." "The entire match fails and nothing
  is captured.").
- Make every state change and side effect visible, and when it matters,
  name who owns each decision ("Ask the model whether...", "Let local
  policy decide...").
- Break long enumerations vertically, one option per line with trailing
  commas and a closing "or"/"and".
- Match the reader's altitude: collapse mechanical detail into its meaning,
  expand dense one-liners into their real steps. When a whole function is
  one idea, say it in one line.

## Domain notes

- Classes: `Define the "ClassName" class:`, the invariant first, then one
  `Define` per public operation, ordered by story rather than file order.
- Recursion: mark the definition with `Define`, state base cases first,
  and phrase recursive steps using the operation's own verb.
- Concurrency: give each actor its own block, translate channels and queues
  as physical metaphors, and make the shutdown protocol explicit - it is
  nearly always implicit in the source.
- Errors: surface silent fallbacks ("treating an unreadable file as
  empty") and keep the retryable/fatal distinction visible.
- SQL and declarative code: narrate the transformation as stages in
  thinking order, and surface the semantics hiding in keywords.
- Regex: describe the shape top to bottom ("expect, in this exact
  order:"), note what is captured versus skipped, and state that any
  deviation fails the whole match.

## Faithfulness

Preserve the source's actual behavior, bugs included. If you notice a bug
or ambiguity, keep the block faithful and mention the issue in one or two
sentences of prose after it - never silently fix it in the pseudocode.

## Quality checklist before delivering

No operators or code syntax in any line; no `If` missing an alternative
that matters; no loop without a visible ending; no exit without a named
reason; no comment narrating the line below it; no state renamed mid-block;
no swallowed error left unsurfaced; no nesting past four levels; no real
code inside the block; no prose pretending to be Pseudo; title present.
