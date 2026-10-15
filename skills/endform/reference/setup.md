# Set up Endform in a repository

Endform is a drop-in replacement for the Playwright test runner: `npx endform@latest test` replaces `playwright test` and runs every test on its own machine. The tests stay as they are; setup adds a few settings to `playwright.config.ts`.

## Steps

1. Run `npx endform@latest login` so the CLI can run tests on Endform for this account. It prints a URL to approve and tells you which command to run next, so show the URL and follow its output until the login completes.
2. Find the Playwright project:
   - If this repository already has one, use it and leave its tests as they are.
   - If not, add `@playwright/test` as a dev dependency, create `playwright.config.ts` with `testDir` set to `./tests`, `forbidOnly: !!process.env.CI`, and a single chromium project, and add `test-results/` and `playwright-report/` to `.gitignore`. Leave out `workers` and `reporter` settings; Endform handles those. Then ask which three or four pages or flows matter most, and write one small test for each with stable locators such as roles and labels. Then check they pass with `npx endform@latest test --organization-id <organization-id>`.
   - In both cases, do not install browsers; Endform provides them. Run Endform from the directory that contains `playwright.config.ts`, and in CI set the job's working directory there.
3. Merge these settings into `playwright.config.ts` and keep everything else:
   ```ts
   export default defineConfig({
     fullyParallel: true,
     retries: 2,
     use: {
       baseURL: process.env.BASE_URL || "<the URL the suite uses today>",
       trace: "retain-on-failure",
     },
   });
   ```
   - `fullyParallel: true` lets Endform run every test in parallel. Endform only reads `fullyParallel` per project, so `test.describe.configure({ mode: "serial" })` and `test.describe.serial` do not keep tests together. If tests in a file share `beforeAll` state or worker fixtures, move those files to a project with `fullyParallel: false` and exclude them from the other projects with `testIgnore`. Each file in that project then runs on one machine, in order. A retry reruns only the failed tests, so setup that one test leaves for the next belongs in `beforeAll` or a fixture.
   - Endform retries only as often as `retries` allows. Keep the value if the config already sets one.
   - If the config starts a dev server through `webServer`, leave `webServer` and `baseURL` as they are, unless the tests will run against Vercel previews. Otherwise read `baseURL` from `BASE_URL`, falling back to the URL the suite uses today, and if none is in the repo, ask whether to add a `webServer` that starts the local dev server or to use a deployed URL such as staging or a Vercel preview.
   - `trace: "retain-on-failure"` gives failed tests a trace in the Endform dashboard.
4. Some suites need an Endform config for one of these reasons. Endform transfers imported files and the environment variables tests reference automatically, so do not add one up front. Add the narrowest setting that fixes an observed failure. `npx endform@latest test --debug --organization-id <organization-id>` shows which files and environment variables are sent to the runners:
   - a test reads files without importing them: `additionalFiles` (globs).
   - a test needs environment variables it does not reference through `process.env`: `environmentVariables` (regexes).
   - the tests target a localhost server that is not started through `webServer`: `proxyNetworkHosts: ["<loopback>"]`, which sends that traffic through the local machine.
   - tests fail or time out under load because the target cannot handle many tests at once, such as a local server, a staging database, or a payment sandbox: `concurrentTestLimits`, for example `[{ scope: "within-suite-run", limit: 40 }]`, or with `label: "tag:@checkout"` to limit only the tests that hit the bottleneck.
   - To add a setting, use the repository's existing Endform config, in this directory or a parent up to the repository root.
   - Convert a legacy `endform.jsonc` or `endform.json` with `npx endform@latest migrate-config` first, since a new `endform.config.ts` next to it silently replaces it.
   - If there is none, add `endform` as a dev dependency, create `endform.config.ts` next to `playwright.config.ts` using `defineEndformConfig` from `"endform"`, and read https://endform.dev/docs/_llms-txt/run-and-configure-endform.txt for its options.
5. When the setup is done, run `npx endform@latest test --organization-id <organization-id>`. It takes Playwright's test selection and run flags, such as file filters, `--grep`, `--project`, and `--retries`, but not Playwright's --workers, --headed, or --ui. Stopping the command cancels the run, so give it a timeout longer than the suite takes. Fix configuration problems and run it again until the run completes. If tests fail for reasons unrelated to the Endform setup, report them instead of changing the tests. Then give the dashboard link it prints.

## Requirements

- Playwright in TypeScript or JavaScript, one of the latest three minor versions. Node 22 or newer locally.
- Each test must set up what it needs itself, and not depend on an earlier test to do it. With `fullyParallel: true`, every test runs on a separate machine. With `fullyParallel: false`, every file runs on a separate machine, and its tests run in order. When a test fails, the retry runs only that test, not the tests before it. To share setup between tests in a file, use `beforeAll` or a worker fixture.
- Each test is limited to 4 minutes, each file to 15 minutes when `fullyParallel` is false.
- `.only()` and `.fail()` are not supported.

## CI

To run Endform in CI as well, continue with [ci.md](ci.md).
