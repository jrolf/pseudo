# Pseudo

```text
               __  _  __                          \      |      /
         _.--~~  \ | /  ~~--._               .     \     |     /     .
       .'   .--~ \\|// ~--.   '.                      __..---..__
      /   .'   .-(o o)-.   '.   \       - -- --    .-'           '-.    -- -
     ;   /    /   \\|//   \    \  ;     - -- -    /                 \    -- -
     '  '    '     \|/     '    ' '               ;                   ;
                    |         ~^~ ~~^~ ^~~ ~^~ ~^|                   |^~ ~~^
                    |        ^~ ~^~~ ^~^ ~~^ ~^~^ \                 / ~^~ ^~
                   (|       ~^~ ~~^~ ^~~^~ ~^~ ~~^ '-..___ _ ___..-' ^~ ~~^~
                    |)     ~~^ ~^~^ ~~^~ ^~~^ ~^~ ~^~ ~^ ': : :' ~^ ~^~ ^~~
                   (|     ^~ ~^~~^ ~^~ ~^~~ ^~~^~ ~^~~^~ ~ ': :' ~ ~^~^ ~~^
                    |
     _______________|________________________________________________________
       .  '  .  .      ' .   .      p s e u d o      .   '  .   ` .  '  .  '
```

**Point at any code, in any language, and get back logic that anyone can read.**

Pseudo is a pseudocode style with one big idea: write every line as a plain
English sentence, and keep the indentation structure of Python. English
carries the meaning. Indentation carries the control flow. Nothing else is
allowed in.

The result is something like an Esperanto for logic. A JavaScript closure, a
SQL window query, a Rust state machine, and a COBOL batch job all translate
into the same notation - and that notation is readable by your product
manager, your new hire, your security reviewer, and your future self who
forgot how any of this works.

This repo packages the style as a **Claude Code skill, rule, and slash
command** that you (or your AI coding agent) can install in about a minute.
They work in Cursor too.

## See it before you believe it

Here is a debounce function, the kind of thing that shows up in every
frontend codebase and every interview:

```javascript
function debounce(fn, wait, immediate) {
  let timeout;
  return function (...args) {
    const context = this;
    const later = function () {
      timeout = null;
      if (!immediate) fn.apply(context, args);
    };
    const callNow = immediate && !timeout;
    clearTimeout(timeout);
    timeout = setTimeout(later, wait);
    if (callNow) fn.apply(context, args);
  };
}
```

And here is what your agent hands back when you ask for Pseudo:

```python
# DEBOUNCE: COALESCE A BURST OF CALLS INTO ONE

Define "debounce", given [fn, wait, immediate]:

    Where fn is the function being protected from over-calling.
    Where wait is how long the silence must last, in milliseconds,
    before the function actually fires.
    Where immediate, when set, switches the wrapper into fire-first mode.

    Build and return a wrapper around the original function,
    sharing one countdown timer that survives between calls.

    Whenever the wrapper is called:
        # Every new call resets the clock. Only silence lets the timer
        # finish, which is what turns a burst of calls into a single one.
        Cancel the countdown timer if one is running.
        Start a fresh countdown for the configured waiting period.

        If the wrapper is in fire-first mode and no countdown was running:
            # Fire-first mode responds instantly to the first call in a
            # burst, then ignores the rest of the burst instead of the
            # start of it.
            Call the original function now with the caller's arguments.

    When a countdown finishes without being cancelled:
        Mark that no countdown is running.
        If the wrapper is in fire-last mode:
            Call the original function with the most recent arguments.
```

Read that second block again. You did not need to know JavaScript. You did
not need to know what `apply` does or why `timeout = null` matters. And yet
nothing was dumbed down: the two modes, the timer reset, the exact moment
the real function fires - it is all there, more clearly than the source
states it. The comments are not narration; they are a voiceover explaining
*why* each part exists.

That is the whole trick, and it works on everything. The
[examples gallery](examples/README.md) proves it across ten languages and
ten domains: a Python merge algorithm, SQL analytics, a TypeScript reducer,
a Rust worker pool, a Node.js stream pipeline, Dijkstra in C++, symbolic
differentiation in Lisp, pointer surgery in C, and a regex nobody wants to
read - all coming out in the same plain notation.

## Why this is a superpower

**If you write code:** you can finally show your work. Drop a Pseudo block
into a PR description and watch review time collapse. Sketch a design in
Pseudo before implementing and let the team argue about logic instead of
syntax. Inherit a legacy codebase in a language you barely know and read it
anyway.

**If you don't write code:** this is your decoder ring. Product managers can
verify that the code actually implements the spec. Analysts can audit the
query someone else wrote. Anyone can point an AI agent at any function in
the company codebase and ask "what does this really do?" - and get an answer
that respects both their intelligence and the code's actual behavior.

**If you work with AI agents (everyone, now):** Pseudo is a shared language
between you and the machine. Agents write it fluently, humans read it
effortlessly, and it survives translation in both directions - you can
review an agent's plan in Pseudo before it writes a line of real code, and
you can hand an agent a Pseudo design and say "build this."

Programming languages optimize for machines. Pseudo optimizes for the
conversation *about* the machine. Most of software engineering turns out to
be that conversation.

## What's in this repo

```
pseudo/
├── README.md                  You are here
├── SPEC.md                    The canonical style specification (the ten rules)
├── examples/                  Ten worked translations across ten languages:
│                              JS, Python, SQL, TypeScript, Rust, Node,
│                              C++, Lisp, C, and regex
├── skills/pseudo/SKILL.md     The skill: the full translation doctrine,
│                              rules, vocabulary, and domain playbook
├── rules/pseudo-style.md      The rule: makes Pseudo the house pseudocode style
└── commands/pseudo.md         The command: /pseudo <target> on demand
```

Three artifacts, three jobs:

- The **skill** is the workhorse. Once installed, your agent knows the full
  style and applies it whenever you ask for pseudocode or an explanation of
  how code works. It also gives you a `/pseudo` slash command automatically.
- The **rule** is for teams. Drop it into a project and every agent session
  in that repo writes its pseudocode in Pseudo style - in docs, PR
  descriptions, design notes, everywhere - without being asked.
- The **command** is the minimal, legacy-format version for anyone who just
  wants `/pseudo` and nothing else. (If you install the skill, you can skip
  this - skills supersede commands and provide the same slash command.)

## Installation

### The easy way (recommended): let your agent install it

Open Claude Code or Cursor and paste this:

> Look at https://github.com/jrolf/pseudo and install the Pseudo skill from
> it. Copy `skills/pseudo/SKILL.md` into my personal skills directory so it
> works in all my projects.

That's it. Your agent will fetch the skill and put it in the right place.
Then try it:

> Translate the gnarliest function in this repo to Pseudo.

### Manual install: Claude Code

Personal (all your projects):

```bash
mkdir -p ~/.claude/skills/pseudo
curl -o ~/.claude/skills/pseudo/SKILL.md \
  https://raw.githubusercontent.com/jrolf/pseudo/main/skills/pseudo/SKILL.md
```

Per-project (shared with your team via git): put the same file at
`.claude/skills/pseudo/SKILL.md` inside your repo.

Want Pseudo to be the default pseudocode style for a whole project? Also
copy the rule:

```bash
mkdir -p .claude/rules
curl -o .claude/rules/pseudo-style.md \
  https://raw.githubusercontent.com/jrolf/pseudo/main/rules/pseudo-style.md
```

New sessions in that project will pick both up automatically. Type `/pseudo`
or just ask for pseudocode.

### Manual install: Cursor

Cursor reads skills from `~/.cursor/skills/`:

```bash
mkdir -p ~/.cursor/skills/pseudo
curl -o ~/.cursor/skills/pseudo/SKILL.md \
  https://raw.githubusercontent.com/jrolf/pseudo/main/skills/pseudo/SKILL.md
```

For a project rule in Cursor, paste the contents of `rules/pseudo-style.md`
into your project's `AGENTS.md` or a `.cursor/rules/` rule file.

### No install at all

You can use Pseudo today with any capable AI assistant by pointing it at the
spec:

> Read https://github.com/jrolf/pseudo/blob/main/SPEC.md and then translate
> the following code into Pseudo: ...

## Things to try once it's installed

- "Translate this function to Pseudo" - the bread and butter.
- "Give me a Pseudo walkthrough of this entire file, one block per phase."
- "Write the Pseudo for the feature we just discussed, before any real code."
- "Translate this SQL query to Pseudo so I can send it to the finance team."
- "Review this PR and include a Pseudo block showing what the change does."
- "Here's my Pseudo design. Implement it in Python."

That last one is the sleeper feature: Pseudo runs in both directions. It is
a reading language and a writing language.

## The ten rules at a glance

1. Every line is an English sentence.
2. Indentation is control flow, and it is load-bearing.
3. Block openers end with a colon (`If ... :`, `Otherwise:`, `For each ... :`, `While ... :`).
4. Name things the way a human would - never variable names.
5. Comments are voiceover (why), not narration (what).
6. Title every block with an ALL-CAPS comment.
7. Make every exit explicit and name its reason.
8. Make state changes and side effects visible.
9. Break long enumerations vertically.
10. Match the reader's altitude, not the code's line count.

The full specification, with good/bad examples for each rule and the
anti-pattern checklist, is in [SPEC.md](SPEC.md).

## Where this came from

This style emerged from a practical problem: comparing dozens of AI agent
architectures side by side. Real implementations were in different languages
and frameworks; prose summaries lost the control flow that made the designs
different in the first place. What survived was this middle notation -
English sentences on a Python skeleton - and it turned out to be the most
useful artifact of the whole exercise. Once you have a notation where a
non-programmer and a compiler engineer can point at the same line and mean
the same thing, you start using it for everything.

## Contributing

The best contribution is a great example: a piece of real-world code whose
Pseudo translation reveals something the source hides. Open a PR adding a
file to `examples/` following the existing format (original, translation,
what it reveals). Refinements to the spec are welcome too, with the caveat
that the style's power comes from staying small - proposals that add
notation face a high bar.

## License

MIT. Take it, use it, teach it, ship it.
