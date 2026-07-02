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
result must be readable by someone who has never seen the source language -
a product manager, an analyst, a student, an engineer from a different stack -
while remaining precise enough that an engineer could reimplement the logic
from it.

Hold both bars at once. If a non-programmer would stumble, the English has
failed. If an engineer could not rebuild the behavior, the structure has
failed. Every rule below exists to protect one of those two bars.

## The deliverable

Emit Pseudo inside a fenced code block tagged `python`. The tag is a
rendering trick, not a claim: the content is never actual Python, but the
`python` tag buys indentation-aware display and pleasant comment
highlighting in nearly every editor, chat client, and markdown viewer.

- Use four spaces per indentation level.
- Keep lines comfortably under about 80 characters.
- Start every self-contained block with an ALL-CAPS title comment.
- The Pseudo block is the deliverable. Surrounding prose should be at most
  a few sentences, reserved for things the block cannot carry: a bug you
  noticed, a caveat, a piece of missing context.

Every block, whatever its size, tends toward this skeleton:

```python
# TITLE OF THE LOGIC

# Optional voiceover: what problem this logic exists to solve.

Set up the starting state:
    Name each piece of state the way a human would.

While the work is not done (or: For each unit of work):
    # Voiceover for this phase, when the why is not obvious.
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

### 1. Every line is an English sentence

Imperative mood, ends with a period. No operators, no symbols, no
camelCase, no function-call syntax, no square-bracket indexing.

```python
# Bad
history.append(msg)
Set retries = retries - 1.
Check if usr.isActive.

# Good
Add the new message to the conversation history.
Decrement the retry counter.
Check whether the user's account is active.
```

Numbers and quoted literal values are allowed when they are the point:
"Wait exactly three seconds." "If the status code is 404:".

### 2. Indentation is control flow, and it is load-bearing

Four spaces of indent means "this happens inside that". This is the one
piece of formality Pseudo refuses to give up, because it is what makes the
logic visible at a glance. Never flatten branches into run-on prose like
"and then, if that works, also...". If two lines are at the same depth,
they run in sequence at the same level; if a line is indented under
another, the outer line governs it. Treat that promise as sacred - a reader
who cannot trust the indentation cannot trust anything.

### 3. Block openers end with a colon

Any line that governs an indented block ends with a colon, exactly like
Python. Keep to a small, stable vocabulary:

| Opener | Use for |
|---|---|
| `If ... :` | A conditional branch |
| `Otherwise if ... :` | The next branch (or `elif` in long chains) |
| `Otherwise:` | The final fallback branch |
| `For each ... :` | Iteration over a collection |
| `While ... :` | Iteration until a condition changes |
| `Define "name", given [params]:` | Defining a function, method, or class that exists in the source |
| `To <do something> ... :` | Defining an operation with no source identifier (design sketches, extracted sub-operations) |
| A descriptive phrase, e.g. `Validate the decision:` | Grouping a phase of related steps, like a section header |
| `After the loop ends:` / `When ... :` | Deferred or event-driven blocks |

`elif` is an accepted shorthand for `Otherwise if`, especially in long
state-machine chains where the repetition would drown the content.

### 4. Name things the way a human would

Refer to state with human noun phrases: "the retry counter", "the working
memory", "the user's latest message", "the running total". Never invent
variable names, and never carry cryptic names over from the source. If the
original calls it `usr_ctx_blob`, Pseudo calls it "the user's context".

The single exception is the `Define` opener, where the real identifier and
parameter names from the source appear verbatim (camelCase and all) so a
reader can jump between the translation and the code:

```python
Define "fetchWithRetry", given [url, attempts]:
```

Inside the body, those parameters go back to being human phrases ("the
resource's address", "the allowed number of attempts").

Two refinements:

- **Be consistent.** Once you call something "the frontier", it is "the
  frontier" for the rest of the block. Renaming state mid-block is how
  translations quietly become wrong.
- **Prefer role over type.** Not "the OrderedDict" but "the entries, kept
  in order from least to most recently used". The reader needs the job the
  data does, not the container it lives in.

### 5. Comments are voiceover, not narration

Comments (lines starting with `#`) are the narrator's voice over the
action. They explain **why** a section exists, what tension it resolves,
what invisible constraint it honors, or how it connects to the bigger
picture. They never restate what the line below already says.

```python
# Bad - narrates the obvious
# Update the counters
Update the elapsed time, token count, and iteration count.

# Good - explains the why and the stakes
# Budgets prevent the loop from becoming a cost or reliability hazard.
Update the elapsed time, token count, and iteration count.
```

Guidelines for placing voiceover:

- Put one at the top of each major phase whose purpose is not obvious.
- Spend your longest comment on the line most likely to confuse - the
  stale-entry check in Dijkstra, the request-identity comparison in a
  reducer, the pointer saved before a link is flipped. Comment budget
  should be proportional to reader confusion, not to code size.
- Great voiceover often names the ghost being guarded against: "Responses
  can arrive late and out of order; anything not matching the current
  request is a ghost from an abandoned fetch."
- If a whole block needs orientation, a two-or-three-line voiceover
  directly under the title is welcome.

### 6. Title the block

Start every self-contained block with an ALL-CAPS comment naming what it
is. A good title names the logic *and* its point when it can:

```python
# DEBOUNCE: COALESCE A BURST OF CALLS INTO ONE
# FETCH-STATE REDUCER: ONE STATE AT A TIME, STALE RESULTS IGNORED
# REVERSE A LINKED LIST IN PLACE, FLIPPING ONE LINK AT A TIME
```

The title turns the block into a citable, shareable artifact.

### 7. Make every exit explicit and name its reason

Code falls out of loops silently; Pseudo never does. Every way the logic
can end gets its own line, and the line names the reason:

```python
If the goal is complete:
    Stop with a goal-achieved terminal reason.

If the budget is exhausted:
    Save a checkpoint.
    Stop with a budget-exhausted terminal reason.

If all retries were used up:
    Stop with a retries-exhausted error and the last failure reason.
```

This applies to happy paths too ("Return the parsed result.") and to
non-obvious endings like "The entire match fails and nothing is captured."
If a reader asks "how does this end?", the answer must be a line they can
point at.

### 8. Make state changes and side effects visible

When something is mutated, persisted, sent, scheduled, or destroyed, say so
plainly: "Update the task state." "Persist the reminder." "Send the message
to the user." "Destroy the partial output stream." A reader should be able
to circle every line where the world changes.

When it matters, also make the **owner of a decision** visible: "Ask the
model whether...", "Let local policy decide whether...", "The caller
chooses...". Decision ownership is often the most important architectural
fact in a piece of code, and it is nearly always implicit in the source.

### 9. Break long enumerations vertically

When a step involves choosing from or listing several things, stack them
with trailing commas and a closing "or"/"and", indented under the opener:

```python
Choose the next action from:
    answer now,
    use a tool,
    ask the user a question,
    or finish.

Connect four stations into one assembly line:
    read the log file a chunk at a time,
    keep only the lines containing the error marker,
    compress whatever survives,
    and write the compressed result to the archive file.
```

This keeps lines short and makes the option space countable at a glance.

### 10. Match the reader's altitude, not the code's line count

Pseudo is a translation, not a transliteration. Collapse mechanical detail
into its meaning; expand dense one-liners into their real steps. The test:
could a smart person outside this codebase read the block once and explain
the logic back to you?

Here is the same fragment at three altitudes. Only one is right:

```python
# TOO LOW - transliterates the mechanics, drowns the meaning
Open the file.
Create a buffered reader over the file handle.
Initialize the line counter to zero.
While the reader has not reached the end of the file:
    Read one line into the line buffer.
    Increment the line counter.

# TOO HIGH - prose in a trench coat, structure carries nothing
Count the lines in the file.

# RIGHT - meaning first, structure where it earns its keep
For each line in the file:
    Add one to the running count.
Return the count.
```

When the whole function truly is one idea ("count the lines"), say so in
one line and move on - the altitude test cuts both ways.

## Vocabulary reference

Consistent vocabulary keeps blocks feeling like one language rather than
improvised prose. Prefer these phrasings:

**Ending things.** "Stop with a <named> terminal reason." / "Return the
<result>." / "The worker finishes." / "The entire match fails and nothing
is captured." / "Skip it; it is stale."

**Changing state.** "Update...", "Record...", "Mark... as...", "Add... to
...", "Remove... from...", "Remember...", "Forget...", "Become <state>"
(for state machines), "Stamp the arrival time."

**Side effects across a boundary.** "Send...", "Persist...", "Save...",
"Schedule...", "Write... to disk", "Ask the user...", "Report the failure
through the completion callback."

**Waiting and time.** "Wait briefly, a little longer each time." /
"...waiting if none is ready." / "Let the remaining work drain through."

**Quantities and comparisons.** "exceeds", "at most", "at least", "beats
the best known cost", "over capacity", "within the budget". Never `>`,
`<=`, `==`.

**Uncertainty and ownership.** "Ask the model whether...", "Let policy
decide...", "The caller provides...", "treating an unreadable file as
empty" (for silent fallbacks - always surface them).

## Domain playbook

Different kinds of code stress the notation differently. These are the
proven moves, domain by domain.

### Plain functions and algorithms

The bread and butter. One function becomes one block. Identify the skeleton
first - the loops, the branches, the state that changes, every way it can
end - then write the block top to bottom. Name the loop invariant if the
algorithm has one ("Keep track of three positions: the part already
reversed, the link being flipped, and the untouched remainder"), because
the invariant is usually the sentence that makes the algorithm click.

### Functions with names

When the source defines a named callable, mark the definition explicitly
with a `Define` opener: the real identifier in quotes, then the real
parameter names in square brackets. Omit `, given [...]` when there are no
parameters. The body indents beneath it, exactly like a Python `def`:

```python
# REVERSE A LINKED LIST IN PLACE

Define "reverse", given [head]:

    Where head is the first link of the chain.

    Keep track of three positions while walking the chain:
        ...
```

**Where clauses.** Never assume a parameter's meaning is obvious. Directly
under the opener, add one `Where <param> is ...` sentence per parameter
that a smart outsider could not confidently picture, then a blank line
before the body. A Where clause answers "what kind of thing is this, and
what role does it play here?" - never the source type:

```python
Define "dijkstra", given [graph, source]:

    Where graph is the map of the network: for every place, the list
    of roads leaving it and what each road costs to travel.
    Where source is the place all journeys start from.
```

Skip a clause only when the parameter is genuinely self-explanatory in
context; when in doubt, write it. A non-obvious return value carries the
same duty on the way out: "Return the table of best known costs, one per
place."

Use the `To ... :` recipe form only when there is no source identifier to
quote - a design sketched before code exists, or a sub-operation you
extracted for readability.

### Classes and objects

A class opens with `Define the "ClassName" class:`. Everything inside it
must be clearly one of two things - a voiceover comment or an operation.
So the object's big idea goes in a `#` comment directly under the opener,
and its starting state is communicated where it actually happens: in the
constructor, translated like any other method under its real name
(`__init__`, `constructor`, `new`, and so on). The constructor's body is
where the object's invariant gets stated, because the constructor is what
brings that invariant into existence. Then one `Define` per public
operation:

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
        # A lookup counts as a use, so the entry earns a reprieve.
        Move the entry to the most-recently-used end.
        Return the stored value.

    Define "put", given [key, value]:
        If the key is already in the cache:
            Move the existing entry to the most-recently-used end.
        Write the value under the key.
        If the cache is now over capacity:
            Evict the entry at the least-recently-used end.
```

Translate private helpers inline inside the operations that use them (a
helper that exists only for tidiness in the source does not deserve its own
`Define`). Order the operations by the story they tell, not by their order
in the source file.

### Recursion

Recursion reads naturally in Pseudo as long as you do three things: mark
the definition (with `Define` for source functions, `To ... :` for
sketches), state the base cases first, and phrase recursive steps as uses
of the operation's own verb:

```python
Define "deriv", given [expr, var]:

    If the formula is a plain number:
        The derivative is zero.

    If the formula is a sum of two parts:
        Build a new sum from:
            the derivative of the first part,
            and the derivative of the second part.
```

Never write "call this function recursively" - just use the operation's
verb ("the derivative of the first part") and let the structure carry it.

### Concurrency: threads, channels, workers

Concurrency is where Pseudo pays for itself, because the design is
distributed across syntax in the source. The moves:

- Give each concurrent actor its own block or opener ("Each worker, on its
  own thread, repeats forever:").
- Translate channels and queues as physical metaphors that keep their
  behavior: "a conveyor belt carrying file paths out to the workers."
- Make the shutdown protocol explicit; it is nearly always implicit in the
  source (a channel drop, a sentinel value, a closed file descriptor).
  "Closing the belt IS the shutdown signal. There is no stop message."
- Make mutual exclusion a sentence: "Only one worker at a time may reach
  into the job belt; a lock guarantees every file is claimed exactly once."

### Async: promises, callbacks, event loops

Do not translate the async machinery; translate the order of events.
"When the countdown finishes without being cancelled:" beats any sentence
containing the word "callback". For pipelines with backpressure or
buffering, say the guarantee out loud, because it is the point: "No station
is allowed to outrun the next; if the output is slow, the slowdown
propagates backward until even the reading pauses."

### Error handling and exceptions

Translate `try/except` as what actually happens, not as structure:

```python
Read the file, treating an unreadable file as empty.

If the response failed in a way that retrying could fix:
    Wait briefly, a little longer each time.
Otherwise:
    # Some failures (bad credentials, malformed input) will never
    # succeed on retry, so stop immediately instead of wasting attempts.
    Stop with a clear error that names the cause.
```

Two obligations: silent fallbacks must be surfaced ("treating an unreadable
file as empty" - the reader deserves to know errors are being swallowed),
and the retryable/fatal distinction must be visible whenever the source
makes one.

### Declarative languages: SQL, queries, dataflow

Declarative code says *what*, so Pseudo narrates the transformation as
stages, in the order a human would think them (which is rarely the order
SQL writes them). Group-then-rank-then-filter, not SELECT-FROM-WHERE.
Surface the semantics hiding in keywords: `PARTITION BY` becomes "the
ranking restarts per region, so small-region customers compete only with
each other"; `RANK()` ties become "ties share a rank, so top three can
return more than three."

### Regular expressions

A regex is a contract about the shape of text. Describe the shape top to
bottom with "expect, in this exact order:", one clause per component, and
say which parts are captured and which are deliberately skipped. Always end
with the brutal clause: "If the line deviates from this shape at any point,
the entire match fails and nothing is captured."

### State machines, reducers, and switch-heavy code

Lead with the state space as an enumeration ("The state is always exactly
one of: idle, loading a specific request, success..., or error...").
Translate transitions with "Become <state>" and guard clauses with "Keep
the current state unchanged." Long `elif` chains are fine here - the
repetition *is* the structure.

### Pointer and memory code

Name the positions, not the pointers. "The part already reversed", "the
link currently being flipped", "the untouched remainder" - then the pointer
dance becomes a walk with an invariant. Spend voiceover on ordering
constraints ("Save the way forward FIRST; flipping this link is exactly
what destroys the path to the rest of the chain") because ordering is what
pointer bugs are made of.

### Functional pipelines: map, filter, reduce

Either narrate as stages (like SQL) when the pipeline is long, or collapse
to meaning when it is short. `xs.filter(valid).map(price).reduce(sum)` is
just: "Add up the prices of the valid items." Do not manufacture a loop the
source does not conceptually have.

## Worked example

Source, a generic retry wrapper:

```javascript
async function fetchWithRetry(url, attempts = 3) {
  let delay = 200;
  for (let i = 0; i < attempts; i++) {
    try {
      const res = await fetch(url);
      if (res.ok) return await res.json();
      if (res.status >= 400 && res.status < 500) {
        throw new Error(`Client error ${res.status}`);
      }
    } catch (e) {
      if (i === attempts - 1) throw e;
    }
    await new Promise((r) => setTimeout(r, delay));
    delay *= 2;
  }
}
```

Translation:

```python
# FETCH WITH BOUNDED, BACKING-OFF RETRIES

Define "fetchWithRetry", given [url, attempts]:

    Where url is the address of the resource being requested.
    Where attempts is how many tries are allowed before giving up,
    three if the caller does not say.

    Set the waiting period to a fifth of a second.

    For each attempt, up to the allowed number of attempts:
        Request the resource.

        If the response is good:
            Return the parsed result.

        If the response is a client error (roughly, codes in the 400s):
            # A client error means the request itself is wrong. Retrying
            # the same wrong request can never succeed, so it is treated
            # as fatal... unless attempts remain, in which case this code
            # swallows it and retries anyway - see the note below.
            Raise a client-error failure.

        If this was the last allowed attempt:
            Re-raise whatever failure just occurred.

        # Backing off protects a struggling server from being hammered.
        Wait for the waiting period.
        Double the waiting period for next time.
```

And one sentence of prose after the block: "Note a likely bug: the
client-error failure is caught by the same net as network failures, so
client errors are retried anyway despite the clear intent that they should
be fatal; only the last attempt actually escapes."

That is the full pattern: faithful structure, human names, voiceover on
the confusing part, and the bug reported in prose - not silently fixed.

## Multi-block documents

When translating a whole file, class, or subsystem, produce several blocks
rather than one giant one:

- One block per meaningful operation or phase; skip trivial accessors and
  boilerplate entirely (say so in prose if their absence might surprise).
- Order the blocks by the story they tell together - typically lifecycle
  order (setup, main path, edge paths, teardown), not file order.
- One or two sentences of plain prose between blocks to hand the reader
  from one to the next is good practice.
- Nesting deeper than about four levels means the inner logic wants to be
  its own named block. Pull it out with a `To ... :` opener and use its
  verb where it was.

## Faithfulness

Pseudo is a translation, not an improvement. These obligations are
non-negotiable:

- **Preserve actual behavior, bugs included.** If you notice a bug, keep
  the translation faithful to what the code does, and report the bug in one
  or two sentences of prose after the block. A translation that silently
  fixes bugs is worse than useless - it certifies code that does not exist.
- **Translate only what you have read.** If the target calls something that
  materially shapes control flow and you cannot see it, read it first; if
  you truly cannot, say so explicitly rather than guessing.
- **Surface silent behavior.** Swallowed exceptions, default fallbacks,
  implicit type coercions that change outcomes - if the source hides them,
  the translation must not.
- **Keep the reader's trust in structure.** Never bend indentation for
  visual appeal. If the structure looks ugly, the code's structure is ugly,
  and that is information.

## Final quality checklist

Run this before delivering. Any hit means revise:

- [ ] A line containing `()`, `=`, `->`, `[i]`, a camelCase word, or an
      operator instead of words - anywhere except the quoted identifier
      and bracketed parameter list of a `Define` opener.
- [ ] An `If` with no visible alternative, where the alternative matters.
- [ ] A loop with no visible way to end.
- [ ] An exit that does not name its reason.
- [ ] A comment that repeats the line below it.
- [ ] State renamed mid-block ("the frontier" becoming "the queue").
- [ ] A silent fallback or swallowed error the translation does not surface.
- [ ] Nesting deeper than four levels.
- [ ] Real executable code mixed into the block.
- [ ] Prose paragraphs pretending to be Pseudo (if the indentation carries
      no information, it is not Pseudo yet).
- [ ] A missing ALL-CAPS title.
- [ ] A block a smart outsider could not read once and explain back.
