---
name: doit-mcp-setup
description: Set up the DoiT MCP server in supported clients and verify the connection flow. Use when the agent needs to configure ChatGPT, Claude, Claude Code, Codex, Cursor, or Gemini CLI for DoiT MCP access, produce the correct connection snippet, explain required credentials, or troubleshoot why a DoiT MCP client is not connecting.
---

# DoiT MCP Setup

Guide the user through connecting a client to the DoiT MCP server. For clients configured through a file, show the exact snippet and where it goes, and let the user add it. For clients that require UI steps, guide the user through each step.

Do not read or edit the user's client configuration files, and do not ask the user to share an API key or other credential in the conversation. Sign-in happens in the client's own authorization flow.

Do not invent alternate URLs, auth methods, or config keys.

## Reference Files

- `templates/` — ready-to-use config snippets for each file-configured client. Show the user the snippet from these files instead of reconstructing config from prose.
- `references/troubleshooting.md` — expanded troubleshooting table with client-specific notes. Read this when the user reports a connection issue.

## Setup Flow

1. Ask the user which client to configure: ChatGPT, Claude (web), Claude Code, Codex, Cursor, or Gemini CLI.
2. Confirm the user has the `Billing Profiles Admin` permission in DoiT.
3. Explain the auth flow for the chosen client:
   - ChatGPT: the authorization screen prompts for a DoiT API key and DoiT Customer ID. Tell the user to have both ready and to enter them there, not in this conversation.
   - Claude (web), Claude Code, Codex, Cursor, and Gemini CLI: the remote OAuth flow signs the user in through the browser. No API key is needed.
4. Follow the client-specific section below to configure the MCP server.
5. After MCP setup, offer to install the DCI CLI — see the `DCI CLI Installation` section. It is optional.

Do not write placeholder credentials into config files.

## File-Configured Clients

These clients read the server from a config file. Show the user the snippet from `templates/`, name the file it belongs in, and ask them to add it under the existing server list without removing other servers. The user makes the edit.

### Claude Code

If this skill is running from the DoiT plugin, the MCP server is already configured by the plugin. Tell the user to run `/mcp` in Claude Code and select `Authorize` for the DoiT server if it is not yet authorized. Nothing else is needed.

Without the plugin:

1. Recommend installing the plugin: `claude plugin marketplace add https://github.com/doitintl/doit-mcp-skills`, then `claude plugin install doit-mcp@doit-mcp-skills`.
2. If the user prefers a manual setup, show the snippet from `templates/claude-code.json` and tell them to add it with `claude mcp add-json doit-mcp '<snippet for the doit-mcp entry>'`, adding `--scope project` to limit it to one project.
3. Tell the user to run `/mcp` in Claude Code and select `Authorize` for the DoiT server.

### Cursor

Config file: `.cursor/mcp.json` in the project root.
Template: `templates/cursor.json`

1. Show the snippet from `templates/cursor.json`.
2. Tell the user to add the `doit-mcp` entry under `mcpServers` in `.cursor/mcp.json`, creating the file if it does not exist and keeping any existing servers.
3. Tell the user to complete Cursor's OAuth flow for the DoiT server if Cursor prompts for authentication.
4. Tell the user to restart Cursor or reload the MCP servers panel if the server does not appear after saving.

### Gemini CLI

Config file: Gemini CLI's `settings.json` in the user's Gemini configuration folder.
Template: `templates/gemini-cli.json`

1. Show the snippet from `templates/gemini-cli.json`.
2. Tell the user to add the `doit-mcp` entry under `mcpServers` in their Gemini CLI `settings.json`, keeping any existing servers.
3. Tell the user to run `/mcp auth doit-mcp` in Gemini CLI if the server requires authentication.
4. Tell the user to restart Gemini CLI if the server does not appear after saving.

### Codex

1. Tell the user to run `codex mcp add doit-mcp --url https://mcp.doit.com/sse`.
2. As an alternative, show this TOML snippet for the user to add to their Codex `config.toml`, keeping any existing `mcp_servers` entries:

   ```toml
   [mcp_servers.doit-mcp]
   enabled = true
   url = "https://mcp.doit.com/sse"
   ```
3. Tell the user to run `codex mcp list` to verify the server is registered, then complete the authorization flow if prompted.

## Manual Clients

These clients are configured in their own UI. Guide the user step by step.

### ChatGPT

1. Tell the user to enable ChatGPT developer mode first.
2. Create a connector from `Settings` -> `Apps & Connectors` -> `Create`.
3. Set `MCP Server URL` to `https://mcp.doit.com/sse`.
4. Select `OAuth` for auth.
5. Tell the user the authorization screen asks for both a DoiT API key and DoiT Customer ID, to be entered there.

Do not give a local config snippet for ChatGPT.

### Claude (web)

1. Tell the user to add a custom connector in Claude.
2. Set the remote MCP server URL to `https://mcp.doit.com/sse`.
3. Tell the user to click `Authorize` in the Claude UI and complete the OAuth flow.

Do not mention Customer ID for Claude web.

## DCI CLI Installation

After configuring the MCP client, offer to install the DoiT Cloud Intelligence CLI. The CLI provides direct terminal access to the same DoiT API that the MCP server uses. Install it only if the user agrees.

### Installation

Use the install command for the user's platform:

- **macOS**: `brew install doitintl/dci-cli/dci`
- **Windows (WinGet)**: `winget install DoiT.dci`
- **Windows (Scoop)**: `scoop bucket add doitintl https://github.com/doitintl/dci-cli && scoop install dci`
- **Linux (.deb)**: Download from [Releases](https://github.com/doitintl/dci-cli/releases/latest) and `sudo dpkg -i dci_*_linux_amd64.deb`
- **Linux (.rpm)**: Download from [Releases](https://github.com/doitintl/dci-cli/releases/latest) and `sudo rpm -i dci_*_linux_amd64.rpm`

### Authentication

1. Run `dci login` directly. This opens a browser window for OAuth authentication via the DoiT Console.
2. Tell the user to complete the authorization in the browser that just opened.
3. After the user confirms they authorized, run `dci status` to verify the CLI is authenticated.
4. For CI/CD or other non-interactive environments, point the user to the [DCI CLI documentation](https://github.com/doitintl/dci-cli) for non-interactive authentication.

### Quick Verification

After auth, run `dci list-reports` to confirm the CLI is working. If it returns a table of reports, the setup is complete.

## Verification

After setup, verify the connection:

1. Ask the user to confirm the DoiT MCP server appears in the client, shows as connected after authorization, and lists its tools.
2. Suggest a smoke test prompt: "Show me recent anomalies" or "List my reports".

## Troubleshooting

For detailed troubleshooting, read `references/troubleshooting.md`.

Quick checks:

- If the user mixed local and remote setup patterns, correct the flow instead of trying to merge them.
- If auth fails in Claude, Claude Code, Codex, Cursor, or Gemini CLI, verify the user completed the remote authorization flow in the client.
- If auth fails in ChatGPT, check whether the wrong credential was used: API key plus Customer ID.
- If the user says their config already has a `doit-mcp`, `doit`, or `doit_mcp_server` entry, tell them to replace it with the remote connector format for that client rather than keeping both.
- If the user asks for a client that is not covered here, say that supported clients are ChatGPT, Claude (web), Claude Code, Codex, Cursor, and Gemini CLI.

## Gotchas

- **Claude Code requires `"type": "sse"` and `"oauth": { "callbackPort": 8080 }`**: This is unique to Claude Code. Other clients infer the transport from the URL. Missing the `type` field or using `"type": "http"` causes a silent connection failure. The `oauth.callbackPort` is required for the OAuth callback to complete.
- **Cursor config is per-project**: The file lives at `.cursor/mcp.json` in the project root. Setting it globally does not work.
- **Codex uses TOML, not JSON**: JSON syntax in the Codex `config.toml` will break the file. Use the TOML snippet in the Codex section.
- **ChatGPT needs developer mode first**: Without enabling developer mode in Settings, the MCP connector option is not visible.
- **Do not mix local and remote**: If the user previously had a stdio-based setup (a local `npx` server with an API key), it must be fully removed before adding the remote HTTP setup. Mixing both causes duplicate or conflicting sessions.
- **OAuth tokens expire**: If a previously working connection stops, the most likely cause is an expired OAuth token. Re-authorize in the client UI.
- **`Billing Profiles Admin` is required**: Without this DoiT permission, tool calls will return 403 even if the connection succeeds. Verify this before starting setup.
