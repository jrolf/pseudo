# Example 2: A Python LRU Cache

Even in a famously readable language, data-structure code hides its intent
in bookkeeping. Here is a minimal least-recently-used cache built on an
ordered dictionary.

## The original

```python
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity):
        self.capacity = capacity
        self.store = OrderedDict()

    def get(self, key):
        if key not in self.store:
            return None
        self.store.move_to_end(key)
        return self.store[key]

    def put(self, key, value):
        if key in self.store:
            self.store.move_to_end(key)
        self.store[key] = value
        if len(self.store) > self.capacity:
            self.store.popitem(last=False)
```

## The Pseudo translation

```python
# LEAST-RECENTLY-USED CACHE

The cache keeps its entries in order from least recently used to most
recently used, and holds at most a fixed number of entries.

Looking something up:
    If the key is not in the cache:
        Return nothing.
    # A lookup counts as a use, so the entry earns a reprieve from eviction.
    Move the entry to the most-recently-used end.
    Return the stored value.

Storing something:
    If the key is already in the cache:
        Move the existing entry to the most-recently-used end.
    Write the value under the key.

    If the cache is now over capacity:
        # The entry at the far end is, by construction, the one nobody
        # has touched for the longest time. It pays the price.
        Evict the entry at the least-recently-used end.
```

## What the translation reveals

The whole algorithm is one idea: **the order of the entries is the record of
recency, and eviction just harvests the far end.** In the source, that idea
is smeared across `move_to_end`, `popitem(last=False)`, and knowledge of how
`OrderedDict` behaves. In Pseudo, it fits in one voiceover comment, and the
two operations read as two short stories about the same invariant.
