---
name: post2all
description: Use the Post2All CLI to create, schedule, inspect, update, cancel, and delete social media posts across connected accounts.
---

# Post2All CLI

Use the `post2all` command-line interface to manage a user's Post2All workspace. Run commands on the user's behalf when they ask to create, schedule, or manage social posts.

## Install And Authenticate

Check whether the CLI is available:

```bash
post2all --help
```

If it is not installed, use it without a global install:

```bash
npx @post2all/cli <command>
```

The examples below use `post2all`. Substitute `npx @post2all/cli` when needed.

Before performing workspace operations, validate authentication:

```bash
post2all config whoami --json
```

Authentication is resolved in this order:

1. The global `--api-key <key>` option.
2. The `POST2ALL_API_KEY` environment variable.
3. The local config created by `post2all config set-key <apiKey>`.

Never print, log, commit, or repeat an API key. If authentication is missing or expired, direct the user to create a key in Post2All and configure it themselves. Do not ask them to paste a key into chat.

## Core Workflow

1. Run `post2all config whoami --json` to verify access.
2. Run `post2all accounts --json` and use returned account IDs. Never guess IDs.
3. Check each selected account's `supportedPostTypes` before choosing `text`, `image`, or `video`.
4. Clarify content, accounts, post type, status, media, and scheduling timezone when they are not safely inferable.
5. Prefer `--json` for commands whose output will be parsed or used by later commands.
6. Run the command and report the post ID, resulting status, target accounts, and scheduled time.

Use a draft for review-oriented requests. Use `scheduled` only with an explicit future time. Use `publish_now` only when the user clearly requests immediate publication. Note that omitting `--status` from `post create` publishes immediately.

## Discover Accounts And Posts

List connected accounts:

```bash
post2all accounts --json
```

List posts, optionally filtering by status or type:

```bash
post2all posts --status scheduled --limit 100 --json
post2all posts --type image --page 2 --json
```

Valid status filters are `draft`, `scheduled`, `published`, `partially_failed`, and `failed`. Valid types are `text`, `image`, and `video`.

Fetch one post before changing or deleting it:

```bash
post2all post get <postId> --json
```

## Create Posts

Create a draft:

```bash
post2all post create \
  --type text \
  --accounts acc_1,acc_2 \
  --content "Draft content" \
  --status draft \
  --json
```

Schedule a post:

```bash
post2all post create \
  --type text \
  --accounts acc_1,acc_2 \
  --content "Scheduled content" \
  --status scheduled \
  --scheduled-at "2026-06-21T09:00:00+05:30" \
  --json
```

Publish immediately only when explicitly requested:

```bash
post2all post create \
  --type text \
  --accounts acc_1 \
  --content "Publish now" \
  --status publish_now \
  --json
```

### Media

Upload every local file before creating a post. Post creation accepts media IDs only.

```bash
post2all media upload ./photo1.jpg ./photo2.png --json
post2all post create --type image --accounts acc_1 --media-ids <mediaId1>,<mediaId2> --status draft --json
```

```bash
post2all post create \
  --type image \
  --accounts acc_1 \
  --content "Photo gallery" \
  --media-ids <mediaId1>,<mediaId2> \
  --status draft \
  --json
```

Use `--type video` for video posts. Confirm files exist before executing. Supported image formats are jpg, jpeg, png, gif, webp, and svg; supported video formats are mp4, webm, mov, avi, and mkv.

### Platform-Specific Settings

Use `--platform-settings` for platform-specific captions or fields. Quote JSON with single quotes in shells that support it:

```bash
post2all post create \
  --type text \
  --accounts acc_twitter,acc_threads \
  --content "Main caption" \
  --platform-settings '{"threads":{"caption":"Short Threads caption"}}' \
  --status draft \
  --json
```

Settings are keyed by platform, not account ID. Do not invent platform fields. Do not shorten every platform to the lowest character limit. Keep the main caption appropriate for platforms that support it, and add a shorter `caption` override only for constrained platforms such as X or Threads.

## Manage Posts

Only draft and scheduled posts can be updated. Only supplied fields are changed:

```bash
post2all post update <postId> --content "Revised content" --json
post2all post update <postId> \
  --status scheduled \
  --scheduled-at "2026-06-22T10:00:00+05:30" \
  --json
```

Move a scheduled post back to draft:

```bash
post2all post cancel <postId> --json
```

Toggle between draft and scheduled:

```bash
post2all post status <postId> --status draft --json
post2all post status <postId> --status scheduled --json
```

Permanently delete a post and cancel any pending schedule:

```bash
post2all post delete <postId> --json
```

Before a destructive delete, fetch the post and obtain confirmation unless the user already clearly identified the post and asked for deletion. Prefer `cancel` when the intent is only to stop publication while preserving the content.

## Scheduling Rules

- Always pass scheduled times as ISO 8601 with `Z` or an explicit UTC offset.
- Interpret a local time in the user's known timezone. Ask for the timezone if it cannot be determined reliably.
- Confirm the resolved timestamp when relative language such as "tomorrow morning" is ambiguous.
- Do not schedule a post in the past.
- `--scheduled-at` is required when creating or updating a post to `scheduled`.

## Errors And Recovery

- `INVALID_API_KEY` or `EXPIRED_API_KEY`: have the user configure a valid key, then retry `config whoami`.
- `INVALID_ACCOUNTS`: refresh IDs with `accounts --json`.
- `POST_NOT_FOUND`: refresh with `posts --json` and verify the ID.
- `UNSUPPORTED_MEDIA`: check the file type and selected accounts' supported post types.
- `RATE_LIMITED`: wait before retrying; do not loop aggressively.
- `PLAN_UPGRADE_REQUIRED` or `FORBIDDEN`: explain the account or plan restriction instead of repeatedly retrying.
- `INVALID_REQUEST`: inspect `post2all <command> --help` and correct the supplied flags.

Use `--base-url <url>` only when the user explicitly works with a non-default Post2All deployment.
