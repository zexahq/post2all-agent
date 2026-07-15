# Post2All Agent Plugin

Post2All Agent Plugin teaches AI coding assistants to create, schedule, inspect, update, cancel, and delete social posts through Post2All. It includes shared agent guidance plus the hosted Model Context Protocol connection for supported clients.

```text
https://mcp.post2all.com/api/mcp
```

## Capabilities

- List connected social accounts and supported post types.
- Inspect draft, scheduled, publishing, published, partially failed, and failed posts.
- Create text, image, and video posts.
- Save drafts, schedule future delivery, or publish immediately.
- Apply settings independently to every destination account.
- Discover dynamic publishing options such as Discord channels and TikTok privacy choices.
- Update or cancel draft and scheduled posts.
- Delete posts when explicitly requested.

Actual capabilities depend on the connected accounts, platform restrictions, workspace permissions, and Post2All plan.

Agents should list accounts, then call `publishing_schema` once with all selected account IDs before composing. It returns only public publishing capabilities, fixed choices, and account overrides such as X subscription limits. Call `publishing_options` only when the schema requests account discovery (for example Discord channels or TikTok creator restrictions). Post creation still requires a single `text`, `image`, or `video` type; mixed image/video posts are not supported yet.

On MCP, call the read-only `post_validate` tool after constructing targets and before previewing or creating the post. Preview and validation share the same account, platform, schedule, media-presence, and content checks.

## Supported clients

This repository includes metadata and shared guidance for:

- Claude Code
- OpenAI Codex
- MCP-compatible clients that support a remote HTTP MCP server
- Coding agents with shell access to the `@post2all/cli` package

The shared MCP configuration lives in `.mcp.json`. Agent guidance is maintained in `skills/post2all/SKILL.md` and mirrored into the packaged plugin directory.

## Before you start

You need:

- A Post2All account and workspace.
- At least one connected social account for publishing.
- A supported MCP client or the Post2All CLI.
- A browser for the MCP OAuth consent flow.

The hosted MCP server uses OAuth 2.1. The CLI uses a Post2All API key configured locally; agents should never ask users to paste API keys into chat.

## Claude Code setup

```bash
claude plugin marketplace add https://github.com/zexahq/post2all-agent
claude plugin install post2all@post2all-plugins
```

Complete the browser OAuth flow when Claude first connects to Post2All.

## Codex setup

```bash
codex plugin marketplace add https://github.com/zexahq/post2all-agent
codex plugin add post2all@post2all-agent
```

Manual MCP configuration:

```toml
[mcp_servers.post2all]
url = "https://mcp.post2all.com/api/mcp"
```

Complete the browser sign-in flow and authorize the Post2All workspace you want the agent to use.

## Target and delivery model

Every destination is represented by a target containing its platform discriminator, social account ID, and settings:

```json
{
  "targets": [
    {
      "platform": "discord",
      "accountId": "acc_discord_123",
      "settings": {
        "channelId": "1234567890",
        "autoCrosspost": true
      }
    },
    {
      "platform": "threads",
      "accountId": "acc_threads_123",
      "settings": {
        "caption": "A shorter Threads version",
        "topicTag": "buildinpublic"
      }
    }
  ],
  "delivery": {
    "mode": "scheduled",
    "scheduledAt": "2026-07-20T09:00:00+05:30"
  }
}
```

The `platform` value selects the exact settings schema. Invalid cross-platform fields are rejected. Multiple accounts on the same platform are separate targets.

Delivery modes are:

- `draft`: save without publishing; targets and incomplete settings may be omitted.
- `now`: publish immediately.
- `scheduled`: publish at a timezone-aware ISO 8601 timestamp.

## CLI examples

```bash
post2all config whoami --json
post2all constraints <accountId...> --json
post2all accounts --json
post2all account publishing-options acc_discord_123 --json
```

Create a draft:

```bash
post2all post create \
  --type text \
  --content "Work in progress" \
  --delivery draft \
  --json
```

Publish immediately:

```bash
post2all post create \
  --type text \
  --content "New release shipping today 🚀" \
  --targets '[{"platform":"linkedin","accountId":"acc_linkedin_123","settings":{}}]' \
  --delivery now \
  --json
```

Schedule:

```bash
post2all post create \
  --type text \
  --content "Scheduled update" \
  --targets '[{"platform":"linkedin","accountId":"acc_linkedin_123","settings":{}}]' \
  --delivery scheduled \
  --scheduled-at "2026-07-20T09:00:00+05:30" \
  --json
```

## Example prompts

- “List my connected Post2All accounts.”
- “Show the available Discord channels for this account.”
- “Create a draft LinkedIn post from this changelog.”
- “Schedule this announcement for tomorrow at 9 AM IST.”
- “Publish to LinkedIn and Threads, but use shorter copy on Threads.”
- “Show my scheduled posts for this week.”
- “Cancel the scheduled launch post.”

## Authentication and permissions

- MCP access is scoped to the workspace authorized through OAuth.
- The agent can only perform actions allowed by the user's Post2All account and plan.
- Long-lived MCP sessions depend on refresh-token-capable scopes requested by the client.
- Access can be revoked from Post2All account settings.

## Safety

Review immediate publishing, time-sensitive schedules, and destructive deletes before confirming them. Prefer drafts for review workflows. Fetch a post before deleting it unless the user already clearly identified it and requested deletion.

## Troubleshooting

- Confirm the plugin or MCP server is enabled when tools are missing.
- Repeat OAuth sign-in when authorization fails.
- List accounts again when an account ID or platform is rejected.
- Load account publishing options when a required dynamic setting is missing.
- Check media type, size, target platform support, and required settings when publishing fails.

## Links

- Post2All: https://www.post2all.com
- MCP setup: https://www.post2all.com/docs/mcp-server
- REST API: https://www.post2all.com/docs/api-reference
- Repository: https://github.com/zexahq/post2all-agent
- Issues: https://github.com/zexahq/post2all-agent/issues
