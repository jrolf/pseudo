# Example 4: An AI Agent Loop (Design Before Code)

This example runs in the opposite direction. There is no source code here at
all: Pseudo is being used as a *design language*, sketching how an AI agent
should behave before a single line gets implemented. This is where the style
originally came from - comparing dozens of agent-loop architectures in one
shared notation - and it remains one of its strongest uses. A team can argue
about, approve, and version-control this document, then hand it to any
engineer (or any coding agent) in any language.

## The Pseudo design

```python
# REACT-STYLE TOOL-USING AGENT LOOP

Build the initial conversation:
    Add instructions requiring the model to produce, at each step, either:
        a thought,
        an action with its input,
        or a final answer.
    Add the descriptions of the available tools.
    Add the user's task.

For each iteration up to the maximum number of iterations:
    Ask the model for its next step.

    If the model produced a final answer:
        Store the final answer.
        Stop with a task-complete terminal reason.

    If the model produced an action:
        Parse the requested tool and its input.
        Validate the request against the allowed tools.
        If the request is not allowed:
            # Feeding the refusal back in keeps the model honest without
            # crashing the run - it can choose a different move next turn.
            Add a refusal notice to the conversation.
            Continue to the next iteration.
        Execute the tool.
        Add the tool's result to the conversation as an observation.
        Continue to the next iteration.

    Otherwise:
        # The model produced neither an action nor an answer, which means
        # the response was malformed. One nudge is cheap; looping on
        # malformed output forever is not.
        Add a reminder of the required format to the conversation.
        Continue to the next iteration.

If the maximum number of iterations was reached:
    Stop with a max-iterations terminal reason and the best partial result.
```

## What this buys you

Every load-bearing decision is now on the record and reviewable by the whole
team, technical or not: the loop is bounded, disallowed tools produce a
recoverable refusal rather than a crash, malformed output gets a nudge
instead of an infinite loop, and both ways the run can end are named. When
someone later asks "what happens if the model asks for a tool it doesn't
have?", the answer is a line you can point to - and if the implementation
disagrees with this document, one of them is wrong by definition.
