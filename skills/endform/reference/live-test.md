# Run code inside a live test

A live test pauses one test on Endform at a chosen line and keeps its browser running, so you can run JavaScript in the test's scope again and again, with fixtures such as `page`, and read a fresh ARIA snapshot of the page after each step. Use it to see why a step fails only in CI, to find locators that work on the real page, or to try the next steps of a new test before writing them. Startup can take a few minutes. Sessions end on their own after about 12 minutes. Stop the session when you are done.

```bash
npx endform@latest live-test start tests/checkout.spec.ts:47 --line 72
npx endform@latest live-test run <sessionId> --code 'await page.getByRole("button", { name: "Pay" }).click(); console.log(await page.title());'
npx endform@latest live-test run <sessionId> - < snippet.js
npx endform@latest live-test status <sessionId>
npx endform@latest live-test stop <sessionId>
```

- `start` takes Playwright-style test file filters, optionally with a declaration line, plus `--grep`, `--grep-invert`, `--project`, and `--config`. The selection must resolve to one test. `--line` is the 1-based source line to pause before and defaults to the first statement of the test body. It prints the session id, the checkpoint location, a `results:` directory for the session, and the path of the initial page snapshot.
- `run` takes the code with `--code` or from stdin when the last argument is `-`. The code can be several statements and use `await`. It prints the code's stdout and stderr and exits non-zero when the code threw. The session stays paused, so you can call it again. The page snapshot after each run and any new artifacts, such as screenshots, are saved in the session's results directory.
- `status` prints JSON with `status`: `starting`, `paused`, `running`, `ended`, or `stale`. While the session is paused or running it also has `snapshotPath`, the latest page snapshot; read that file to see the page.
