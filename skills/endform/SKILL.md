---
name: endform
description: 'Run, set up, and debug Playwright tests on Endform with the Endform CLI. Use whenever the user mentions Endform or the repository uses it (an `endform` dependency, `endform test` in a script or CI workflow, or an `endform.config.*` file), for requests like "run the e2e tests", "why did CI fail", "did my change break this test", "which tests are flaky", "why is the suite slow", or "quarantine this test". Covers first setup and CI, test analytics over time (pass rates, flakiness, durations), failure investigation with replays of recorded test attempts, quarantine, and live debugging inside a running test.'
metadata:
  author: endform
---

# Endform

Endform runs Playwright suites in parallel on isolated machines. The CLI is `npx endform@latest`. It takes Playwright's test selection and run flags, such as file filters, `--grep`, `--project`, `--repeat-each`, and `--retries`, but not Playwright's --workers, --headed, --ui, or --list.

## References

Read the one for the task:

- Set up Endform, first run, local server, config files: [reference/setup.md](reference/setup.md)
- Run Endform in CI, such as GitHub Actions or GitLab CI: steps 2 to 4 of [reference/setup.md](reference/setup.md), then [reference/ci.md](reference/ci.md)
- Test the Vercel preview of each pull request: steps 2 to 4 of [reference/setup.md](reference/setup.md), then [reference/vercel.md](reference/vercel.md)
- Test the Netlify Deploy Preview of each pull request: steps 2 to 4 of [reference/setup.md](reference/setup.md), then [reference/netlify.md](reference/netlify.md)
- Find and fix failing, flaky, and slow tests, see what a test attempt did and whether a change caused a failure, replays, quarantine a failing or flaky test, limit concurrency for tests that fail under load, pass rates, past runs: [reference/investigate.md](reference/investigate.md)
- Run code inside a live test to inspect a failing step, find locators, or try new steps: [reference/live-test.md](reference/live-test.md)

If the reference does not cover the case, say so and point the user to https://endform.dev/docs.

## Login and organization

Run the command the task needs and react to its output:

- **"No Endform credentials found"** or **"Unauthorized"**: run `npx endform@latest login`, show the URL it prints and ask the user to finish browser authorization. If it says login runs in two steps, then run `npx endform@latest login --resume-pending` and keep it running until it completes. Resume the task once login succeeds. Do not work around a missing login.
- **"Organization selection is required"**: run `npx endform@latest whoami` to list the organizations, ask the user which one, then pass `--organization-id <id>` on every `analytics`, `samples`, `test`, `live-test`, `quarantine`, `connect`, and `api-key` command.

Commands in the references show `--organization-id <organization-id>`. Fill in the chosen id, or leave the flag out until the CLI asks for an organization.

## Rules

- If you cannot run commands, use the Endform MCP tools when they are connected, otherwise say you need a shell. The remote MCP server reads run data, inspects test attempts, and manages quarantine, but cannot run tests or live tests. Before mixing CLI and MCP results, check with `npx endform@latest whoami` that they use the same organization.- Make the repository changes the task needs and summarize what changed. Ask before storing secrets or changing settings outside the repository, and commit or open a pull request only when asked.
- Never write an API key or other secret into the repository. GitHub Actions authenticates through the Endform GitHub App or the `ENDFORM_API_KEY` secret. Other CI systems use the secret.
- Let `npx endform@latest test` finish. Stopping it, or a tool timeout that kills it, cancels the suite run, so give it a timeout longer than the suite takes.
