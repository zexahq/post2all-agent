# post2all Agent Plugin

post2all Agent Plugin teaches AI coding assistants to create, schedule, inspect, update, cancel, and manage posts through post2all. It includes shared agent guidance plus the hosted Model Context Protocol connection for supported clients.

```text
https://mcp.post2all.com/mcp
```

## Capabilities

- List connected social accounts and their publishing capabilities.
- Inspect draft, scheduled, publishing, published, partially failed, and failed posts.
- Create posts with optional text and media (images, videos, or mixed when the platform allows).
- Save drafts, schedule future delivery, or publish immediately.
- Apply settings independently to every destination account.
- Discover dynamic publishing options such as Discord channels and TikTok privacy choices.
- Update draft, scheduled, failed, and partially failed posts while retained media is available; cancel scheduled posts.
- Inspect `deletion.available` and `deletion.reason` on published targets and remove one confirmed live social post on platforms where deletion is publicly supported.
- Remove posts from post2all when explicitly requested. This local deletion does not remove content already published to social platforms.

Actual capabilities depend on the connected accounts, platform restrictions, workspace permissions, and post2all plan.

Agents should list accounts, then call `publishing_schema` once with all selected account IDs before composing. It returns only public publishing capabilities, fixed choices, and account overrides such as X subscription limits. Call `publishing_options` only when the schema requests account discovery (for example Discord channels or TikTok creator restrictions). Do not send a fixed post type. Composition is inferred from attached media; mixed image/video is allowed only when a platform sets `media.allowMixedMedia`.

On MCP, call the read-only `post_validate` tool after constructing targets and before creating the post.

## Supported clients

This repository includes metadata and shared guidance for:

- Claude Code
- OpenAI Codex
- MCP-compatible clients that support a remote HTTP MCP server
- Coding agents with shell access to the `@post2all/cli` package

The shared MCP configuration lives in `.mcp.json`. Agent guidance is maintained in `skills/post2all/SKILL.md` and mirrored into the packaged plugin directory.

## Before you start

You need:

- A post2all account and workspace.
- At least one connected social account for publishing.
- A supported MCP client or the post2all CLI.
- A browser for the MCP OAuth consent flow.

The hosted MCP server uses OAuth 2.1. The CLI uses a post2all API key configured locally; agents should never ask users to paste API keys into chat.

## Claude Code setup

```bash
claude plugin marketplace add zexahq/post2all-agent
claude plugin install post2all@post2all-plugins
```

Complete the browser OAuth flow when Claude first connects to post2all.

## Codex setup

```bash
codex plugin marketplace add https://github.com/zexahq/post2all-agent
codex plugin add post2all@post2all-agent
```

Manual MCP configuration:

```toml
[mcp_servers.post2all]
url = "https://mcp.post2all.com/mcp"
```

Complete the browser sign-in flow and authorize the post2all workspace you want the agent to use.

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
  --content "Work in progress" \
  --delivery draft \
  --json
```

Publish immediately:

```bash
post2all post create \
  --content "New release shipping today 🚀" \
  --targets '[{"platform":"linkedin","accountId":"acc_linkedin_123","settings":{}}]' \
  --delivery now \
  --json
```

Schedule:

```bash
post2all post create \
  --content "Scheduled update" \
  --targets '[{"platform":"linkedin","accountId":"acc_linkedin_123","settings":{}}]' \
  --delivery scheduled \
  --scheduled-at "2026-07-20T09:00:00+05:30" \
  --json
```

## Example prompts

- “List my connected post2all accounts.”
- “Show the available Discord channels for this account.”
- “Create a draft LinkedIn post from this changelog.”
- “Schedule this announcement for tomorrow at 9 AM IST.”
- “Publish to LinkedIn and Threads, but use shorter copy on Threads.”
- “Show my scheduled posts for this week.”
- “Cancel the scheduled launch post.”

## Authentication and permissions

- MCP access is scoped to the workspace authorized through OAuth.
- The agent can only perform actions allowed by the user's post2all account and plan.
- Long-lived MCP sessions depend on refresh-token-capable scopes requested by the client.
- Access can be revoked from post2all account settings.

## Safety

Review immediate publishing, time-sensitive schedules, published deletion, and destructive post2all record deletion before confirming them. Prefer drafts for review workflows. Before deleting a live social post, use `post_get`, require `deletion.available` to be `true`, show the exact account/platform, and obtain explicit confirmation. When unavailable, use `deletion.reason` to explain why. That runtime state already includes public rollout and current account/platform constraints; private rollout platforms stay blocked on public MCP/CLI surfaces. Deleting the post2all record is a separate action and does not remove live social content.

## Troubleshooting

- Confirm the plugin or MCP server is enabled when tools are missing.
- Repeat OAuth sign-in when authorization fails.
- List accounts again when an account ID or platform is rejected.
- Load account publishing options when a required dynamic setting is missing.
- Check media type, size, target platform support, and required settings when publishing fails.

## Links

- post2all: https://www.post2all.com
- MCP setup: https://www.post2all.com/docs/mcp
- REST API: https://www.post2all.com/docs/api-reference
- Privacy policy: https://www.post2all.com/privacy-policy
- Repository: https://github.com/zexahq/post2all-agent
- Issues: https://github.com/zexahq/post2all-agent/issues
