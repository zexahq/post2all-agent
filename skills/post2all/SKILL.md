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
2. Use the MCP tools directly for post management.
3. If the user asks to publish to specific platforms, confirm the target account IDs from the account listing first.
4. Prefer drafts or scheduled posts unless the user clearly asks to publish immediately.

## Notes

- This plugin is MCP-first and does not require API-key headers.
- Long-lived OAuth sessions depend on the client requesting refresh-token-capable scopes such as `offline_access`.
