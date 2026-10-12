# Endform skills

[Endform](https://endform.dev) runs your existing Playwright suite in parallel, up to 10x faster, and helps you keep it healthy as it grows.

The `endform` skill lets your agent run your tests on Endform and find failing, flaky, and slow tests. It can then fix or quarantine them, or debug a test step by step in a live test.

## Install

### Claude Code

```bash
claude plugin marketplace add endformdev/skills
claude plugin install endform@endformdev
```

### Codex

```bash
codex plugin marketplace add endformdev/skills
codex plugin add endform@endformdev
```

### VS Code

In VS Code, enable `chat.plugins.enabled`, run **Chat: Install Plugin From Source** from the Command Palette, and enter `https://github.com/endformdev/skills`.

### Other agents

Install the skill with [skills.sh](https://skills.sh):

```bash
npx skills add endformdev/skills --skill endform
```

Pick your agents when prompted. Add `-g` to install it for all your projects.

## Use

Ask your agent, for example:

- "Set up Endform so our tests run on it in CI"
- "Which of our e2e tests are flaky?"
- "The checkout test only fails in CI, can you look into it?"
- "Quarantine the flaky search test so it stops blocking our PRs"

In Claude Code you can also call the skill directly with `/endform:endform`.

## MCP server

The Endform MCP server lets your agent query your test runs and debug failing tests on Endform. The Claude Code, Codex, and VS Code plugins add it for you.

To add it without the plugin, in Claude Code:

```bash
claude mcp add --transport stdio --scope local endform -- npx -y endform@latest mcp
```

For Cursor and other agents, see the [MCP server docs](https://endform.dev/docs/reference/mcp-server).

## Repository layout

```
skills/endform/SKILL.md      The skill: when to use Endform and which reference to read
skills/endform/reference/    The reference the skill reads for each task: setup, CI, investigating tests, live tests
plugin.json                  Plugin manifest for Codex, GitHub Copilot, VS Code, and Cursor
mcp.json                     MCP server for those agents
.agents/plugins/             Marketplace for Codex
.claude-plugin/              Plugin and marketplace for Claude Code
.mcp.json                    MCP server for Claude Code
assets/logo.png              Logo for plugin directories
```

## License

[MIT](LICENSE)
