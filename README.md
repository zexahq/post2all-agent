# Post2All Agent Plugin

Post2All Agent Plugin connects AI coding assistants to your Post2All workspace through the Model Context Protocol (MCP). Once connected, supported clients can list your connected social accounts and help create, schedule, update, cancel, or delete social posts without leaving your editor or agent session.

The plugin uses the hosted Post2All MCP server:

```text
https://www.post2all.com/api/mcp
```

## What You Can Do

- List connected social accounts across supported platforms.
- Inspect existing draft, scheduled, published, or failed posts.
- Create new text, image, or video posts.
- Save posts as drafts, schedule them for later, or publish immediately.
- Update draft or scheduled posts before they go live.
- Cancel scheduled posts or delete posts when needed.

Actual capabilities depend on your Post2All workspace, connected accounts, plan limits, and account permissions.

## Supported Clients

This repository includes plugin metadata and shared guidance for:

- Claude Code
- OpenAI Codex
- Any MCP-compatible client that can connect to a remote HTTP MCP server

The shared MCP configuration lives in `.mcp.json`, and the shared agent guidance lives in `skills/post2all/SKILL.md`.

## Before You Start

Make sure you have:

- A Post2All account and workspace.
- At least one connected social account in Post2All.
- A supported MCP client such as Claude Code or Codex.
- A browser available for the OAuth sign-in and consent flow.

Post2All MCP access is OAuth-based. API keys are not used for this plugin.

## Claude Code Setup

Add this repository as a Claude Code plugin marketplace:

```bash
claude plugin marketplace add https://github.com/zexahq/post2all-agent
claude plugin install post2all@post2all-plugins
```

After installation, enable the plugin if prompted. When Claude Code first connects to Post2All, complete the browser-based OAuth flow and grant access to your active workspace.

## Codex Setup

This repository includes a Codex plugin manifest at `.codex-plugin/plugin.json` and a shared MCP config at `.mcp.json`.

If your Codex environment supports installing plugins from public repositories, install this repository as the Post2All plugin source:

```text
https://github.com/zexahq/post2all-agent
```

If you configure MCP manually, add the Post2All MCP server URL to your Codex MCP configuration:

```toml
[mcp_servers.post2all]
url = "https://www.post2all.com/api/mcp"
```

When Codex connects, complete the OAuth flow in your browser and select the Post2All workspace you want the agent to use.

## Example Workflows

Once connected, try prompts like:

- "List my connected Post2All accounts."
- "Create a draft LinkedIn post from this changelog."
- "Schedule this announcement for tomorrow at 9 AM."
- "Show my scheduled posts for this week."
- "Cancel the scheduled post about the launch update."

For platform-specific publishing, ask the agent to list accounts first so you can confirm the correct target account.

## Authentication And Permissions

The MCP server uses OAuth 2.1. Your agent client will open a browser sign-in flow the first time it needs access.

- Access is scoped to the Post2All workspace you authorize.
- The agent can only act through the permissions granted to your Post2All account.
- Long-lived sessions depend on whether your MCP client requests refresh-token-capable scopes such as `offline_access`.
- You can revoke access from your Post2All account settings when you no longer want a client to connect.

## Safety Notes

MCP clients can take actions in connected tools using your account permissions. Review high-impact or time-sensitive actions before confirming them, especially publish-now requests, scheduled campaigns, and deletes.

For safer workflows, prefer drafts or scheduled posts unless you explicitly want to publish immediately.

## Troubleshooting

- If tools are unavailable, confirm the plugin or MCP server is enabled in your client.
- If authorization fails, repeat the browser sign-in flow and ensure an active Post2All workspace is selected.
- If no accounts appear, connect social accounts in Post2All first.
- If publishing fails, check account permissions, media requirements, platform limits, and your Post2All plan limits.

## Links

- Post2All: https://www.post2all.com
- MCP setup docs: https://www.post2all.com/docs/mcp-server
- Repository: https://github.com/zexahq/post2all-agent
- Issues: https://github.com/zexahq/post2all-agent/issues
