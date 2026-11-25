# Endform for Claude

[Endform](https://endform.dev) runs your existing Playwright suite in parallel, up to 10x faster, and helps you keep it healthy as it grows.

This plugin adds the `endform` skill and the Endform MCP server to Claude Code. The skill lets Claude run your tests on Endform and find failing, flaky, and slow tests. It can then fix or quarantine them, or debug a test step by step in a live test.

## Use

Ask Claude, for example:

- "Set up Endform so our tests run on it in CI"
- "Which of our e2e tests are flaky?"
- "The checkout test only fails in CI, can you look into it?"
- "Quarantine the flaky search test so it stops blocking our PRs"

You can also call the skill directly with `/endform`.

## MCP server

The plugin adds the remote Endform MCP server at `https://endform.dev/mcp`. Run `/mcp` in Claude Code and sign in with your Endform account. You choose all your organizations or a single one when you connect.

Endform also has a local MCP server that can debug a test live in your checkout. See the [MCP server docs](https://endform.dev/docs/reference/mcp-server) to add it.

## Data and privacy

The skill runs the Endform CLI from npm. It sends the Playwright tests you select, what they need to run, and their results to Endform. The remote MCP server only uses data already in your Endform account. See our [privacy policy](https://endform.dev/privacy) for more details.

## License

[MIT](LICENSE)
