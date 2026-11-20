# Endform skills

[Endform](https://endform.dev) runs your existing Playwright suite in parallel, up to 10x faster, and helps you keep it healthy as it grows.

The `endform` skill lets your agent run your tests on Endform and find failing, flaky, and slow tests. It can then fix or quarantine them, or debug a test step by step in a live test.

## Install

### Claude Code

```bash
claude plugin marketplace add endformdev/skills
claude plugin install endform@endformdev
```

Claude Code doesn't update plugins from this marketplace automatically. To turn that on, run `/plugin`, go to **Marketplaces**, select `endformdev`, and select **Enable auto-update**. To update once, run `claude plugin update endform@endformdev`.

### Codex

```bash
codex plugin marketplace add endformdev/skills
codex plugin add endform@endformdev
```

### Antigravity

```bash
git clone https://github.com/endformdev/skills endform-skills
agy plugin install ./endform-skills
```

Then run `/plugin`, expand **endform** under **Installed**, and choose **Sign in**.

### VS Code

In VS Code, enable `chat.plugins.enabled`, run **Chat: Install Plugin From Source** from the Command Palette, and enter `https://github.com/endformdev/skills`.

### Other agents

Install the skill with [skills.sh](https://skills.sh):

```bash
npx skills add endformdev/skills --skill endform
```

Pick your agents when prompted, or pass them with `-a`, for example `-a claude-code`. Add `-g` to install it for all your projects.

## Use

Ask your agent, for example:

- "Set up Endform so our tests run on it in CI"
- "Which of our e2e tests are flaky?"
- "The checkout test only fails in CI, can you look into it?"
- "Quarantine the flaky search test so it stops blocking our PRs"

You can also call the skill directly: `/endform` in Claude Code and Cursor, `/endform:endform` in VS Code, and `$endform` in Codex.

## MCP server

The Endform MCP server lets your agent query your test runs and debug failing tests on Endform. It comes in two versions:

|                     | Local                                           | Remote                                                  |
| ------------------- | ----------------------------------------------- | ------------------------------------------------------- |
| Runs                | On your machine, through `npx`                  | On Endform, at `https://endform.dev/mcp`                |
| Best for            | Coding agents working in your repository        | Agents that can't run local commands, such as claude.ai |
| Sign-in             | `npx endform@latest login` or `ENDFORM_API_KEY` | Your Endform account, in the browser                    |
| Organizations       | Any you can access                              | All of yours, or one you choose when you connect        |
| Live test debugging | Yes                                             | No, it needs your local checkout                        |

### Local

The Codex and VS Code plugins add it for you. To add it in Claude Code:

```bash
claude mcp add --transport stdio --scope local endform -- npx -y endform@latest mcp
```

For Cursor and other agents, see the [MCP server docs](https://endform.dev/docs/reference/mcp-server).

### Remote

Add `https://endform.dev/mcp` as a remote MCP server in your agent and sign in with your Endform account. You choose all your organizations or a single one when you connect.

The Claude Code and Antigravity plugins add it for you: run `/mcp` in Claude Code or `/plugin` in Antigravity to sign in. To add it without the plugin, in Claude Code, add it and then run `/mcp`:

```bash
claude mcp add --transport http endform-remote https://endform.dev/mcp
```

In Codex:

```bash
codex mcp add endform-remote --url https://endform.dev/mcp
codex mcp login endform-remote
```

In VS Code, add this to `.vscode/mcp.json` and sign in when prompted:

```json
{
  "servers": {
    "endform-remote": {
      "type": "http",
      "url": "https://endform.dev/mcp"
    }
  }
}
```

### Using both

If you use the local server in one agent and the remote server in another, there's nothing else to set up: each agent has its own connection.

You can also add both to one agent by giving them different names, as the commands above do. It's usually better to pick one: local if the agent can run commands, remote if it can't.

## Data and privacy

Your agent runs the Endform CLI from npm. It sends the Playwright tests you select, what they need to run, and their results to Endform. The remote MCP server only uses data already in your Endform account. See our [privacy policy](https://endform.dev/privacy) for more details.

## Repository layout

```
skills/endform/SKILL.md      The skill: when to use Endform and which reference to read
skills/endform/reference/    The reference the skill reads for each task: setup, CI, investigating tests, live tests
plugin.json                  Plugin manifest for Codex, GitHub Copilot, VS Code, and Cursor
mcp.json                     MCP server for those agents
.agents/plugins/             Marketplace for Codex
.claude-plugin/              Plugin and marketplace for Claude Code
.mcp.json                    MCP server for Claude Code
.antigravity-plugin/         Plugin manifest for Antigravity
mcp_config.json              MCP server for Antigravity
```

## License

[MIT](LICENSE)
