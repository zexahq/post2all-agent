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
3. Call `publishing_schema` once with all selected account IDs and use its latest capabilities, fixed field values, and account-specific limits.
4. If that response includes discovery keys, call `publishing_options` once with the relevant selected account IDs. Do not call it for fixed fields.
5. Attach media as needed (none, images, videos, or a mix when `media.allowMixedMedia` is true). Do not send a fixed post type.
6. Build one typed target per destination and call `post_validate` before preview or create.
7. Preview the content and obtain explicit confirmation before scheduling or immediate publication, especially for TikTok.
8. Use a draft unless the user clearly requests scheduling or immediate publication.
9. Report the post ID, status, target accounts, and scheduled time.

```bash
post2all accounts --json
post2all constraints <accountId...> --json
post2all account publishing-options <accountId...> --json
```

Publishing options provide platform capabilities and dynamic values such as Discord channels and TikTok creator information, including privacy choices, interaction restrictions, and video-duration limits.

Treat `publishing_schema` as authoritative. Do not rely on memorized limits or enum values. It contains public publishing metadata only; use `publishing_options` for dynamic choices such as Discord channels and TikTok creator restrictions. Post composition is inferred from attached media (text-only, images, videos, or a mix when a platform sets media.allowMixedMedia). Do not send a fixed post type.

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
  --content "Work in progress" \
  --delivery draft \
  --json
```

Publish immediately only when explicitly requested:

```bash
post2all post create \
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
  --content "Photo update" \
  --media-ids media_123 \
  --targets '[{"platform":"instagram","accountId":"acc_instagram_123","settings":{"altText":"Product dashboard"}}]' \
  --delivery now \
  --json
```

Do not pass local paths directly to post creation.

Composition is inferred from attached media. Mixed image+video is only valid for platforms with `media.allowMixedMedia: true` (for example Instagram, Threads, Telegram, Discord).

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
- TikTok: `caption`, `title`, `description`, `tiktokContentPostingMethod`, `tiktokAutoAddMusic`, `tiktokPrivacyLevel`, `tiktokDisableComment`, `tiktokDisableDuet`, `tiktokDisableStitch`, `tiktokCommercialContentToggle`, `tiktokBrandOrganicToggle`, `tiktokBrandContentToggle`

For TikTok, `tiktokContentPostingMethod` is `DIRECT_POST` or `UPLOAD`. Direct Post requires manually selecting a creator-supported `tiktokPrivacyLevel`; never default it. For photo posts only, `tiktokAutoAddMusic` is an optional Direct Post setting and defaults to `false`; set it to `true` only when the user explicitly wants TikTok to add recommended music. TikTok's API does not allow selecting a specific sound. Interaction settings are opt-in: the `tiktokDisable*` fields should remain enabled (`true`/omitted by the UI's default) unless the user explicitly allows the interaction, and Duet/Stitch apply only to video posts. Upload sends media to the creator's TikTok inbox so they can finish and publish it in the TikTok app; privacy, interaction, commercial, and auto-add-music settings are not required for that mode.

When TikTok is selected, use the fresh `creatorInfo` response before constructing the target. Show the creator identity and preview the media, caption/title/description, privacy, interactions, and commercial disclosure. If commercial disclosure is enabled, select at least one of `tiktokBrandOrganicToggle` or `tiktokBrandContentToggle`; branded content cannot use `SELF_ONLY`. Before scheduling or publishing, obtain the user's explicit consent to TikTok's Music Usage Confirmation, and also its Branded Content Policy when branded content is selected.

Do not invent settings. Use shared content by default and target-level `caption` only for account-specific copy.

## Read and manage posts

```bash
post2all posts --status scheduled --limit 100 --json
post2all post get <postId> --json
```

Valid status filters are `draft`, `scheduled`, `publishing`, `published`, `completed`, `partially_failed`, and `failed`. A `completed` post succeeded but includes at least one upload-only target that still needs user action in the platform app.

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
- `UNSUPPORTED_MEDIA`: check media type, size, mixed-media rules, and platform capabilities.
- `POST_NOT_FOUND`: refresh the post list.
- `RATE_LIMITED`: wait before retrying.
- `PLAN_UPGRADE_REQUIRED` / `FORBIDDEN`: explain the plan or permission restriction instead of retrying.

Prefer `--json` whenever output will be parsed or used in a later command.
