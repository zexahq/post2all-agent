---
name: post2all
description: Use Post2All MCP tools to inspect connected social accounts and create, schedule, update, cancel, or delete posts.
---

# Post2All

This plugin connects agent clients to Post2All through the hosted MCP server.

## What To Use

Prefer the Post2All MCP tools from this plugin when the user wants to:

- list connected Post2All accounts
- inspect existing posts
- create draft, scheduled, or publish-now posts
- update, cancel, or delete scheduled content

## Authentication

The MCP connection uses OAuth in the browser.

If the Post2All tools are unavailable or fail with an auth error:

1. Ask the user to enable or install the plugin if it is not already available.
2. Ask the user to complete the browser sign-in flow for Post2All.
3. Make sure they have an active workspace selected in Post2All before granting access.

## Workflow

1. Start by listing accounts when account selection matters.
2. For new or edited content, call `post_prepare_preview` before `post_create` whenever the tool is available. The preview validates platform limits, shows final platform captions, and does not create or publish anything.
3. If the user asks to publish to specific platforms, confirm the target account IDs from the account listing first.
4. Use `platformSettings` for platform-specific captions or fields. Do not invent account-specific settings. Example: `{"threads":{"caption":"Short Threads version"},"youtube":{"title":"Video title"}}`.
5. If a platform has a lower caption limit, keep the main content for platforms with room and use `platformSettings[platform].caption` for the shorter platform version.
6. After the user approves the preview, call `post_create` with the same final content, selected accounts, status, schedule, media, and `platformSettings`.
7. Prefer drafts or scheduled posts unless the user clearly asks to publish immediately.

## Scheduling Time

Scheduled times must include a timezone. Use ISO 8601 with `Z` or an explicit offset when calling tools, for example `2026-06-20T09:00:00+05:30`.

If the user gives a local time, interpret it in the user's local timezone unless they explicitly name another timezone. If the timezone is unclear, ask one short clarifying question before scheduling.

## Notes

- This plugin is MCP-first and does not require API-key headers.
- Long-lived OAuth sessions depend on the client requesting refresh-token-capable scopes such as `offline_access`.
- Post2All enforces platform caption limits on the final effective platform caption: custom platform caption first, otherwise the main content.
