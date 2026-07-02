---
description: Translate code into Pseudo (plain-English, Python-shaped pseudocode)
argument-hint: [file, function, class, or description of what to translate]
---

Translate the following target into Pseudo: $ARGUMENTS

If no target was given, translate the code most recently discussed in this
conversation, or ask one focused question to identify it.

Pseudo is pseudocode written as plain English with Python-shaped indentation.
Follow these steps:

1. Read the target code fully, including anything it calls that materially
   shapes its control flow. Do not translate code you have not read.
2. Produce the translation as a fenced code block tagged `python`, obeying
   the Pseudo rules:
   - Start with an ALL-CAPS title comment naming the logic.
   - Every line is an imperative English sentence ending with a period; no
     operators, symbols, camelCase, or function-call syntax.
   - Indentation (four spaces) carries the control flow; block openers end
     with a colon (`If ... :`, `Otherwise:`, `For each ... :`,
     `While ... :`, or a descriptive grouping phrase like "Validate the
     input:").
   - Name state in human terms, never with variable names from the source.
   - Add `#` voiceover comments at the top of major phases explaining why
     the phase exists - never comments that restate the next line.
   - Make every exit explicit and name its reason; make every state change
     and side effect visible.
   - Break long option lists vertically, one per line.
3. Keep the altitude right: collapse mechanical detail into its meaning,
   expand dense one-liners into their real steps. A smart person who has
   never seen this programming language should be able to read the block
   once and explain the logic back.
4. Preserve the source's actual behavior, bugs included. If you notice a
   bug or ambiguity, mention it in one or two sentences of prose after the
   block instead of silently fixing it.

The Pseudo block is the deliverable. Keep surrounding prose to a minimum.
