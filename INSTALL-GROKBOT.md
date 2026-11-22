# Install on Grok Bot / Cursor agents

Coolify MCP is a remote Streamable HTTP server on your Coolify instance. This plugin does not ship a local package to run.

Put `COOLIFY_API_TOKEN` in a secret field or a plugin variable. The two setting names are `COOLIFY_BASE_URL` and `COOLIFY_API_TOKEN`. Full Cursor install steps are in the [README](./README.md).

## AddMcpServer (remote)

| Field | Value |
| --- | --- |
| name | `coolify`, or a distinct name such as `coolify-staging` / `coolify-production` when you add more than one team |
| url | The origin in `COOLIFY_BASE_URL`, plus the path `/mcp` |
| headers | Header name `Authorization`. Value is the `Bearer` scheme and the `COOLIFY_API_TOKEN` secret |

AddMcpServer takes that resolved URL and header. It does not read `.cursor-plugin/plugin.json`. The plugin file `mcp.json` is the place that uses the placeholders `${COOLIFY_BASE_URL}` and `${COOLIFY_API_TOKEN}`. A project or user `mcp.json` that reads the shell uses `${env:COOLIFY_BASE_URL}` and `${env:COOLIFY_API_TOKEN}` instead.

## Multiple Coolify teams

An API token is team-scoped. For two teams, add two MCP servers with different names and different tokens. The same `COOLIFY_BASE_URL` is fine when both teams are on one instance.

## Verify

Ask the agent to call `get_current_team` or `get_infrastructure_overview`. The team should be the one bound to that token. A token limited to selected projects, environments, or resources does not receive `get_current_team`.

## Permissions

| Ability | When to add it |
| --- | --- |
| `read` | Inspect resources |
| `read:sensitive` | Log summaries and `get_env_values` |
| `deploy` | Deploy, start, stop, restart, cancel, rollback |
| `write` | Create or update resources |
| `delete` | Delete resources |
