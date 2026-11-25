# Test Vercel previews with Endform

On GitHub Actions, Endform can run the tests against the Vercel preview of each pull request. The `endformdev/actions/run-with-vercel-deployment` action waits for the preview and passes its URL to the tests.

1. The tests run against the Vercel preview, so `baseURL` must read `process.env.BASE_URL`, even when `webServer` starts a dev server. Then start `webServer` only when `BASE_URL` is not set, so local runs keep working.
2. Add the `endformdev/actions/run-with-vercel-deployment` action with `set-url-env-var: BASE_URL` before `npx endform@latest test`. Endform reports on the Vercel check only when the workflow uses this action.
   - If a workflow already runs `playwright test`, directly or through a package script such as `pnpm test:e2e`, add the action before it and replace that command with `npx endform@latest test`, leaving the package script as it is. Keep the other steps, and drop what Endform replaces: browser installs and caches, `--shard` matrix jobs, and the job that merges their blob reports. Otherwise create `.github/workflows/endform-e2e.yml` with this shape, using the repository's package manager:
     ```yaml
     name: End-to-end tests
     on:
       pull_request:
     permissions:
       contents: read
       id-token: write
     jobs:
       e2e:
         runs-on: ubuntu-latest
         steps:
           - uses: actions/checkout@v7
           - uses: actions/setup-node@v7
           - run: npm ci
           - uses: endformdev/actions/run-with-vercel-deployment@main
             with:
               project-name: <vercel-project-name>
               set-url-env-var: BASE_URL
               set-vercel-bypass-env-var: VERCEL_BYPASS_TOKEN
           - run: npx endform@latest test
             working-directory: .
     ```
   - The workflow must run on pull requests, since previews exist only for them.
   - Take the Vercel project name from `.vercel/project.json`, or ask for it.
   - The action authenticates only through the Endform GitHub App with OIDC: add `id-token: write` to the workflow's permissions and keep `contents: read`, since listing any permission removes the ones not listed. Do not add an API key.
3. Make sure the Endform GitHub App and Vercel are connected with `npx endform@latest connect github --organization-id <organization-id>` and `npx endform@latest connect vercel --organization-id <organization-id>`. Each says so when the integration is already connected. Otherwise it prints a URL where the install is finished in the browser: show the URL. If it says connecting runs in two steps, then run the same command with `--wait` and keep it running until it is connected. These commands need a login: if one says you are not logged in, run `npx endform@latest login`, show the URL it prints for approval in the browser. If it says login runs in two steps, then run `npx endform@latest login --resume-pending` and keep it running until the login completes, then run the command again.
4. If the tests read other environment variables, such as app credentials, list them and ask for them to be added as GitHub Actions secrets. Reference them in the workflow, never write their values.
