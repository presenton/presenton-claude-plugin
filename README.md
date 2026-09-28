# Presenton Claude Plugin

An Agent Skill and Claude plugin for creating presentation files with [Presenton](https://presenton.ai).

The included `presenton` skill turns presentation requests into editable PPTX files, PDFs, and PNG slide images. It uses Presenton's remote MCP connector to search designs and icons, import user-provided images, validate generated slide HTML, export each requested format, and return a shareable preview link.

## Get the skill

- [ClawHub](https://clawhub.ai/presenton/skills/presenton)
- [skills.sh](https://www.skills.sh/presenton/skills/presenton)

You can also copy [`skills/presenton`](skills/presenton) into the skills directory used by your compatible agent.

## Install as a Claude plugin

The plugin bundles a remote MCP connection to `https://api.presenton.ai/skills/mcp`, allowing its tools to work from Claude.ai without direct code-execution egress to the Presenton REST API. The endpoint must be deployed as a Streamable HTTP MCP server exposing the six tools documented in [`skills/presenton/references/mcp-contract.md`](skills/presenton/references/mcp-contract.md).

Build the upload ZIP from the repository root:

```bash
mkdir -p dist
python3 -m zipfile -c dist/presenton-claude-plugin.zip \
  .claude-plugin/plugin.json .mcp.json LICENSE README.md \
  skills/presenton/SKILL.md \
  skills/presenton/references/html-authoring-system-prompt.md \
  skills/presenton/references/html-format.md \
  skills/presenton/references/mcp-contract.md
```

Upload `dist/presenton-claude-plugin.zip` through **Customize → Plugins** in Claude.ai. Enable the plugin, connect Presenton from its **Connectors** tab, start a new conversation, and invoke `/presenton:presenton` with a presentation request. The ZIP contains only the manifest, remote MCP configuration, skill instructions, required references, README, and license. It contains no local runtime helpers or tests; the direct REST API examples remain repository-only documentation.

Claude may still show a general trust warning during installation. That prompt is controlled by Claude and cannot be suppressed in this plugin's manifest. It does not by itself mean the ZIP starts a local process: this plugin declares only an HTTPS MCP server and has no hooks or local MCP commands. Review the ZIP and the remote connector before continuing.

### Claude.ai tool approvals

Claude.ai controls MCP tool approvals; a plugin cannot silently auto-approve its own tools. Custom connector tools may initially show **Allow once**, **Always allow**, and **Deny**. If you trust Presenton and do not want a confirmation for every call, go to **Customize → Connectors → Presenton → Tool permissions** and set the required tools—or the connector—to **Always allow**. You may also choose **Always allow** in the approval dialog. Permissions can be requested separately for different tools, and reinstalling or replacing the plugin may require approving the tools again.

Test the repository directly during development:

```bash
claude --plugin-dir .
```

The installed skill is namespaced as `/presenton:presenton`.

Before publishing a release, validate the plugin from the repository root:

```bash
claude plugin validate . --strict
```

The repository includes the required [`.claude-plugin/plugin.json`](.claude-plugin/plugin.json) manifest. Submit the repository through the [Claude developer portal](https://claude.ai/directory/manage). To release a new version, update the existing submission there and select **Check for new commits**; do not create a second submission for the same repository and plugin path. If version 1 was submitted through the older Claude Console form, follow [Anthropic's migration instructions](https://claude.com/docs/directory/publish): withdraw it in Console when that option is available, or contact `directory@anthropic.com` to move it to the developer portal.

## What it supports

- PPTX, PDF, and PNG exports
- Custom design briefs or Presenton design search
- User-provided images and searchable icons
- Editable HTML text in generated PowerPoint files
- Shareable presentation previews
- HTML structure, asset, and font validation before export
- Remote MCP operation from Claude.ai, Claude Desktop, Cowork, and Claude Code

The Claude plugin connects to Presenton's remote MCP endpoint at `https://api.presenton.ai/skills/mcp`; no API key is configured in the plugin.

## Repository layout

```text
.claude-plugin/
└── plugin.json              # Claude Code plugin manifest
.mcp.json                    # Presenton remote MCP connection
skills/presenton/
├── SKILL.md                  # Skill instructions and workflow
├── agents/openai.yaml        # Agent-facing metadata
└── references/               # MCP contract and HTML format documentation
```

## License

Licensed under the [Apache License 2.0](LICENSE).
