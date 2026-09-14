---
name: post2all
description: Use post2all to create, schedule, inspect, update, cancel, manage, and analyze social posts across connected accounts.
---

# post2all

Use the `post2all` CLI when shell access is available, or the connected post2all MCP tools when the client exposes them. Both interfaces use the same target and delivery model and expose normalized account/post analytics.

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

## Embedded SaaS account connections

If the user is building post2all into their own SaaS, use the public REST API or official SDK for social-account onboarding. Do **not** claim MCP exposes account-connect/reconnect tools.

The headless flow is:

1. The SaaS backend authenticates to post2all with its workspace API key.
2. It calls `POST /api/v1/accounts/connect/:platform` with its own HTTPS `redirectUrl`.
3. post2all returns an opaque `connectionId` and, for OAuth platforms, an `authorizationUrl`.
4. The end user's browser opens the provider authorization URL. The end user does not need a post2all account, login, or browser session.
5. The provider returns to post2all, which validates one-time server-side state and stores the provider credentials.
6. post2all redirects the browser back to the SaaS with `post2all_connection_id` and `post2all_status` only.
7. The SaaS backend verifies the authoritative result with `GET /api/v1/accounts/connections/:connectionId` using its API key.

Keep the post2all API key on the SaaS backend. Never put it in frontend JavaScript or ask an end user to provide it. Reconnect through `POST /api/v1/accounts/:id/reconnect`; post2all requires the newly authorized provider identity to match the existing account. Disconnect through `DELETE /api/v1/accounts/:id`.

Reference: `https://www.post2all.com/docs/api-reference/account-connections`.

## Profiles and organized work

Profiles are an optional organization layer for clients, brands, or customers inside one post2all workspace. They are available on Business and Agency plans: Business supports 2 named profiles and Agency supports up to 10 profiles. Creator does not include profile operations.

Never invent a profile ID. On CLI, discover profiles with `post2all profile list --json`; on MCP, use `profiles_list`. Manage them with the corresponding `profile_*` tools/commands when the user asks.

For CLI account/post work, put `--profile-id <profileId>` on the root `post2all` command. For MCP tools that accept it, pass the same optional `profileId`. For REST/SDK integrations, the SDK helper `client.forProfile(profileId)` sends the shared `x-profile-id` header automatically. This is an organization/filtering context, not a separate security credential. Omitting profile context means the complete workspace.

Account assignment is an organization-level management action: `post2all account profile <accountId> <profileId>` assigns it; `--clear` removes the profile assignment. Each account belongs to at most one profile at a time. Moving an account changes where it appears in filtered account lists but does not move historical posts or break existing drafts, schedules, retries, analytics, or post targets. Deleting a profile never deletes its accounts or posts; they become profile-less.

For embedded SaaS onboarding, create/map a profile before starting the REST/SDK account connection when the integrator wants a convenient customer organization layer. Keep the workspace API key on the trusted SaaS backend because the workspace remains the authorization boundary. MCP can manage/filter profiles, but social account OAuth/connect/reconnect remains a REST/SDK/CLI flow rather than an MCP connection tool.

## Core workflow

1. Verify access.
2. List accounts and use returned IDs and platform values. Never guess either.
3. Call `publishing_schema` once with all selected account IDs and use its latest capabilities, fixed field values, and account-specific limits.
4. If that response includes discovery keys, call `publishing_options` once with the relevant selected account IDs. Do not call it for fixed fields.
5. Attach media as needed (none, images, videos, or a mix when `media.allowMixedMedia` is true). Do not send a fixed post type.
6. Build one typed target per destination and call `post_validate` before create.
7. Review the content and obtain explicit confirmation before scheduling or immediate publication, especially for TikTok.
8. Use a draft unless the user clearly requests scheduling or immediate publication.
9. Report the post ID, status, target accounts, and scheduled time.

```bash
post2all accounts --json
post2all constraints <accountId...> --json
post2all account publishing-options <accountId...> --json
```

Publishing options provide platform capabilities and dynamic values such as Pinterest boards, Discord channels, and TikTok creator information, including privacy choices, interaction restrictions, and video-duration limits.

For Pinterest image and video targets, `boardId` is required when publishing or scheduling. Choose it from the account's returned `boards` options; do not guess or omit it.

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

For post2all-managed media, upload local files first and pass the returned media IDs:

```bash
post2all media upload ./photo.jpg --json

post2all post create \
  --content "Photo update" \
  --media '[{"id":"media_123","altText":"Product dashboard with weekly engagement chart"}]' \
  --targets '[{"platform":"instagram","accountId":"acc_instagram_123","settings":{}}]' \
  --delivery now \
  --json
```

Do not pass local paths directly to post creation.

If the caller already hosts stable media at a public HTTPS URL, attach it directly without uploading a duplicate:

```bash
post2all post create \
  --content "Product demo" \
  --media '[{"url":"https://cdn.example.com/demo.mp4","altText":"Product demo walkthrough"}]' \
  --delivery draft \
  --json
```

Direct URL media remains caller-managed and must continue serving the intended file until all scheduled publishing attempts and retries complete. If a URL is temporary/signed, but post2all should retain the file, securely import it first:

```bash
post2all media upload --url "https://temporary.example.com/demo.mp4" --filename demo.mp4 --json
```

The URL import flow creates a normal post2all-managed media ID; after it succeeds, the original source URL may expire.

Composition is inferred from attached media. Each preferred media item uses either a managed `id` or a caller-managed public HTTPS `url`, never both. Mixed image+video is only valid for platforms with `media.allowMixedMedia: true` (for example Instagram, Threads, Telegram, Discord). When `publishing_schema` exposes `capability.media.altText`, set optional `altText` on each media item and obey its `mediaTypes` and `maxLength`. X intentionally does not expose media alt text. `mediaIds` remains a compatibility input for ID-only media; do not combine it with the preferred per-media shape.

PNG input is accepted when the selected capability includes `image/png`. Instagram and TikTok advertise `media.convertPngToJpeg: true`; post2all converts PNG images to JPEG only for those destinations while publishing. Other platforms retain PNG and transparency. This is automatic and requires no additional MCP or API input.

## Supported settings

- Twitter/X: `caption`
- LinkedIn: `caption`
- YouTube: `caption`, `title`, `description`, `tags`, `privacyStatus`, `categoryId`, `thumbnail`, `thumbnailTimestamp`
- Instagram: `caption`, `thumbnail`, `thumbnailTimestamp`, plus comment-hook settings (`instagramCommentHookEnabled`, `instagramCommentHookMatchMode`, `instagramCommentHookKeywords`, `instagramCommentHookExcludeKeywords`, `instagramCommentHookMatchType`, `instagramCommentHookReplyMessage`, `instagramCommentHookDmEnabled`, `instagramCommentHookPublicReplyEnabled`, `instagramCommentHookPublicReplyMessage`)
- Facebook: `caption`
- Pinterest: `caption`, `title`, `linkUrl`, `boardId`, `aiDisclosure`, `thumbnail`, `thumbnailTimestamp`; choose `boardId` from the returned `boards` options. Set `aiDisclosure: true` when the Pin was modified by AI.
- Threads: `caption`, `topicTag`
- Dribbble: `caption`, `title`, `description`, `tags`, `teamId`, `lowProfile`
- Bluesky: `caption`
- Telegram: `caption`, `linkUrl`, `linkText`, `disableNotification`, `protectContent`
- Discord: `caption`, `channelId`, `autoCrosspost`
- TikTok: `caption`, `title`, `tiktokContentPostingMethod`, `tiktokAutoAddMusic`, `tiktokPrivacyLevel`, `tiktokDisableComment`, `tiktokDisableDuet`, `tiktokDisableStitch`, `tiktokCommercialContentToggle`, `tiktokBrandOrganicToggle`, `tiktokBrandContentToggle`
- Wircle: no platform-specific target settings; use shared text/media and the capabilities returned by `publishing_schema`

For TikTok, `tiktokContentPostingMethod` is `DIRECT_POST` or `UPLOAD`. Direct Post requires manually selecting a creator-supported `tiktokPrivacyLevel`; never default it. For photo posts only, `tiktokAutoAddMusic` is an optional Direct Post setting and defaults to `false`; set it to `true` only when the user explicitly wants TikTok to add recommended music. TikTok's API does not allow selecting a specific sound. Interaction settings are opt-in: the `tiktokDisable*` fields should remain enabled (`true`/omitted by the UI's default) unless the user explicitly allows the interaction, and Duet/Stitch apply only to video posts. Upload sends media to the creator's TikTok inbox so they can finish and publish it in the TikTok app; privacy, interaction, commercial, and auto-add-music settings are not required for that mode.

When TikTok is selected, use the fresh `creatorInfo` response before constructing the target. Show the creator identity and review the media, caption/title, privacy, interactions, and commercial disclosure. If commercial disclosure is enabled, select at least one of `tiktokBrandOrganicToggle` or `tiktokBrandContentToggle`; branded content cannot use `SELF_ONLY`. Before scheduling or publishing, obtain the user's explicit consent to TikTok's Music Usage Confirmation, and also its Branded Content Policy when branded content is selected.

Do not invent settings. Use shared content by default and target-level `caption` only for account-specific copy. Alt text is media metadata, not a target setting.

For Instagram comment hooks, set `instagramCommentHookEnabled: true` on the target and at least one of the DM or the public reply: `instagramCommentHookDmEnabled` (default `true`) with `instagramCommentHookReplyMessage`, and/or `instagramCommentHookPublicReplyEnabled: true` with `instagramCommentHookPublicReplyMessage`. Match mode is `any` (every comment) or `keywords` with up to 10 `instagramCommentHookKeywords`; match type is `exact`, `contains`, `full_word`, or `begins_with`. `instagramCommentHookExcludeKeywords` always suppress the reply. Instagram sends at most one private DM per comment, so repeat commenters get nothing further. The connected Instagram account must have granted message permissions; if saving fails with a reconnect message, ask the user to reconnect the account and try again.

## Read and manage posts

```bash
post2all posts --status scheduled --limit 100 --json
post2all post get <postId> --json
```

Valid status filters are `draft`, `scheduled`, `publishing`, `published`, `completed`, `partially_failed`, and `failed`. A `completed` post succeeded but includes at least one upload-only target that still needs user action in the platform app.

Draft, scheduled, failed, and partially failed posts can be updated while retained media is available. Supplied `targets` and preferred `media` arrays replace existing arrays; replacements may use managed IDs and/or caller-managed public HTTPS URLs. `mediaIds` remains compatibility-only for ID-only attachments:

```bash
post2all post update <postId> \
  --content "Revised content" \
  --targets '[{"platform":"linkedin","accountId":"acc_linkedin_123","settings":{}}]' \
  --delivery scheduled \
  --scheduled-at "2026-07-21T10:00:00+05:30" \
  --json
```

Partially failed posts are retried without a target list. Successful accounts are immutable; retry only attempts targets whose current delivery status is failed. Expired-token accounts remain visible for reconnection from Accounts, while plan-disabled accounts are excluded.

Use:

    post2all post retry <postId> --json
    post2all post retry <postId> --scheduled-at "2026-07-21T10:00:00+05:30" --json

Publish an existing draft:

```bash
post2all post update <postId> --delivery now --json
```

Cancel a schedule while preserving the post:

```bash
post2all post cancel <postId> --json
```

Delete one already-published social copy while keeping the post2all record:

```bash
post2all post get <postId> --json
post2all post delete-published <postId> --post-account-id <postAccountId> --json
```

Use `post get` (or MCP `post_get`) first. Only offer deletion for a target whose `deletion.available` field is `true`, show the exact account/platform, and obtain explicit confirmation. When unavailable, use `deletion.reason` to explain why. On MCP, call `post_delete_published` with that one target ID. The runtime state already includes public rollout, account state, provider IDs, and limits such as Telegram's deletion window, and the server rechecks them during deletion. Private rollout platforms remain unavailable through public agent surfaces.

Permanently delete from post2all only after checking the post and confirming intent:

```bash
post2all post get <postId> --json
post2all post delete <postId> --json
```

This removes the post2all record and cancels any pending schedule. Content already published to social platforms remains live there; do not imply that this command removes the live social posts.

## Analytics

Use analytics tools only after identifying the exact connected account or post. Never guess account IDs, post IDs, metric names, or unavailable values.

For MCP:

- `analytics_account` returns normalized account metrics, time series, metric definitions, and previous-period comparisons for one connected account.
- `analytics_account_posts` lists provider-wide post performance. On supported platforms this includes posts published directly on the social platform, not only posts published through post2all.
- `analytics_post` returns analytics independently for every target of one post2all post, so one unavailable or failing provider does not hide successful analytics from the other targets.

CLI equivalents:

```bash
post2all account analytics <accountId> --json
post2all account analytics-posts <accountId> --sort-by views --sort-direction desc --json
post2all post analytics <postId> --json
```

Account analytics use the latest 30 inclusive UTC days by default. An explicit range can be at most 90 days:

```bash
post2all account analytics <accountId> \
  --start-date 2026-08-01 \
  --end-date 2026-08-31 \
  --json
```

Use `--refresh` (or MCP `refresh: true`) only when the user explicitly asks for fresh/current provider data or cached data is materially unsuitable. Normal requests should use post2all's analytics cache to reduce provider rate-limit pressure and improve latency.

Treat returned metric definitions and capabilities as authoritative. Different platforms expose different metrics. Do not invent a missing metric or convert an unavailable metric into zero.

For `analytics_account_posts`, `origin: "post2all"` means the provider post was matched to a post2all post. `origin: "external"` means the post was discovered from the provider but was not matched to post2all; such items intentionally may not have `postId` or `postAccountId`.

For `analytics_post`, always inspect `analyticsStatus`. Important states include `live`, `no_data`, `not_published`, `account_disconnected`, `account_disabled`, `reconnect_required`, `coming_soon`, `unavailable`, and `provider_error`. An empty `metrics` array does not by itself mean zero engagement.

## Errors and recovery

- `INVALID_API_KEY` / `EXPIRED_API_KEY`: configure a valid key.
- `INVALID_ACCOUNTS`: refresh account IDs and verify each target's platform.
- `ACCOUNT_NOT_FOUND`: refresh connected accounts and retry with a valid account ID.
- `ANALYTICS_UNAVAILABLE`: explain that analytics are not available for that account/platform instead of inventing data.
- `ANALYTICS_PROVIDER_ERROR`: report the provider analytics failure and retry only when useful; do not treat it as zero metrics.
- `INVALID_REQUEST`: inspect field-level issue paths such as `targets.0.settings.channelId`.
- `MEDIA_NOT_FOUND`: upload again or use valid media IDs from the same workspace.
- `MEDIA_UNAVAILABLE`: the source media or a crop derivative is missing or deleted while publishing. The affected account fails with the stable public code `MEDIA_UNAVAILABLE` and message `Media is unavailable.`; other accounts continue independently. Restore or re-upload the media before retrying that target.
- `UNSUPPORTED_MEDIA`: check media type, size, mixed-media rules, and platform capabilities.
- `POST_NOT_FOUND`: refresh the post list.
- `RATE_LIMITED`: wait before retrying.
- `PLAN_UPGRADE_REQUIRED` / `FORBIDDEN`: explain the plan or permission restriction instead of retrying.

Prefer `--json` whenever output will be parsed or used in a later command.
