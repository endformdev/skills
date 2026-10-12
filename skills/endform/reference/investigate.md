# Find and fix failing, flaky, and slow tests

Both commands print JSON and use the active login or `ENDFORM_API_KEY`. Users in several organizations must pass `--organization-id <id>`.

Two datasets: `suite-runs` is one Endform run of a suite, `test-runs` is one test inside a suite run. Use `analytics` to find a trend, then `samples` to inspect the matching executions.

## analytics

`--metric` is `count`, `pass-rate`, or `average-duration`. `--group-by` is `outcome`, `branch`, `test`, or `project`. Suite runs cannot group by test or project. The response has an `interval`, a `unit`, and one series per group.

```bash
npx endform@latest analytics suite-runs --metric count --group-by outcome
npx endform@latest analytics suite-runs --metric pass-rate --group-by branch --period 2W
npx endform@latest analytics test-runs --metric pass-rate --group-by test --where 'branch = main'
npx endform@latest analytics test-runs --metric average-duration --group-by project --period 24H --ending-at 2026-08-11T12:00:00Z
```

## samples

`--limit` defaults to 10, maximum 50. When the response has `nextCursor`, pass it to `--cursor` for the next page. Test-run samples include every attempt in order with error message, error location, and OpenTelemetry trace id when available.

```bash
npx endform@latest samples test-runs --where 'testRun.outcome = fail'
npx endform@latest samples test-runs --where 'testRun.outcome = pass, testRun.testAttemptsCount > 1' --period 30D --limit 25
npx endform@latest samples test-runs --where 'testRun.durationSeconds > 30' --limit 10
npx endform@latest samples test-runs --where 'suiteRun.id = sr_123, testRun.outcome = fail'
```

## Filters

`--where` takes comma-separated `key operator value` clauses. Run `npx endform@latest samples --help` for the complete list of keys and operators. The keys map directly to response paths, so a filter on `testRun.outcome` reads the same field in the output. Suite run outcomes are `pass` or `fail`, test run and attempt outcomes can also be `skip`, durations are seconds.

`--period` is an integer followed by `H`, `D`, `W`, or `M`, default `7D`. `--ending-at` fixes the end of the window.

## Reading the output

Filter to failures and keep limits small. Summarize an error to its first meaningful frame instead of pasting the whole stack. When several tests fail with the same error message, report the shared cause once.

## Fix a failing or flaky test

1. Find the pattern with `analytics`: when the test started failing or retrying, on which branches, and in which projects. A test that passes only after retries (`testRun.testAttemptsCount > 1`) is flaky; one that fails on every run since a commit is broken.
2. Read the evidence with `samples`: the error message and location of each attempt. The same error on every attempt points at one cause; changing errors point at timing or shared state. Failures that appear only when many tests run at once point at load, see [Limit concurrency](#limit-concurrency).
3. Reproduce it in a live test, see [live-test.md](live-test.md): pause before the failing step and inspect the page.
4. Fix the test or the app. Wait for what the test needs instead of adding `page.waitForTimeout()` or `force: true`: use web-first assertions such as `await expect(page.getByRole("button", { name: "Pay" })).toBeEnabled()`, which retry until they pass, wait for a request with `page.waitForResponse()`, and find elements by role, label, or text instead of CSS selectors.
5. Check the fix on Endform. The repeats run in parallel, so this takes about as long as one run:

   ```bash
   npx endform@latest test tests/checkout.spec.ts:12 --repeat-each 20 --retries 0
   ```

## Quarantine a test

A quarantined test still runs on every suite run, but its failures no longer fail the suite. It runs once without retries, and its real outcomes stay in `samples` and `analytics`. Use it for a failing or flaky test while it is being fixed. Prefer it over `test.skip()`, `test.fixme()`, or deleting the test, which stop the results that show when the test recovers.

```bash
npx endform@latest quarantine list
npx endform@latest quarantine create --suite-id <suiteId> --test-id <testId> --branch main --reason "Flaky payment step" --ends-at 2026-11-01T00:00:00Z
npx endform@latest quarantine deactivate <ruleId>
```

`--suite-id` and `--test-id` are the exact `suite.id` and `test.id` from `samples test-runs`. A rule needs at least one of `--test-id`, `--project`, or `--branch`, and a scope flag left out matches every value, so a rule with only `--branch` quarantines every test in the suite on that branch. Scope rules to the test with `--test-id`, and add `--reason` and `--ends-at` so the rule does not outlive the fix. `update <ruleId>` replaces all of a rule's settings: a scope flag you leave out then widens the rule, and a left-out `--reason` or `--ends-at` is cleared. Check the rule with `quarantine list` and pass every setting you keep. The suite cannot change, and inactive rules cannot be updated. A rule changes your team's shared settings, so ask the user before creating, updating, or deactivating one.

- Rules match the test id, which changes when the test is renamed, moved, or its describe blocks change, so a renamed test is no longer quarantined.
- When several active rules match a test, the oldest applies. Check `quarantine list` before deactivating, since another matching rule keeps the test quarantined.
- A quarantined failure in a setup project still fails the suite, because the projects that depend on it cannot run.

To judge whether a quarantined test has recovered, read its runs since the fix. They ran without retries, so a pass there is a pass without retries. Deactivate the rule once the test passes across several runs; a passing suite alone shows nothing about it.

```bash
npx endform@latest samples test-runs --where 'test.id = <testId>' --period 7D --limit 20
```

## Limit concurrency

When tests fail only under load, such as timeouts, rate limits, or tests changing the same account at once, limit how many run at the same time with `concurrentTestLimits` in the Endform config instead of making the suite serial. Step 4 of [setup.md](setup.md) covers where the config lives.

```ts
export default defineEndformConfig({
  concurrentTestLimits: [
    { scope: "within-suite-run", label: "tag:@checkout", limit: 3 },
    { scope: "across-all-runs", label: "tag:@writes-to-staging", limit: 10 },
  ],
});
```

- `label` is `tag:<tag>`, `project:<name>`, or `lock:<name>`. Without a label, the limit covers every test. A test must fit within every limit it matches.
- `within-suite-run` counts one suite run. `across-all-runs` also counts the organization's other runs, such as parallel CI jobs against the same staging environment.
- Tests that share a Playwright 1.63 test lock never run at the same time. A `lock:<name>` label changes that lock's limit or applies it across all runs.
- Prefer the narrowest label: limiting one tag to 3 keeps the rest of the suite parallel.
