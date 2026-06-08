# Post2All Agent Plugin

Shared agent plugin bundle for Post2All.

This plugin is MCP-first and connects supported agent clients to the hosted Post2All MCP server at `https://www.post2all.com/api/mcp`.

## Structure

```text
post2all-agent/
├── .agents/
│   └── plugins/
│       └── marketplace.json
├── .claude-plugin/
│   ├── marketplace.json
│   └── plugin.json
├── .codex-plugin/
│   └── plugin.json
├── .mcp.json
├── skills/
│   └── post2all/
│       └── SKILL.md
└── README.md
```

## Claude Code Local Testing

From this workspace root:

```bash
claude plugin marketplace add ./post2all-agent
claude plugin install post2all@post2all-plugins
```

Then enable the plugin and let Claude Code complete the OAuth flow in the browser when it first connects.

## Codex Local Testing

This repository uses the same shared `.mcp.json` and `skills/` folder for Codex. The Codex plugin manifest lives at `.codex-plugin/plugin.json`.

Validate the Codex plugin from this workspace root:

```bash
python3 /Users/bhavikagarwal/.codex/skills/.system/plugin-creator/scripts/validate_plugin.py ./post2all-agent
```

The repo-local Codex marketplace metadata lives at `.agents/plugins/marketplace.json` for local development and future distribution wiring.

## Authentication

- MCP auth is OAuth-only
- No API-key bearer headers are used for MCP
- The user must have an active Post2All workspace selected before granting access

## Current Scope

The plugin bundles:

- plugin manifest metadata
- local marketplace manifests for distribution/testing
- hosted MCP server configuration
- shared agent skill guidance that teaches when to use the Post2All MCP tools
