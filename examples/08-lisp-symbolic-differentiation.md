# Example 8: Symbolic Differentiation in Lisp

The classic. Lisp treats formulas as nested lists, so a program can do
calculus on formulas the way other programs do arithmetic on numbers. It is
one of the most beautiful ideas in computing, wrapped in the parentheses
that scare everyone away.

## The original

```lisp
(defun deriv (expr var)
  (cond
    ((numberp expr) 0)
    ((symbolp expr) (if (eq expr var) 1 0))
    ((eq (first expr) '+)
     (list '+ (deriv (second expr) var)
              (deriv (third expr) var)))
    ((eq (first expr) '*)
     (list '+
           (list '* (second expr) (deriv (third expr) var))
           (list '* (deriv (second expr) var) (third expr))))
    (t (error "Unknown expression: ~a" expr))))
```

## The Pseudo translation

```python
# SYMBOLIC DIFFERENTIATION: TAKE THE DERIVATIVE OF A FORMULA, AS DATA

# The formula arrives as a nested list, like "the sum of x and the
# product of 3 and x". The answer is another formula, not a number.
# The function looks at the formula's shape, applies the matching rule
# from calculus, and calls itself on the smaller pieces inside.

Define "deriv", given [expr, var]:

    If the formula is a plain number:
        The derivative is zero.

    If the formula is a lone variable:
        If it is the very variable we are differentiating by:
            The derivative is one.
        Otherwise:
            The derivative is zero.

    If the formula is a sum of two parts:
        # The sum rule: differentiate each part and add the results.
        Build a new sum from:
            the derivative of the first part,
            and the derivative of the second part.

    If the formula is a product of two parts:
        # The product rule: each factor takes a turn being differentiated
        # while the other stands still.
        Build a new sum from:
            the first part times the derivative of the second,
            and the derivative of the first part times the second.

    Otherwise:
        Stop with an unknown-expression error naming the formula.
```

## What the translation reveals

Strip away the parentheses and what remains is a calculus lesson: four
shape-matching rules, two of them recursive. The Pseudo makes the deep idea
explicit in the opening voiceover - the answer is another formula, not a
number - which is the fact that makes symbolic differentiation "symbolic"
and which the Lisp never states. It also surfaces the honest limitation
that the code only handles two-part sums and products, something easy to
miss among the `second`s and `third`s.
