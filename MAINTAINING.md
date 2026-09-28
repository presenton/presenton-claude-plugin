# Maintaining the Presenton Claude plugin

The plugin declares a remote HTTP MCP server in [`.mcp.json`](.mcp.json). The endpoint is `https://api.presenton.ai/skills/mcp` and must expose the tools described in [`mcp-contract.md`](skills/presenton/references/mcp-contract.md). The skill uses that connector; it does not run local helper scripts or call the REST API directly.

## Local validation

Load the repository in Claude Code with `claude --plugin-dir .`. Validate the plugin before a release:

```bash
claude plugin validate . --strict
```

## Build the upload ZIP

From the repository root:

```bash
mkdir -p dist
python3 -m zipfile -c dist/presenton-claude-plugin.zip \
  .claude-plugin/plugin.json .mcp.json LICENSE README.md \
  skills/presenton/SKILL.md \
  skills/presenton/references/html-authoring-system-prompt.md \
  skills/presenton/references/html-format.md \
  skills/presenton/references/mcp-contract.md
```

The ZIP contains the manifest, connector configuration, skill, required references, README, and license. It excludes the repository's direct REST API examples. Upload it through **Customize → Plugins** for testing. Connect Presenton from the plugin's **Connectors** tab.

## Directory releases

The [Claude developer portal](https://claude.ai/directory/manage) requires a paid Claude plan (Pro, Max, Team, or Enterprise); Free accounts cannot submit. Update an existing portal submission with **Check for new commits** rather than creating another submission for the same repository and plugin path. Raise the version in [plugin.json](.claude-plugin/plugin.json) for each release.

A listing submitted through the older Claude Console form stays as it is and does not gain the portal's new-version workflow until migrated. Follow [Anthropic's migration instructions](https://claude.com/docs/directory/publish): withdraw the Console submission when that option is available, or contact `directory@anthropic.com` to ask about moving it. A paid plan is still required to submit through the new portal.

Claude may show a general plugin trust warning during installation. This plugin has no hooks or local MCP commands; its MCP configuration references only the remote HTTPS endpoint. Claude controls connector tool approvals. The plugin cannot auto-approve them for users.
