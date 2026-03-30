---
name: zernio-cli
description: Use when user wants to post to social media (Twitter, LinkedIn, Instagram, TikTok, YouTube, Bluesky, etc.), schedule posts across platforms, or manage social media content via Zernio
---

# Zernio CLI

## Overview

Zernio CLI schedules and manages social media posts across 14 platforms from the terminal. Run via `npx @zernio/cli` - no global install needed.

## Safety Guidelines

**CRITICAL: Always ask user permission before posting.**

1. **Prepare the command** - show user exactly what will be posted
2. **Get explicit approval** - "Should I run this command to post?"
3. **Only then execute** - after user confirms

**Never post without asking first**, even if the user seems to request it directly.

**Default to draft mode** - use `--draft` flag unless user explicitly wants to publish immediately.

## When to Use

- User wants to post to social media platforms
- User mentions scheduling posts for later
- User wants to post to multiple platforms at once
- User has social media accounts to manage

**When NOT to use:**
- Direct API integration (use Zernio API skill instead)
- One-off manual posting (CLI is for automation/scheduling)

## Quick Reference

| Task | Command |
|------|---------|
| **Check auth** | `npx @zernio/cli auth:check` |
| **Login (browser)** | `npx @zernio/cli auth:login` |
| **List accounts** | `npx @zernio/cli accounts:list` |
| **Create draft** | `npx @zernio/cli posts:create --text "..." --accounts id1,id2 --draft` |
| **Publish post** | `npx @zernio/cli posts:create --text "..." --accounts id1,id2` (ASK FIRST) |
| **Schedule post** | Add `--scheduledAt "2025-06-01T09:00:00Z"` (ASK FIRST) |
| **Upload media** | `npx @zernio/cli media:upload <file>` |
| **List posts** | `npx @zernio/cli posts:list` |

## Authentication

**ALWAYS check auth first:**
```bash
npx @zernio/cli auth:check
```

**If not authenticated:**
```bash
# Browser login (recommended - creates API key automatically)
npx @zernio/cli auth:login

# Or manual key (get from zernio.com/dashboard/api-keys)
npx @zernio/cli auth:set --key "sk_your-api-key"
```

Config stored in `~/.zernio/config.json`

## Common Workflows

### Simple Post
```bash
# 1. Get account IDs
npx @zernio/cli accounts:list

# 2. Create draft first (ALWAYS start with draft)
npx @zernio/cli posts:create \
  --text "Your message here" \
  --accounts account-id-1,account-id-2 \
  --draft

# 3. Show command to user, ASK permission, then publish
# Only run after explicit user approval:
npx @zernio/cli posts:create \
  --text "Your message here" \
  --accounts account-id-1,account-id-2
```

### Scheduled Post
```bash
# Scheduled posts are safer (user can cancel before publish time)
# Still ASK permission before running this command
npx @zernio/cli posts:create \
  --text "Your message" \
  --accounts account-id \
  --scheduledAt "2025-06-01T09:00:00Z"
```

### Post with Media
```bash
# 1. Upload media FIRST
npx @zernio/cli media:upload ~/path/to/image.png
# Returns: {"url": "https://..."}

# 2. Create draft with media URL
npx @zernio/cli posts:create \
  --text "Your message" \
  --accounts account-id \
  --media "https://..." \
  --draft

# 3. ASK permission, show user the draft, then publish
# Only run after explicit approval (remove --draft flag):
npx @zernio/cli posts:create \
  --text "Your message" \
  --accounts account-id \
  --media "https://..."
```

**Multiple images:** Use comma-separated URLs: `--media "url1,url2,url3"`

## Supported Platforms

Instagram, TikTok, X (Twitter), LinkedIn, Facebook, Threads, YouTube, Bluesky, Pinterest, Reddit, Snapchat, Telegram, Google Business Profile

**Platform requirements:**
- Text-only: All platforms
- Media required: Instagram, TikTok (images/video)
- Video required: YouTube, TikTok

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Command `zernio` not found | Use `npx @zernio/cli` (note the @) |
| "Account not found" | Run `accounts:list` to get correct IDs |
| Media not showing | Upload media first, then use returned URL |
| Post fails on some platforms | Check platform requirements (some need media) |
| Date format wrong | Use ISO 8601: `2025-06-01T09:00:00Z` |
| `Unknown argument: status` | Use `--draft` flag, NOT `--status draft` |
| **Posted without asking** | **STOP. Always ask permission before posting** |

## Links

- [Full CLI Docs](https://docs.zernio.com/cli)
- [GitHub](https://github.com/zernio-dev/zernio-cli)
- [npm Package](https://www.npmjs.com/package/@zernio/cli)
