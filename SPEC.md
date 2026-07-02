# The Pseudo Specification

Pseudo is a way of writing pseudocode that reads as plain English but keeps the
structural skeleton of Python. Every line is a sentence a non-programmer can
read. Every indent is real control flow a programmer can trust.

We call the result **Python-shaped English**: indented like Python, written
like prose. It is intentionally not executable. Its job is to reveal how a
piece of logic actually works - the control flow, the state changes, the
decisions, the exits - without requiring the reader to know any particular
programming language.

This document is the canonical definition of the style. If you are an AI agent
that has been asked to "write Pseudo" or "translate this to Pseudo", this file
is your ground truth.

---

## A complete example first

Before any rules, here is what Pseudo looks like. This is a translation of a
generic retry wrapper around a network call:

```python
# BOUNDED RETRY AROUND A NETWORK CALL

Build the request from the current inputs.
Set the retry counter to the maximum number of attempts.

While retries remain:
    Send the request.

    If the response is valid:
        Return the parsed result.

    If the response failed in a way that retrying could fix:
        # Waiting between attempts prevents hammering a struggling server.
        Wait briefly, a little longer each time.
        Decrement the retry counter.
        Continue to the next attempt.

    Otherwise:
        # Some failures (bad credentials, malformed input) will never
        # succeed on retry, so we stop immediately instead of wasting attempts.
        Stop with a clear error that names the cause.

If all retries were used up:
    Stop with a retries-exhausted error and the last failure reason.
```

Notice what just happened: a product manager can read that top to bottom and
understand the retry policy. An engineer can see the exact control flow,
including both terminal states. Nobody needed to know which language the
original was written in.

---

## The ten rules

### 1. Every line is an English sentence

Write sentences, not code. Statements are imperative ("Send the request."),
end with a period, and use no operators, no camelCase, no symbols.

```python
# Good
Add the new message to the conversation history.

# Bad
history.append(msg)
```

### 2. Indentation is control flow, and it is load-bearing

Four spaces of indent means "this happens inside that". This is the one piece
of formal structure Pseudo refuses to give up, because it is the piece that
makes logic visible at a glance. Never flatten nesting into prose like "and
then, if that works, also...".

### 3. Block openers end with a colon

Any line that governs an indented block ends with a colon, exactly like
Python. The openers are a small, stable vocabulary:

```python
If the plan is invalid twice:
Otherwise if the user interrupted:
Otherwise:
For each step in the plan:
While the task is not done:
Initialize the run:
After the loop ends:
```

Conditionals use `If` / `Otherwise if` / `Otherwise`. (Python's `elif` is an
accepted shorthand for `Otherwise if`, especially in long state-machine
chains.) Loops use `For each ... :` and `While ... :`. Named functions,
methods, and classes open with `Define` (see "Defining functions, methods,
and classes" below). You may also open a block with a plain descriptive
phrase ("Validate the decision:") to group related steps, the way a section
header would.

### 4. Name things the way a human would

Refer to state with human noun phrases: "the running result", "the retry
counter", "the working memory", "the user's latest message". Never invent
variable names, and never carry cryptic names over from the source code. If
the original calls it `usr_ctx_blob`, Pseudo calls it "the user's context".

Two exceptions are sanctioned: the `Define` opener (see "Defining
functions, methods, and classes"), where the real identifier and parameter
names from the source appear verbatim so readers can jump between the
translation and the code, and the optional trailing anchor (see "The
anchor extension"), which quotes a source expression in brackets at the
end of a sentence.

### 5. Comments are voiceover, not narration

Comments (lines starting with `#`) explain **why** a section exists, what
tension it resolves, or how it connects to the bigger picture. They are the
narrator's voice over the action. They never restate what the line below
already says.

```python
# Good - explains the why and the stakes
# Budgets prevent the loop from becoming a cost or reliability hazard.
Update the elapsed time, token count, and iteration count.

# Bad - narrates the obvious
# Update the counters
Update the elapsed time, token count, and iteration count.
```

A well-placed voiceover comment at the top of each major phase is often the
difference between pseudocode that is merely accurate and pseudocode that
actually teaches.

### 6. Title the block

Start every self-contained Pseudo block with an ALL-CAPS comment naming what
it is:

```python
# PLAN AND EXECUTE LOOP
```

This turns each block into a citable, shareable artifact with a name.

### 7. Make every exit explicit and name its reason

Code falls out of loops silently; Pseudo never does. Every way the logic can
end gets its own line, and the line names the reason:

```python
If the goal is complete:
    Stop with a goal-achieved terminal reason.

If the budget is exhausted:
    Save a checkpoint.
    Stop with a budget-exhausted terminal reason.
```

### 8. Make state changes and side effects visible

When something is mutated, persisted, sent, or scheduled, say so plainly:
"Update the task state.", "Persist the reminder.", "Send the message to the
user." A reader should be able to circle every line where the world changes.
When it matters, also make it clear **who** decides ("Ask the model whether...",
"Let local policy decide whether...").

### 9. Break long enumerations vertically

When a step involves choosing from or listing several things, stack them with
trailing commas and a closing "or" / "and", indented under the opener:

```python
Choose the next action from:
    answer now,
    use a tool,
    ask the user a question,
    or finish.
```

This keeps lines short and makes the option space countable at a glance.

### 10. Match the reader's altitude, not the code's line count

Pseudo is a translation, not a transliteration. Collapse mechanical detail
("open the file, create a reader, iterate the reader") into its meaning
("For each row in the file:"). Expand dense one-liners into their actual
steps. The test is always: could a smart person outside this codebase read
the block once and explain the logic back to you?

---

## Defining functions, methods, and classes

When the source defines a named callable, Pseudo marks the definition
explicitly rather than hiding it behind a descriptive phrase. The opener is:

```python
Define "name", given [param, param]:
```

Three details make this work:

- The quoted name is the **real identifier from the source, verbatim**.
  This is the one place where source spelling (camelCase and all) is
  welcome inside Pseudo, because it lets a reader jump between the
  translation and the code.
- The square brackets list the **real parameter names**, comma-separated.
  Omit `, given [...]` entirely when there are no parameters.
- The body is indented beneath the opener, exactly like a Python `def`.

### Explain the parameters with Where clauses

Never assume a parameter's meaning is obvious. Directly under the opener,
add one `Where <param> is ...` sentence per parameter that a smart outsider
could not confidently picture, then a blank line before the body begins:

```python
Define "dijkstra", given [graph, source]:

    Where graph is the map of the network: for every place, the list
    of roads leaving it and what each road costs to travel.
    Where source is the place all journeys start from.

    Record the best known cost to reach every place:
        ...
```

Where clauses answer "what kind of thing is this, and what role does it
play here?" - not the source type. Skip a clause only when the parameter
is genuinely self-explanatory in context; when in doubt, write it. The
same goes for a non-obvious return value: a closing line like "Return the
table of best known costs, one per place." carries the same duty on the
way out.

### Classes

A class opens with `Define the "ClassName" class:`. Everything inside it
must be clearly one of two things - a voiceover comment or an operation -
so the object's big idea goes in a `#` comment, and its starting state is
communicated where it actually happens: in the constructor, translated
like any other method under its real name (`__init__`, `constructor`,
`new`, and so on). The constructor's body is where the object's invariant
gets stated, because the constructor is what brings that invariant into
existence:

```python
# LEAST-RECENTLY-USED CACHE

Define the "LRUCache" class:

    # One idea runs this whole object: the order of the entries IS the
    # record of recency, so eviction only ever harvests the far end.

    Define "__init__", given [capacity]:
        Where capacity is the most entries the cache may hold at once.

        Remember the capacity.
        Start with an empty collection of entries, kept in order from
        least recently used to most recently used.

    Define "get", given [key]:
        If the key is not in the cache:
            Return nothing.
        # A lookup counts as a use, so the entry earns a reprieve
        # from eviction.
        Move the entry to the most-recently-used end.
        Return the stored value.

    Define "put", given [key, value]:
        If the key is already in the cache:
            Move the existing entry to the most-recently-used end.
        Write the value under the key.

        If the cache is now over capacity:
            # The entry at the far end is, by construction, the one
            # nobody has touched for the longest time. It pays the price.
            Evict the entry at the least-recently-used end.
```

Private helpers that exist only for tidiness in the source are translated
inline inside the operations that use them; a helper earns its own `Define`
only when it carries an idea of its own.

When there is no source identifier to quote - sketching a design before any
code exists, or naming a sub-operation you extracted for readability - use
the recipe form instead: `To differentiate a formula with respect to a
variable:`. The `To ... :` opener signals "this is a callable operation"
without inventing a fake identifier.

## Anatomy of a Pseudo block

Most Pseudo blocks, whatever their size, follow the same skeleton:

```python
# TITLE OF THE LOGIC

# Optional voiceover: what problem this logic exists to solve.

Set up the starting state:
    Name each piece of state a human way.

For each unit of work (or: While the condition holds):
    # Voiceover for this phase when the why is not obvious.
    Do the main step.

    If a decision point is reached:
        Take the branch.
    Otherwise:
        Take the other branch.

    If a terminal condition is met:
        Stop with a named reason.

Wrap up:
    Make the final state changes visible.
    Return or report the result.
```

## The auxiliary registers

The narrative register - sentence lines on a Python skeleton - is the
default, and the only register for control flow. But some ideas that come
up constantly when describing code are not control flow, and forcing them
into indented sentences serves nobody. For those, Pseudo has four auxiliary
registers and one document template. Each has exactly one job, and the
governing principle is strict: **many registers, hard boundaries, one
grammar.** Registers switch between blocks, never within a line. Within a
line you are always in sentence mode. The shared grammar - trustworthy
indentation, named exits, voiceover comments, human naming - holds
everywhere.

### The anchor extension

A sentence line may carry an optional bracketed anchor at the end of the
line, tying it back to the exact expression in the source:

```python
Take the cheapest waiting entry off the frontier.  [frontier.pop]
If this entry is worse than the best known cost for its place:  [cost > dist[node]]
    Skip it; it is stale.
```

A non-programmer reads straight past the brackets; an engineer gets a jump
table into the code. Two rules keep anchors from decaying into code:
the sentence must be complete and correct with the anchor deleted, and
anchors never appear in comments, `Where` clauses, or block titles. Use
them when the reader is likely to move between the translation and the
source - code review, debugging walkthroughs, onboarding docs - and skip
them when the audience will never open the code.

### The taxonomy register

For enumerating a design space, an option menu, or a family of cases,
aligned label-lists beat indented sentences because they are scannable and
countable. Write them in a `text` fence:

```text
Retry:      The same action again, bounded, with repair hints.
Replan:     Discard the remaining plan; build a new one from current state.
Backtrack:  Return to an earlier choice point; try a sibling branch.
Escalate:   Ask a stronger model, another agent, or a human.
```

Rules: the label is a short noun or verb phrase, the description is one
sentence, and there is no nesting. If you feel the need to nest, the
content is control flow in disguise and belongs in the narrative register.

### The algebra register

For compositional definitions - what something is made of - one equation
line beats a paragraph:

```text
Thought  =  Prompt + Context + LLM + Parsing + Validation
Agent    =  Trigger fabric + Memory + Loop + Ledger
```

Rules: only `=` and `+`, every term is a capitalized human noun phrase, and
every term is defined somewhere nearby. Algebra defines parts; it never
describes sequence. "A then B" is narrative, not algebra.

### The transition register

For state machines whose full narrative treatment would drown the reader
in `elif`, arrow lines summarize the map:

```text
ASSESS  ->  RESPOND | PLAN | SLEEP
PLAN    ->  DISPATCH | RESPOND
RESPOND ->  UPDATE_MEMORY | EXIT
```

Rule: the transition register may only ever *summarize* a machine that
also receives a narrative block. The arrows show the shape; the narrative
shows what happens inside each state and why. Neither alone is complete.

### The pattern card

When a document describes several algorithms, systems, or design patterns
side by side, wrap each one in a fixed card so any two can be compared
field by field:

```text
Core idea:  One sentence naming the bet this design makes.
Anatomy:    An algebra line naming the parts.
Exits:      The named terminal states, separated by pipes.
[The narrative Pseudo block.]
Failure modes / caveats:  A taxonomy block or a short prose note.
```

Fields may be dropped when they carry nothing; the card is a comparison
instrument, not a bureaucratic form.

## Choosing the resolution

Pseudo describes logic at whatever altitude the reader needs, and the
biggest translation decision - bigger than any formatting rule - is
choosing that altitude deliberately. Four named resolutions cover the
useful range:

```text
Trace level:      Nearly line-for-line. Every branch and mutation of the
                  source appears. For audits, debugging, and code review.
Function level:   One block per function, mechanical detail collapsed to
                  its meaning. The default for "translate this function".
Component level:  One block per meaningful operation or phase of a file
                  or class; trivial plumbing omitted and said to be
                  omitted. For onboarding and design review.
System level:     One page for many files: how the pieces talk, where
                  state lives, what triggers what. Individual functions
                  appear only as single sentences or algebra terms.
```

The resolution is the reader's choice, not the translator's convenience.
When a request spans multiple files or an entire subsystem and the desired
resolution is not stated, discover it - ask one focused question ("do you
want a near line-for-line logic map, or a one-page picture of how it all
works?") or state the assumption being made and offer the alternative. A
system-level overview that ends by offering to drill into any block at
trace level is often the strongest answer of all.

## Fencing and formatting

Write Pseudo inside fenced code blocks tagged `python`. The tag is a trick:
it is not Python, but the `python` tag gives you indentation-aware rendering
and pleasant comment highlighting in nearly every editor, chat client, and
markdown viewer on earth.

Use four spaces per indent level. Keep lines comfortably under about 80
characters; if a sentence runs long, that is usually a sign it should become
two lines or a vertical enumeration (rule 9).

## What Pseudo is not

- **Not runnable.** Never mix real code into a Pseudo block. The moment one
  line requires a compiler to understand, every line loses its innocence.
- **Not a summary.** A three-sentence paragraph describing code is prose, not
  Pseudo. If the indentation carries no information, it is not Pseudo yet.
- **Not academic pseudocode.** No `←` assignment arrows, no `BEGIN`/`END`,
  no mathematical notation, no numbered algorithm lines. Those styles optimize
  for journals; Pseudo optimizes for humans reading on screens.

## How Pseudo differs from typical pseudocode

Most pseudocode found in the wild fails in one of two directions. It stays too
close to code (variable names, operators, and syntax survive, so it only reads
well if you already know the language), or it drifts too far into prose (the
structure dissolves, so the actual control flow becomes ambiguous). Pseudo
holds the line between the two on purpose:

| Property | Typical "codey" pseudocode | Typical prose summary | Pseudo |
|---|---|---|---|
| Line content | `if x > thresh: flag(x)` | "It flags large values" | "If the value exceeds the threshold: Flag it for review." |
| Control flow | Precise but unreadable | Lost | Precise and readable |
| State names | `x`, `tmp`, `ctx` | Usually absent | "the value", "the review queue" |
| Exits | Implicit fall-through | Vague | Named terminal reasons |
| Comments | Rare, narrate syntax | n/a | Voiceover: why, not what |

## Beyond this spec

This document defines the style. The operational doctrine for actually
producing translations - vocabulary tables, guidance for choosing the
resolution and the register mix, and a domain-by-domain playbook covering
classes, recursion, concurrency, async, error handling, SQL, regexes,
state machines, pointer code, and functional pipelines - lives in the
skill file at `skills/pseudo/SKILL.md`.
The two documents agree by construction; if they ever diverge, this spec
wins and the skill has a bug.

## Style anti-patterns to catch in review

- A narrative line containing `()`, `=`, `->`, `[i]`, or a camelCase word.
  Rewrite it as a sentence. (Two sanctioned exceptions: the quoted
  identifier and bracketed parameter list in a `Define` opener, and a
  trailing bracketed anchor whose deletion leaves the sentence complete.
  Auxiliary-register blocks in `text` fences have their own grammar.)
- An `If` with no visible alternative when the alternative matters. Add the
  `Otherwise:` or state that nothing happens.
- A loop with no visible way to end. Add the exit and name its reason.
- A comment that repeats the line below it. Delete it or promote it to a real
  voiceover.
- Nesting deeper than four levels. Pull the inner logic into its own named
  block ("Handle the failure:") and reference it.
- A block so long the title scrolls away. Split it into phases with plain
  descriptive openers.
