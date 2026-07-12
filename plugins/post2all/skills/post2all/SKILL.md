---
name: post2all
description: Use Post2All to create, schedule, inspect, update, cancel, and delete social posts across connected accounts.
---

# Post2All

Use the `post2all` CLI when shell access is available, or the connected Post2All MCP tools when the client exposes them. Both interfaces use the same target and delivery model.

## Install and authenticate the CLI

Check availability:

```bash
post2all --help
```

Use the package without a global install when needed:

```bash
npx @post2all/cli <command>
```

Before workspace operations, validate authentication:

```bash
post2all config whoami --json
```

Credentials are resolved from `--api-key`, `POST2ALL_API_KEY`, or the local config created by `post2all config set-key`. Never print, repeat, log, or commit an API key. Do not ask the user to paste one into chat.

The hosted MCP server uses OAuth instead of API keys.

## Core workflow

1. Verify access.
2. List accounts and use returned IDs and platform values. Never guess either.
3. Check `supportedPostTypes` before selecting `text`, `image`, or `video`.
4. Load publishing options for accounts with dynamic fields.
5. Build one typed target per destination.
6. Use a draft unless the user clearly requests scheduling or immediate publication.
7. Report the post ID, status, target accounts, and scheduled time.

```bash
post2all accounts --json
post2all account publishing-options <accountId> --json
```

Publishing options provide platform capabilities and dynamic values such as Discord channels and TikTok privacy choices.

## Target model

Each destination is represented independently:

```json
{
  "platform": "discord",
  "accountId": "acc_discord_123",
  "settings": {
    "channelId": "1234567890",
    "autoCrosspost": true
  }
}
```

`platform` is a schema discriminator. Settings from another platform are rejected. Multiple accounts on the same platform must be separate target objects.

For MCP tools, pass the same structure in the `targets` array. For the CLI, serialize the array into `--targets`.

## Delivery model

Use one of:

```json
{ "mode": "draft" }
```

```json
{ "mode": "now" }
```

```json
{
  "mode": "scheduled",
  "scheduledAt": "2026-07-20T09:00:00+05:30"
}
```

CLI equivalents are `--delivery draft`, `--delivery now`, or `--delivery scheduled --scheduled-at <timestamp>`.

Drafts may omit targets and incomplete publishing settings. Immediate and scheduled delivery require complete targets, appropriate media, and all platform-required settings. No CLI delivery flag defaults to a draft.

Always use ISO 8601 scheduled timestamps with `Z` or an explicit UTC offset. Resolve relative times in the user's timezone and never submit a time in the past.

## Create posts

Draft:

```bash
post2all post create \
  --type text \
  --content "Work in progress" \
  --delivery draft \
  --json
```

Publish immediately only when explicitly requested:

```bash
post2all post create \
  --type text \
  --content "New release shipping today 🚀" \
  --targets '[
    {
      "platform": "linkedin",
      "accountId": "acc_linkedin_123",
      "settings": {}
    },
    {
      "platform": "threads",
      "accountId": "acc_threads_123",
      "settings": {
        "caption": "Short Threads version",
        "topicTag": "buildinpublic"
      }
    }
  ]' \
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

## Media

Upload local files first and pass returned media IDs:

```bash
post2all media upload ./photo.jpg --json

post2all post create \
  --type image \
  --content "Photo update" \
  --media-ids media_123 \
  --targets '[{"platform":"instagram","accountId":"acc_instagram_123","settings":{"altText":"Product dashboard"}}]' \
  --delivery now \
  --json
```

Do not pass local paths directly to post creation.

## Supported settings

- Twitter/X: `caption`, `altText`
- LinkedIn: `caption`
- YouTube: `caption`, `title`, `description`, `tags`, `privacyStatus`, `categoryId`, `thumbnail`, `thumbnailTimestamp`
- Instagram: `caption`, `altText`, `thumbnail`, `thumbnailTimestamp`
- Facebook: `caption`
- Pinterest: `caption`, `boardId`, `altText`, `thumbnail`, `thumbnailTimestamp`
- Threads: `caption`, `altText`, `topicTag`
- Dribbble: `caption`, `title`, `description`, `tags`, `teamId`, `lowProfile`
- Bluesky: `caption`, `altText`
- Telegram: `caption`, `linkUrl`, `linkText`, `disableNotification`, `protectContent`
- Discord: `caption`, `channelId`, `autoCrosspost`
- TikTok: `caption`, `title`, `description`, `tiktokPrivacyLevel`, `tiktokDisableComment`, `tiktokDisableDuet`, `tiktokDisableStitch`

Do not invent settings. Use shared content by default and target-level `caption` only for account-specific copy.

## Read and manage posts

```bash
post2all posts --status scheduled --limit 100 --json
post2all post get <postId> --json
```

Valid status filters are `draft`, `scheduled`, `publishing`, `published`, `partially_failed`, and `failed`.

Only draft and scheduled posts can be updated. Supplied `targets` and `mediaIds` arrays replace existing arrays:

```bash
post2all post update <postId> \
  --content "Revised content" \
  --targets '[{"platform":"linkedin","accountId":"acc_linkedin_123","settings":{}}]' \
  --delivery scheduled \
  --scheduled-at "2026-07-21T10:00:00+05:30" \
  --json
```

Publish an existing draft:

```bash
post2all post update <postId> --delivery now --json
```

Cancel a schedule while preserving the post:

```bash
post2all post cancel <postId> --json
```

Permanently delete only after checking the post and confirming intent:

```bash
post2all post get <postId> --json
post2all post delete <postId> --json
```

## Errors and recovery

- `INVALID_API_KEY` / `EXPIRED_API_KEY`: configure a valid key.
- `INVALID_ACCOUNTS`: refresh account IDs and verify each target's platform.
- `INVALID_REQUEST`: inspect field-level issue paths such as `targets.0.settings.channelId`.
- `MEDIA_NOT_FOUND`: upload again or use valid media IDs from the same workspace.
- `UNSUPPORTED_MEDIA`: check post type, media type, size, and platform capabilities.
- `POST_NOT_FOUND`: refresh the post list.
- `RATE_LIMITED`: wait before retrying.
- `PLAN_UPGRADE_REQUIRED` / `FORBIDDEN`: explain the plan or permission restriction instead of retrying.

Prefer `--json` whenever output will be parsed or used in a later command.
