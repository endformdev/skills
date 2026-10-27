# Test Netlify Deploy Previews with Endform

The Endform Netlify extension runs the tests on Endform after Netlify publishes each Deploy Preview, so a pull request is what starts a run. Results show in the Netlify deploy log, in Endform and, when the repository has the Endform GitHub App, as an "Endform" check on the pull request.

1. The extension sets `BASE_URL` to the preview's URL, so `baseURL` must read `process.env.BASE_URL`, even when `webServer` starts a dev server. Then start `webServer` only when `BASE_URL` is not set, so local runs keep working.
2. The extension runs the tests from the Netlify project's base directory with the dependencies the Netlify build installs, so `@playwright/test` must be in that directory's `package.json`. If the Playwright config is not in the base directory or one of its parents, tell me to set **Playwright config path** in the project's Endform configuration in Netlify. The extension starts the tests, so do not add a CI workflow for Endform.
3. If the tests read other environment variables, such as app credentials, list them and ask for them to be added to the Netlify project's environment variables for Deploy Previews. Never write their values.
