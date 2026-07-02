# Example 9: Reversing a Linked List in C

Pointer manipulation is where new programmers' intuition goes to die. Four
lines of C, three pointers, and one wrong move loses half the list forever.

## The original

```c
struct node {
    int value;
    struct node *next;
};

struct node *reverse(struct node *head) {
    struct node *prev = NULL;
    while (head != NULL) {
        struct node *next = head->next;
        head->next = prev;
        prev = head;
        head = next;
    }
    return prev;
}
```

## The Pseudo translation

```python
# REVERSE A LINKED LIST IN PLACE, FLIPPING ONE LINK AT A TIME

Keep track of three positions while walking the chain:
    the part already reversed, which starts out empty,
    the link currently being flipped,
    and the untouched remainder of the chain.

While there is still a link to flip:
    # Save the way forward FIRST. Flipping the current link is exactly
    # the move that destroys the only path to the rest of the chain,
    # so the order of these steps is everything.
    Remember where the untouched remainder begins.
    Point the current link backward, at the already-reversed part.
    The reversed part now includes the current link.
    Step forward into the untouched remainder.

When the walk ends:
    The "reversed part" now holds the entire chain, back to front.
    Return it as the new head of the list.
```

## What the translation reveals

The C gives you four assignments with near-identical names (`prev`, `head`,
`next`, `head->next`) and no hint that their order is load-bearing. The
Pseudo names the invariant the loop maintains - reversed part, current link,
untouched remainder - and spends its voiceover on the one thing that
actually goes wrong in practice: flip before you save the way forward and
the rest of the list is gone. Once you can say the invariant out loud, the
code stops being a puzzle.
