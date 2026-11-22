# Develop and publish

Maintainer notes for this repository. Install and day-to-day configuration are in the [README](../README.md).

## Local test

There is no package to install and no test runner in the repo. Cursor loads the plugin from a local folder.

1. Copy this repository to `~/.cursor/plugins/local/coolify`, so that `.cursor-plugin/plugin.json` sits inside that folder.
2. Restart Cursor, or run **Developer: Reload Window**.
3. On Enterprise, turn on **Allow Local Plugin Imports** under Dashboard → Settings → Security & Identity → Marketplace and Plugins. It is off by default there, and it also gates **Import from Disk** in Customize.
4. Open **Customize** and confirm three skills (`coolify-setup`, `coolify-inspect`, `coolify-deploy`) and the `coolify` MCP server.
5. Set `COOLIFY_BASE_URL` and `COOLIFY_API_TOKEN` on the plugin.
6. Ask the agent to call `get_current_team` or `get_infrastructure_overview`.

Cursor only follows a symlink under `~/.cursor/plugins/local` when the target is also inside that folder. A symlink from that folder to a clone elsewhere is skipped. A marketplace install of a plugin with the same name takes precedence over the local copy.

Static checks you can run in a clone, with no Cursor window and no token:

```bash
python3 -m json.tool .cursor-plugin/plugin.json >/dev/null
python3 -m json.tool mcp.json >/dev/null
```

Then confirm each `skills/*/SKILL.md` starts with YAML frontmatter that includes `name` and `description`, and that those `name` values are `coolify-setup`, `coolify-inspect`, and `coolify-deploy`.

## Versioning

| Source | What it records |
| --- | --- |
| `.cursor-plugin/plugin.json` field `version` | Plugin version. Current value is `0.1.0` |
| `CHANGELOG.md` | Notes for that version. `0.1.0` is dated 2026-09-20 |

As of 2026-11-22, GitHub Releases for this repository are empty, and the repository has no `.github/workflows` and no Actions workflows. Shipping a change is a commit on the default branch plus, when you want the public listing updated, the marketplace re-index described below. Bump `version` and add a changelog section in the same change as any user-visible plugin update.

## Publishing

This is a single-plugin repository. Do not add `.cursor-plugin/marketplace.json` unless the repo starts hosting more than one plugin.

First listing:

1. Keep the Git repository public. Marketplace plugins are open source and reviewed by hand, including later updates.
2. Submit the repository at [cursor.com/marketplace/publish](https://cursor.com/marketplace/publish).
3. The manifest `name` must stay unique lowercase kebab-case (`coolify`). `logo` stays a relative path (`assets/logo.svg`). Every `${VAR}` in `mcp.json` stays declared under `variables`.

The publish form is the application. Cursor's publisher terms say a later change requests a re-index rather than a second application. This repository has no workflow that requests that re-index.

Checked on 2026-11-22: [cursor.com/marketplace/coolify](https://cursor.com/marketplace/coolify) responded as plugin not found, and `https://cursor.directory/plugins/coolify` returned 404. A team can still import the Git repository as a team marketplace from Dashboard → Plugins & MCPs without a public listing.

Official references: [Plugins](https://cursor.com/docs/plugins), [Plugins reference](https://cursor.com/docs/reference/plugins), [Marketplace publisher terms](https://cursor.com/marketplace-publisher-terms).
