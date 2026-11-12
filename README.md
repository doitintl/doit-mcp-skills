# DoiT MCP Skills

AI agent skills that bring [DoiT Cloud Intelligence](https://www.doit.com/) into your favorite AI clients — Claude Code, ChatGPT, Cursor, Codex, and Gemini CLI.

Query cloud costs, investigate anomalies, generate reports, and produce API integration code across AWS, GCP, Azure, Snowflake, Datadog, and more — all through natural language.

## Skills


| Skill                                                                        | Description                                                                              |
| ---------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| **[doit-mcp-setup](skills/doit-mcp-setup/)**                                 | Configure DoiT MCP in any supported AI client with correct credentials and OAuth         |
| **[doit-mcp-reporting](skills/doit-mcp-reporting/)**                         | Generate cost intelligence reports and run ad-hoc Cloud Analytics queries                |
| **[doit-mcp-anomaly-investigation](skills/doit-mcp-anomaly-investigation/)** | Root-cause cost anomalies by correlating DoiT alerts with cloud-specific inspection      |
| **[doit-mcp-api](skills/doit-mcp-api/)**                                     | Generate production-ready API call code (TypeScript, Python, Go, Bash) for DoiT Platform |


## Quick Start

### Claude Code (Plugin)

Install as a plugin for the full experience — skills, MCP server, and tools in one package:

```bash
claude plugin marketplace add https://github.com/doitintl/doit-mcp-skills
claude plugin install doit-mcp@doit-mcp-skills
```

Then run `/mcp` in Claude Code and authorize the DoiT server.

### Claude Code channel (optional)

The plugin can also act as a Claude Code [channel](https://code.claude.com/docs/en/channels) (research preview): while a session is open, it tells Claude about new cost anomalies and cloud provider incidents, and Claude can look into them with the DoiT tools.

1. Run `/plugin configure doit-mcp@doit-mcp-skills` and set **DoiT API key (channel only)** to a personal API token from the DoiT console (Profile > API tokens). The tools keep signing in through your browser; only the channel uses the key.
2. Start Claude Code with the channel:

   ```bash
   claude --channels plugin:doit-mcp@doit-mcp-skills
   ```

Plain `--channels` registers the plugin only where your organization allows it (`channelsEnabled` and `allowedChannelPlugins` in managed settings). Use the plugin id that `/plugin` shows if you installed it from another marketplace.

What the channel runs and sends: `npx -y @doitintl/doit-mcp-server@3.1.0` (the open-source [DoiT MCP server](https://github.com/doitintl/doit-mcp-server), pinned to an exact version) in events-only mode. It serves no tools. With an API key set it polls two read-only DoiT API endpoints (cost anomalies and active cloud incidents) every 5 minutes by default, in every Claude Code session the plugin is loaded into, and sends requests only to the DoiT API. Events contain a fixed sentence and identifiers only. Without an API key it does nothing.

- Support: <https://support.doit.com>
- Privacy policy: <https://www.doit.com/privacy>

### Claude Code (Skills Only)

If you only want the skills without the plugin wrapper:

```bash
claude install-skill https://github.com/doitintl/doit-mcp-skills
```

Then optionally run the setup skill for connection steps:

```
/doit-mcp-setup
```

### Other AI Clients (Cursor, Codex, Gemini CLI, ChatGPT)

The skills in this repo are plain Markdown files — any AI that can read a file can use them. To set up DoiT MCP in a non-Claude client:

1. **Point your agent at the skill file** — paste the contents of the relevant `SKILL.md` into your agent's context, or reference the file directly if your client supports file attachments.
2. **Run setup** — tell your agent: *"Follow the instructions in this skill to configure DoiT MCP for [your client]."*
3. The agent will show the config snippet to add and guide you through any OAuth steps.

Alternatively, ask your agent to run the **doit-mcp-setup** skill for your client:

- **Cursor** — per-project `.cursor/mcp.json` configuration
- **ChatGPT** — manual setup via Actions UI (requires developer mode + API key)
- **Codex** — `~/.codex/config.toml` configuration
- **Gemini CLI** — `~/.gemini/settings.json` configuration

### DCI CLI (Optional)

The setup skill can also install the **DoiT Cloud Intelligence CLI** — a terminal companion that talks to the same DoiT API as the MCP server. Run the setup skill and choose the CLI option, or install it directly:

| Platform | Command |
| -------- | ------- |
| macOS | `brew install doitintl/dci-cli/dci` |
| Windows (WinGet) | `winget install DoiT.dci` |
| Windows (Scoop) | `scoop bucket add doitintl https://github.com/doitintl/dci-cli && scoop install dci` |
| Linux (.deb / .rpm) | Download from [Releases](https://github.com/doitintl/dci-cli/releases/latest) |

After install, run `dci login` to authenticate and `dci list-reports` to verify.

## What You Can Do

**Cost Reporting**

> "Show me my top 5 cost drivers for the last 30 days"
> "Break down GCP costs by project for Q1"

**Anomaly Investigation**

> "Show me recent cost anomalies"
> "Investigate the EC2 cost spike from last Tuesday"

**API Code Generation**

> "Generate TypeScript code to query costs by service for the last month"
> "Show me a Python script that lists all my cloud reports"

## Architecture

```
doit-mcp-skills/
├── .claude-plugin/          # Claude Code plugin metadata
│   ├── plugin.json          # Plugin manifest (skills + MCP config)
│   └── marketplace.json     # Marketplace manifest for plugin install
├── .mcp.json                # MCP server configuration
└── skills/
    ├── doit-mcp-setup/      # Client setup & configuration
    ├── doit-mcp-reporting/  # Report generation & queries
    ├── doit-mcp-anomaly-investigation/
    └── doit-mcp-api/        # API code generation
```

Each skill contains:

- `SKILL.md` — skill instructions and workflow
- `agents/openai.yaml` — agent configuration and tool dependencies
- `references/` — detailed reference docs (dimensions, metrics, field mappings)
- `templates/` — config snippets or output templates

## How It Works

All skills connect to the DoiT MCP server at `https://mcp.doit.com/sse` — no local server installation required. Authentication is handled via OAuth (automated clients) or API key (manual clients like ChatGPT).

**Supported platforms**: AWS, GCP, Azure, Snowflake, Datadog, Databricks, OpenAI

## Prerequisites

- A [DoiT](https://www.doit.com/) account with **Billing Profiles Admin** permission
- An AI client that supports MCP (Claude Code, Cursor, ChatGPT, Codex, or Gemini CLI)

## License

[MIT](LICENSE)