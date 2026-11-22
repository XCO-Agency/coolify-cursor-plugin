# Coolify MCP

Cursor plugin that connects Cursor to the Model Context Protocol server already running on a Coolify Cloud or self-hosted Coolify instance. It is for people who manage that instance from Cursor and want the agent to read servers, projects, applications, databases, services, and deployments, and to run lifecycle actions when the token allows it.

The plugin is a client configuration plus three agent skills. Coolify serves the MCP endpoint. Each install uses that person's own instance and team token.

**Publisher:** [XCO Agency](https://xco.agency). Manifest name `coolify`, display name Coolify MCP, version `0.1.0`, MIT license.

## Repository layout

Cursor discovers components from their default folders. This repository is a single plugin, so there is no `.cursor-plugin/marketplace.json`.

| Path | Role |
| --- | --- |
| `.cursor-plugin/plugin.json` | Manifest: name, version, logo, and the variable schema |
| `mcp.json` | One remote MCP server, named `coolify` |
| `skills/coolify-setup/SKILL.md` | First-time connection |
| `skills/coolify-inspect/SKILL.md` | Read-only browsing |
| `skills/coolify-deploy/SKILL.md` | Deploy, restart, stop, and other lifecycle changes |
| `assets/logo.svg` | Logo referenced by the manifest |
| `.env.example` | Names of the two variables. The plugin does not load this file |
| `INSTALL-GROKBOT.md` | Remote MCP fields for Grok Bot / AddMcpServer |
| `CHANGELOG.md` | Release notes |
| `docs/development.md` | Local testing and publishing |

These default component folders are not in the repository, so the plugin loads none of them: `rules/`, `agents/`, `commands/`, `hooks/`.

`mcp.json` points the `coolify` server at the instance MCP URL and sends the team token as a bearer credential:

```json
{
  "mcpServers": {
    "coolify": {
      "url": "${COOLIFY_BASE_URL}/mcp",
      "headers": {
        "Authorization": "Bearer ${COOLIFY_API_TOKEN}"
      }
    }
  }
}
```

`${COOLIFY_BASE_URL}` and `${COOLIFY_API_TOKEN}` are plugin variables. Cursor substitutes the values entered for the plugin. A hand-written project or user MCP file uses a different syntax, described under [Manual remote MCP](#manual-remote-mcp).

Cursor's plugin reference names a plugin MCP server in approval prompts as `plugin-` plus the plugin name plus the server name. For this plugin that label is `plugin-coolify-coolify`.

## Requirements

- Cursor, with local plugin imports allowed if you are loading the folder yourself. On Enterprise that switch is **Allow Local Plugin Imports** (Dashboard → Settings → Security & Identity → Marketplace and Plugins) and it is off by default.
- A Coolify Cloud or self-hosted instance with a public HTTPS origin.
- Instance MCP enabled. Coolify Cloud already has it on. On self-hosted Coolify an instance admin turns it on under **Settings → Advanced → API and MCP → MCP server**. Until that is on, `/mcp` returns 404. The REST API access switch is separate and can stay off.
- Team MCP enabled for the team that owns the token. Check **Team → General → MCP server**. Team admins and owners can change it. A disabled team returns 403.
- On self-hosted Coolify, if **Settings → Advanced → Allowed IPs** is in use, the client IP has to be on that list. That list also applies to `/mcp`.
- A team API token from **Keys & Tokens → API Tokens**, created while that team is active. Coolify shows the full token once.

The setup skill in this repo still says **Settings → Configuration → Advanced → API and MCP**. Coolify's current setup guide uses **Settings → Advanced**. Follow the guide if the product menus and the skill text differ: [Set up MCP](https://coolify.io/docs/mcp/setup).

## Settings

Names only. Set the values in Cursor's plugin configuration or in your own secret store. Do not commit them.

| Name | Required | What it is |
| --- | --- | --- |
| `COOLIFY_BASE_URL` | yes | Coolify origin: HTTPS, no path, no trailing slash |
| `COOLIFY_API_TOKEN` | yes | Team-scoped API token for that origin |

Both names are required in `.cursor-plugin/plugin.json`. The same names are listed in `.env.example`. Exporting that file does not configure the plugin.

Token abilities are Coolify settings on the token, not plugin variables. Use the smallest set that matches the work:

| Ability | Used for |
| --- | --- |
| `read` | Inventory and inspection |
| `read:sensitive` | Sensitive reads and log summaries, including `get_logs` and `get_env_values` |
| `deploy` | Deploy, start, stop, restart, cancel, rollback |
| `write` | Create and update actions on `create_*` and `manage_*` tools |
| `delete` | Delete resources. `write` does not include delete |

The inspect skill stays on read-only tools. The deploy skill asks for a `deploy`-capable token and for an explicit go-ahead before production or irreversible actions. This plugin does not ship skills for `write` or `delete` workflows.

## Install in Cursor

The public marketplace page for this plugin name was not listed when these docs were written. See [Publishing](docs/development.md#publishing).

### From a marketplace

When the plugin is available in the Cursor Marketplace or a team marketplace:

1. Open **Customize** in the sidebar.
2. Install **coolify** and choose user or project scope.
3. Set `COOLIFY_BASE_URL` and `COOLIFY_API_TOKEN` when Cursor asks, or later under **Plugins → Configure**.

Skills show up under Agent Decides and can be invoked as `/coolify-setup`, `/coolify-inspect`, and `/coolify-deploy`.

A team admin can import this repository from **Dashboard → Plugins & MCPs** and set installation to Default Off, Default On, or Required. Importing the repo does not by itself install the plugin for every developer.

### Manual remote MCP

Use this when you want Coolify MCP without installing the plugin. Coolify's Cursor guide is [here](https://coolify.io/docs/mcp/clients/cursor).

1. Open Cursor settings.
2. Open **Tools & Integrations** and select **MCP**.
3. Add a remote MCP server.
4. Set the URL to the origin in `COOLIFY_BASE_URL` plus the path `/mcp`.
5. Set the `Authorization` header to `Bearer` followed by `COOLIFY_API_TOKEN`.
6. Save and enable the server.

Transport is Streamable HTTP. Leave SSE and stdio unset for this endpoint.

For a project file (`.cursor/mcp.json`) or a user file (`~/.cursor/mcp.json`), Cursor interpolates shell environment variables in `url` and `headers`:

```json
{
  "mcpServers": {
    "coolify": {
      "url": "${env:COOLIFY_BASE_URL}/mcp",
      "headers": {
        "Authorization": "Bearer ${env:COOLIFY_API_TOKEN}"
      }
    }
  }
}
```

That `${env:NAME}` form is for those files. The plugin's own `mcp.json` keeps the `${COOLIFY_BASE_URL}` / `${COOLIFY_API_TOKEN}` form.

Grok Bot and other Cursor agents that add a remote server by fields are covered in [INSTALL-GROKBOT.md](./INSTALL-GROKBOT.md).

### More than one team

A Coolify API token is bound to one team. This plugin declares one server and one pair of variables. For a second team, add another remote MCP server with its own name and its own token. The same `COOLIFY_BASE_URL` is fine when both teams live on one instance.

## Verify

Ask the agent to call `get_current_team` or `get_infrastructure_overview`. The team in the response should be the team that owns the token.

A token limited to selected projects, environments, or resources does not receive `get_current_team`. Use `get_infrastructure_overview` for that token.

| Status | Typical cause |
| --- | --- |
| 404 | Instance MCP is off (self-hosted), or the URL does not end in `/mcp` |
| 403 | Team MCP is off, or the client IP is missing from Allowed IPs |
| 401 | Token missing, expired, paused, revoked, or not sent as `Bearer` |

## Security

- Store `COOLIFY_API_TOKEN` in the plugin variable or the MCP secret field.
- Revoking the token cuts off MCP for clients that use it.
- Coolify hides deployment and lifecycle tools, `get_logs`, and `create_*` / `manage_*` actions the token cannot use.
- Responses omit private keys, S3 secrets, tokens, passwords, and webhook URLs. Full deployment logs are omitted. `get_env_values` is the tool that returns environment values; it needs `read:sensitive`, and the token owner must be a team admin or owner. Values marked shown once are omitted. Log summaries are capped and redacted on a best-effort basis.

## Develop and publish

Local loading, the version field, and marketplace submission are in [docs/development.md](docs/development.md).

## License

MIT. See [LICENSE](./LICENSE). Coolify remains under its own license.

Further reading: [Set up MCP](https://coolify.io/docs/mcp/setup) · [How MCP works](https://coolify.io/docs/mcp/how-mcp-works) · [Cursor client](https://coolify.io/docs/mcp/clients/cursor) · [Cursor plugins](https://cursor.com/docs/plugins) · [Plugin reference](https://cursor.com/docs/reference/plugins)
