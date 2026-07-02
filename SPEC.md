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

The single exception is the `Define` opener (see "Defining functions,
methods, and classes"), where the real identifier and parameter names from
the source appear verbatim so readers can jump between the translation and
the code.

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

A class opens the same way, followed first by the invariant the object
maintains - its standing promise, stated as plain sentences - and then one
`Define` per public operation:

```python
# LEAST-RECENTLY-USED CACHE

Define the "LRUCache" class:

    The cache keeps its entries in order from least recently used to
    most recently used, and holds at most a fixed number of entries.

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
producing translations - vocabulary tables, an altitude guide, and a
domain-by-domain playbook covering classes, recursion, concurrency, async,
error handling, SQL, regexes, state machines, pointer code, and functional
pipelines - lives in the skill file at `.claude/skills/pseudo/SKILL.md`.
The two documents agree by construction; if they ever diverge, this spec
wins and the skill has a bug.

## Style anti-patterns to catch in review

- A line containing `()`, `=`, `->`, `[i]`, or a camelCase word. Rewrite it
  as a sentence. (The one sanctioned exception: the quoted identifier and
  bracketed parameter list in a `Define` opener.)
- An `If` with no visible alternative when the alternative matters. Add the
  `Otherwise:` or state that nothing happens.
- A loop with no visible way to end. Add the exit and name its reason.
- A comment that repeats the line below it. Delete it or promote it to a real
  voiceover.
- Nesting deeper than four levels. Pull the inner logic into its own named
  block ("Handle the failure:") and reference it.
- A block so long the title scrolls away. Split it into phases with plain
  descriptive openers.
