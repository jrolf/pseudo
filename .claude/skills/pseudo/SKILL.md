---
name: pseudo
description: >-
  Translate code into Pseudo, a pseudocode style written as plain English
  sentences with Python-shaped indentation, so anyone can understand the logic
  regardless of programming language. Use whenever the user asks for
  pseudocode, a plain-English explanation of how code works, a walkthrough of
  an algorithm, function, or class, or says "pseudo", "translate this to
  pseudo", or "explain this like I don't know the language".
---

# Pseudo: Python-Shaped English

You are translating code (or a design idea) into **Pseudo**: pseudocode where
every line is an English sentence and every indent is real control flow. The
result must be readable by someone who has never seen the source language,
while remaining precise enough that an engineer could reimplement the logic
from it.

## Output format

Always emit Pseudo inside a fenced code block tagged `python` (the tag buys
indentation-aware rendering everywhere; the content is never actual Python).
Start the block with an ALL-CAPS title comment naming the logic:

```python
# WHAT THIS LOGIC IS CALLED

Set up the starting state:
    Name each piece of state the way a human would.

While the work is not done:
    # Voiceover comments explain why a phase exists, never what a line does.
    Do the main step.

    If a decision point is reached:
        Take this branch.
    Otherwise:
        Take the other branch.

    If a terminal condition is met:
        Stop with a named reason.

Wrap up:
    Make the final state changes visible.
    Return or report the result.
```

## The ten rules

1. **Every line is an English sentence.** Imperative, ends with a period. No
   operators, no symbols, no camelCase, no function-call syntax. Never
   `history.append(msg)`; always "Add the new message to the conversation
   history."
2. **Indentation is control flow.** Four spaces per level. Nesting is the one
   piece of formality Pseudo keeps, because it makes logic visible at a
   glance. Never flatten branches into run-on prose.
3. **Block openers end with a colon.** `If ... :`, `Otherwise if ... :`,
   `Otherwise:`, `For each ... :`, `While ... :`, plus plain descriptive
   phrase openers that group steps like a section header ("Validate the
   decision:"). `elif` is accepted shorthand for `Otherwise if` in long
   state-machine chains.
4. **Name state the way a human would.** "The retry counter", "the user's
   latest message", "the running result". Never carry cryptic variable names
   over from the source.
5. **Comments are voiceover, not narration.** A `#` comment explains why a
   section exists, what tension it resolves, or how it connects to the whole.
   It never restates the line below it. Put one at the top of each major
   phase when the purpose is not obvious.
6. **Title the block** with an ALL-CAPS comment so it becomes a nameable,
   shareable artifact.
7. **Make every exit explicit and name its reason.** No silent fall-through.
   "Stop with a goal-achieved terminal reason." "Stop with a
   retries-exhausted error and the last failure reason."
8. **Make state changes and side effects visible.** Say plainly when
   something is updated, persisted, sent, or scheduled, and when it matters,
   who owns the decision ("Ask the model whether...", "Let local policy
   decide...").
9. **Break long enumerations vertically**, one option per line with trailing
   commas and a closing "or"/"and", indented under the opener.
10. **Match the reader's altitude.** Collapse mechanical detail into its
    meaning; expand dense one-liners into their real steps. The test: could a
    smart outsider read it once and explain the logic back to you?

## Translation procedure

When given a function, class, file, or selection to translate:

1. Read the source fully, including anything it calls that materially shapes
   the control flow. Understand it before writing a single Pseudo line.
2. Identify the skeleton: the loops, the branches, the state that changes,
   the side effects, and every way the logic can end.
3. Choose the altitude. One function usually becomes one block. A class or
   module becomes a short block per meaningful method or phase, ordered by
   the story they tell together, not by their order in the file.
4. Write the block following the ten rules. Draft the structure first, then
   add voiceover comments where a reader would otherwise ask "but why?"
5. Review against the anti-pattern list below and fix anything it catches.
6. After the block, add at most two or three sentences of surrounding prose
   if genuinely needed (for example, naming a caveat the pseudocode cannot
   carry). The Pseudo block itself is the deliverable.

## Anti-patterns to catch before delivering

- A line containing `()`, `=`, `->`, `[i]`, or a camelCase word.
- An `If` with no visible alternative when the alternative matters.
- A loop with no visible way to end.
- A comment that repeats the line below it.
- Nesting deeper than four levels (pull the inner logic into its own named
  block and reference it).
- Real executable code mixed into the block.
- Prose paragraphs pretending to be Pseudo (if the indentation carries no
  information, it is not Pseudo yet).

## Faithfulness

Pseudo is a translation, not an improvement. Preserve the source's actual
behavior, including its bugs and quirks; if you notice a bug, mention it in
prose after the block rather than silently fixing it in the pseudocode. If
the source is ambiguous or you cannot see a called function that matters,
say so explicitly rather than guessing.
