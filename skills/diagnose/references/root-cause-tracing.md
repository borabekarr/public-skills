# Root Cause Tracing

Bugs that surface deep in the call stack (a file created in the wrong directory, a database opened with the wrong path) tempt a fix at the point the error appears. That is a symptom fix: the same bad value keeps entering the system through whatever path produced it.

**Core principle:** trace backward through the call chain until you find the original trigger, then fix at the source.

## The process

1. **Observe the symptom.** State the error exactly (`git init failed in <project>/packages/core`).
2. **Find the immediate cause.** What line of code directly causes it?
3. **Ask what called it, one level at a time.** Follow the chain: function, caller, caller's caller, until a level clearly explains why the bad value exists.
4. **Find the original trigger.** The value usually turns out to be uninitialized, defaulted, or read before setup ran.

Example chain: `git init` runs in `process.cwd()` because `projectDir` was `''`, because a test read `context.tempDir` before `beforeEach` populated it, because the test-setup helper returned an empty string as its initial value. The fix belongs at the setup helper (make `tempDir` a getter that throws if accessed too early), not at the `git init` call site.

## When manual tracing stalls: add instrumentation

Log immediately **before** the dangerous operation, not after it fails, and include the call stack:

```typescript
async function gitInit(directory: string) {
  console.error('DEBUG git init:', { directory, cwd: process.cwd(), stack: new Error().stack });
  await execFileAsync('git', ['init'], { cwd: directory });
}
```

Use `console.error()` in tests, not a logger, because loggers are often suppressed in test runs. Grep the tagged output to find the triggering test and line.

## Finding which test causes pollution

When something shows up during a test run but you do not know which test causes it, bisect: run tests one at a time (or in halves) and stop at the first run that reproduces the artifact.

## Key principle

Fix at the source, not where the error appears. At each level ask "is this the source, or can I trace one more level up?" Once you reach the true source, add validation there, and consider layering additional checks at each intermediate point too (see `defense-in-depth.md`) so the same class of bug cannot re-enter through a different path.
