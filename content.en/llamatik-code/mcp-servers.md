---
title: "MCP Servers"
weight: 3
---

# MCP Servers

Llamatik Code is an IDE-native MCP (Model Context Protocol) host. Connect any MCP-compatible tool server and let the agent call its tools through the same pipeline as built-in tools.

*Available since version 1.4.0.*

## Adding a server

1. Open **Settings → Llamatik Code → MCP Servers**.
2. Click **Add server**.
3. Fill in the server details:

| Field | Description |
|---|---|
| Name | A label for the server (e.g., `github`, `filesystem`) |
| Command | The executable to run (e.g., `npx`, `uvx`, `python`) |
| Arguments | One per line (e.g., `-y @modelcontextprotocol/server-github`) |
| Environment variables | `KEY=VALUE` pairs for secrets or config |
| Working directory | Optional; defaults to the project root |
| Auto-start | Start automatically when the project opens |
| Trust level | LOW / MEDIUM / HIGH — affects default approval behaviour |

4. Click **Test connection** to verify the server starts and lists its tools.

## Managing servers

- **Enable / disable** — toggle a server without removing its configuration.
- **Refresh tools** — re-discovers tools from a running server after an update.
- **View logs** — shows the last 1000 lines of stderr. Useful for debugging.
- **Edit** — modify any field; changes take effect on the next connection.
- **Remove** — stops the server and removes the configuration.

## How MCP tools appear in the agent

Discovered MCP tools are bridged as first-class agent tools under `mcp_<serverName>_<toolName>`. They appear in the agent trace like built-in tools.

You can also control MCP servers directly using six built-in agent tools:

| Tool | Description |
|---|---|
| `mcp_list_servers` | List all configured servers and their connection status |
| `mcp_list_tools` | List tools exposed by a specific server |
| `mcp_call_tool` | Call a tool on a specific server with arguments |
| `mcp_list_resources` | List resources exposed by a server |
| `mcp_read_resource` | Read a named resource from a server |
| `mcp_list_prompts` | List prompt templates exposed by a server |

## Approval flow

Every MCP tool call triggers an approval dialog showing the server name, tool name, arguments, and risk level.

**Risk levels:**
- `LOW` — read-only or informational tools
- `MEDIUM` — tools that modify local state
- `HIGH` — tools containing `write`, `delete`, `execute`, `deploy`, `send`, `push`, or similar keywords — **never auto-approved**

### Free tier
- **Allow once** or **Deny** on every call.
- No persistent approval rules.

### Pro tier
- **Allow always for this tool** — skip future prompts for this specific tool.
- **Allow always for this server** — auto-approve all non-HIGH-risk tools from this server.

## Security

- MCP servers run as isolated child processes. A crashed server does not affect other features.
- Secrets (API keys, tokens, `Authorization` header values) are redacted from all logs and trace output.
- The UI displays a warning when a server's description indicates it may call external APIs.

## Free vs Pro

| | Free | Pro |
|---|---|---|
| Active servers | 1 | Unlimited |
| Auto-approval rules | — | Per-tool + per-server |
| Autonomous multi-step use | — | Yes |
