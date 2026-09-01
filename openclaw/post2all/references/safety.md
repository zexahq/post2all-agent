# Publishing boundaries and safety

post2all can create real public side effects. Keep the boundary between the OpenClaw agent loop and the publishing layer explicit.

## Default behavior

Use drafts by default.

A draft-first workflow lets OpenClaw research, remember context, generate variants, and prepare content without immediately exposing it to the public internet.

The user can then review the queue periodically and schedule or publish approved work.

## Scheduling

Schedule only when one of these is true:

- the user explicitly approved the content, destinations, and time; or
- the user intentionally configured a narrow trusted routine that is allowed to schedule without reviewing every individual item.

Resolve relative times in the user's timezone and use a timezone-aware timestamp.

## Immediate publishing

Immediate publishing is a consequential external action.

Before publishing immediately:

1. verify the target accounts;
2. verify required current platform settings;
3. check media and account capabilities;
4. make sure the user clearly requested immediate publication;
5. do not silently substitute a different account, platform, privacy choice, board, channel, or delivery mode.

## Destructive actions

Cancellation and deletion should be handled deliberately.

- Inspect the post before destructive actions when the target is ambiguous.
- For live social deletion, only offer deletion when the current server state reports `deletion.available: true` for that exact target.
- Show the account/platform being affected and obtain confirmation.
- Explain `deletion.reason` when live deletion is unavailable.
- Deleting the post2all record is separate from deleting a live social post.

## Credentials

Never:

- print or echo a post2all API key;
- ask the user to paste an API key into chat;
- commit a key to a repository;
- store a key in generated social content or logs.

Prefer the local post2all CLI configuration or the hosted MCP OAuth flow.

## Platform-specific approvals

Some platforms require additional choices or confirmations. TikTok is the clearest example.

Never invent or silently default creator-specific privacy, commercial-disclosure, interaction, or policy confirmations. Load fresh publishing options and obtain the required user confirmation.

## Retries

Do not blindly retry publishing mutations.

When a request fails:

- inspect the returned post state or error;
- determine whether the previous attempt created or changed a post;
- reuse/update the existing post when appropriate rather than creating duplicates;
- pause on rate limits;
- surface permission or plan restrictions instead of looping.

## Good unattended boundary

A practical unattended setup is:

```text
OpenClaw loop
  → research / memory / ideas / writing
  → post2all drafts
  → daily human review
  → approved schedule / publish
```

This preserves most of the automation while keeping the final publishing handoff visible and auditable.
