# Defense-in-Depth Validation

One validation point feels sufficient once a bug is fixed, but a single check can be bypassed by a different code path, a later refactor, or a test mock that skips the layer where the check lives. Validating at every layer the data passes through makes the bug structurally impossible instead of merely patched.

## The layers

**Entry point.** Reject obviously invalid input at the API boundary:

```typescript
function createProject(name: string, workingDirectory: string) {
  if (!workingDirectory || workingDirectory.trim() === '') {
    throw new Error('workingDirectory cannot be empty');
  }
  if (!existsSync(workingDirectory)) {
    throw new Error(`workingDirectory does not exist: ${workingDirectory}`);
  }
}
```

**Business logic.** Re-check that the data still makes sense at the point it is actually used, downstream of the entry point. A different caller may have reached this code without going through the entry check.

**Environment guard.** Prevent dangerous operations in specific contexts, for example refusing a destructive filesystem operation outside a known-safe temp directory during tests:

```typescript
if (process.env.NODE_ENV === 'test' && !resolve(directory).startsWith(resolve(tmpdir()))) {
  throw new Error(`Refusing git init outside temp dir during tests: ${directory}`);
}
```

**Debug instrumentation.** When the first three layers all miss a case, a log line before the dangerous operation captures enough context (directory, cwd, stack) to diagnose the gap after the fact.

## Applying the pattern

1. Trace the data flow: where does the bad value originate, and where is it used? (See `root-cause-tracing.md`.)
2. Map every checkpoint the value passes through between those two points.
3. Add validation at each layer, not just the one closest to the fix.
4. Test each layer independently: deliberately try to bypass layer 1 and confirm layer 2 still catches it.

## Why all layers matter

In practice each layer catches cases the others miss: different code paths bypass entry validation, mocks bypass business-logic checks, platform-specific edge cases need the environment guard, and debug logging is what identifies a structural misuse that no validation anticipated. Stopping at one validation point leaves exactly the gap the others would have caught.
