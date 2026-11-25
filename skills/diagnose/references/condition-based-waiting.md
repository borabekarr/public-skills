# Condition-Based Waiting

Flaky tests often guess at timing with an arbitrary delay. The delay passes on a fast machine and fails under load or in CI, because it is not tied to the thing the test actually depends on.

**Core principle:** wait for the condition you care about, not a guess about how long it takes to become true.

```typescript
// wrong: guessing at timing
await new Promise(r => setTimeout(r, 50));
const result = getResult();

// right: waiting for the condition
await waitFor(() => getResult() !== undefined);
const result = getResult();
```

## Generic polling helper

```typescript
async function waitFor<T>(
  condition: () => T | undefined | null | false,
  description: string,
  timeoutMs = 5000
): Promise<T> {
  const startTime = Date.now();
  while (true) {
    const result = condition();
    if (result) return result;
    if (Date.now() - startTime > timeoutMs) {
      throw new Error(`Timeout waiting for ${description} after ${timeoutMs}ms`);
    }
    await new Promise(r => setTimeout(r, 10)); // poll every 10ms
  }
}
```

Common shapes: wait for an event (`waitFor(() => events.find(e => e.type === 'DONE'))`), a state (`waitFor(() => machine.state === 'ready')`), a count (`waitFor(() => items.length >= 5)`), or a file (`waitFor(() => fs.existsSync(path))`).

## Mistakes to avoid

- **Polling too fast** (every 1ms) burns CPU for no benefit. 10ms is plenty.
- **No timeout** turns a hang into an infinite loop instead of a clear failure.
- **Stale data** happens when the condition closes over a value captured before the loop started. Call the getter fresh inside the loop.
- **A condition that is already true before the action** passes immediately, so the test races ahead of the change it meant to wait for. Pick a condition that can only hold after the action took effect, such as a value that did not exist before. If no such value exists, capture the current value first and wait until it differs.

## When an arbitrary timeout is still correct

If you are testing actual timing behavior (debounce, throttle, a fixed tick interval), a raw delay is legitimate, but only after first waiting for the triggering condition, and only with a comment stating the known interval it is based on:

```typescript
await waitForEvent(manager, 'TOOL_STARTED'); // first: wait for the condition
await new Promise(r => setTimeout(r, 200));   // then: 2 ticks at 100ms, documented
```

Do not use this pattern when the right timing is unknown. If you cannot state why the number is correct, it is a guess and belongs back on the wrong side of the pattern above.
