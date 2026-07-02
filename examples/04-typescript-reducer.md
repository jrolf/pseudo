# Example 4: A TypeScript Discriminated-Union Reducer

State-management code is where TypeScript shines and where readers drown.
This reducer manages the lifecycle of a data fetch, and its most important
line is the one that looks the most like boilerplate.

## The original

```typescript
type State =
  | { status: "idle" }
  | { status: "loading"; requestId: string }
  | { status: "success"; data: User[]; fetchedAt: number }
  | { status: "error"; message: string };

type Action =
  | { type: "FETCH"; requestId: string }
  | { type: "RESOLVE"; requestId: string; data: User[] }
  | { type: "REJECT"; requestId: string; message: string }
  | { type: "RESET" };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case "FETCH":
      return { status: "loading", requestId: action.requestId };
    case "RESOLVE":
      if (state.status !== "loading" || state.requestId !== action.requestId) {
        return state;
      }
      return { status: "success", data: action.data, fetchedAt: Date.now() };
    case "REJECT":
      if (state.status !== "loading" || state.requestId !== action.requestId) {
        return state;
      }
      return { status: "error", message: action.message };
    case "RESET":
      return { status: "idle" };
  }
}
```

## The Pseudo translation

```python
# FETCH-STATE REDUCER: ONE STATE AT A TIME, STALE RESULTS IGNORED

The state is always exactly one of:
    idle,
    loading a specific numbered request,
    success with the data and its arrival time,
    or error with a message.

When an event arrives, produce the next state from the current one:

    If the event is "start fetching":
        Become loading, and remember which request this is.

    If the event is "a fetch succeeded":
        # Responses can arrive late and out of order. Only the response
        # matching the request we are currently waiting for may win;
        # anything else is a ghost from an abandoned fetch.
        If we are not loading, or the response is for a different request:
            Keep the current state unchanged.
        Otherwise:
            Become success, holding the data and stamping the arrival time.

    If the event is "a fetch failed":
        If we are not loading, or the failure is for a different request:
            Keep the current state unchanged.
        Otherwise:
            Become error, holding the failure message.

    If the event is "reset":
        Become idle, whatever was happening before.
```

## What the translation reveals

The repeated `requestId` comparison is not boilerplate - it is a race
condition defense, and the whole reason this reducer is safe to use with
fast-clicking users and slow networks. The Pseudo voiceover names the ghost
it is guarding against. The translation also states the discriminated
union's promise in one sentence ("the state is always exactly one of"),
which is the fact every reader of the TypeScript has to reconstruct from
four type declarations.
