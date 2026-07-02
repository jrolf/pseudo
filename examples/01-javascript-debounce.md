# Example 1: A JavaScript Debounce Function

Debounce is a classic interview question and a classic source of confusion.
The code is short, but the closure, the timer juggling, and the `apply` call
make it opaque to anyone who does not live in JavaScript.

## The original

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

## The Pseudo translation

```python
# DEBOUNCE: COALESCE A BURST OF CALLS INTO ONE

Create a wrapper around the original function:
    Keep one shared countdown timer that survives between calls.

Whenever the wrapper is called:
    # Every new call resets the clock. Only silence lets the timer finish,
    # which is what turns a burst of calls into a single one.
    Cancel the countdown timer if one is running.
    Start a fresh countdown for the configured waiting period.

    If the wrapper is in fire-first mode and no countdown was running:
        # Fire-first mode responds instantly to the first call in a burst,
        # then ignores the rest of the burst instead of the start of it.
        Call the original function now with the caller's arguments.

When a countdown finishes without being cancelled:
    Mark that no countdown is running.
    If the wrapper is in fire-last mode:
        Call the original function with the most recent arguments.
```

## What the translation reveals

The original never says the two most important sentences out loud: "every
new call resets the clock" and "fire-first mode ignores the rest of the
burst, not the start of it". Those two facts are the entire reason debounce
exists and the entire difference between its two modes - and in the source,
both are implied by the interaction of `clearTimeout`, `setTimeout`, and a
boolean, rather than stated anywhere.
