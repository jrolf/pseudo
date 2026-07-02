# The Gallery

Each file in this folder is a complete worked example: real code in some
language, followed by its Pseudo translation, followed by a short note on
what the translation makes visible that the original hides.

The ten examples are deliberately drawn from ten different languages and ten
different kinds of logic, because that is the whole point - Pseudo reads the
same no matter what it was translated from. A closure, a query, a pointer
dance, and a regex all come out speaking the same language.

| # | Example | Source language | Domain | What it shows |
|---|---|---|---|---|
| 1 | [Debounce](01-javascript-debounce.md) | JavaScript | UI and timing | Cutting through closure and callback syntax |
| 2 | [LRU cache](02-python-lru-cache.md) | Python | Data structures | Making an invisible invariant visible |
| 3 | [Top customers query](03-sql-window-query.md) | SQL | Analytics | Pseudo works on declarative languages |
| 4 | [Fetch reducer](04-typescript-reducer.md) | TypeScript | State management | A race-condition guard hiding as boilerplate |
| 5 | [Worker pool](05-rust-worker-pool.md) | Rust | Concurrency | Channel closure as a shutdown protocol |
| 6 | [Stream pipeline](06-node-stream-pipeline.md) | Node.js | Streaming I/O | Backpressure, and a real bug caught in review |
| 7 | [Dijkstra](07-cpp-dijkstra.md) | C++ | Graph algorithms | The lazy-deletion trick everyone trips over |
| 8 | [Symbolic differentiation](08-lisp-symbolic-differentiation.md) | Lisp | Recursion and symbols | Calculus rules hiding in parentheses |
| 9 | [Linked list reversal](09-c-linked-list-reversal.md) | C | Pointers and memory | Naming the invariant that makes it safe |
| 10 | [Access log regex](10-regex-log-parsing.md) | Regex | Text patterns | A shape contract, described top to bottom |

To reproduce any of these yourself, install the skill from the repo README
and ask your agent: "Translate this to Pseudo."
