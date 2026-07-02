# Example 2: Merging Overlapping Intervals in Python

A staple of calendar apps, booking systems, and coding interviews: given a
pile of time spans in no particular order, collapse every overlapping group
into a single span. The code is short; the reason it works is a one-line
insight the code never states.

## The original

```python
def merge_intervals(intervals):
    if not intervals:
        return []
    intervals.sort(key=lambda span: span[0])
    merged = [intervals[0]]
    for start, end in intervals[1:]:
        last_end = merged[-1][1]
        if start <= last_end:
            merged[-1][1] = max(last_end, end)
        else:
            merged.append([start, end])
    return merged
```

## The Pseudo translation

```python
# MERGE OVERLAPPING INTERVALS INTO THE FEWEST POSSIBLE SPANS

Define "merge_intervals", given [intervals]:

    Where intervals is a list of spans, each with a start and an end,
    arriving in no particular order - like meetings on a calendar.

    If there are no spans at all:
        Return an empty list.

    # Sorting by start is the insight that makes everything after it
    # easy: once the spans are in start order, a new span can only ever
    # overlap the most recent merged span, never one further back.
    Sort the spans by their start.

    Start the merged list with the earliest span.

    For each remaining span, in sorted order:
        If the span starts before or exactly when the last merged
        span ends:
            # The two overlap or touch, so they become one. Taking the
            # later of the two ends handles spans swallowed whole.
            Extend the last merged span to the later of the two ends.
        Otherwise:
            A gap exists, so add the span to the merged list as its
            own new entry.

    Return the merged list, one span per unbroken stretch of time.
```

One caveat in prose: the sort happens in place, so this function quietly
reorders the caller's original list as a side effect.

## What the translation reveals

The entire algorithm rests on a claim the source never makes: after
sorting, overlap is only ever possible with the most recent merged span.
That is why a single forward pass suffices, and it is the answer to the
question every newcomer asks ("how can it be safe to never look back?").
The Pseudo states the claim in the voiceover, surfaces the swallowed-whole
edge case hiding inside `max`, and reports the in-place-sort side effect
that the tidy-looking Python performs on its caller's data without asking.
