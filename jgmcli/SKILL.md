---
name: jgmcli
description: Gmail CLI (Java) for searching emails, reading threads, sending messages, managing drafts, and handling labels/attachments.
---

# Gmail CLI (Java)

Command-line interface for Gmail operations. Java port of [gmcli](https://github.com/badlogic/pi-skills/tree/main/gmcli).

## Installation

**Option 1: Run directly with JBang (no install)**
```bash
jbang jgmcli@maxandersen/skills --help
```

**Option 2: Install as command**
```bash
jbang app install jgmcli@maxandersen/skills
jgmcli --help
```

## Setup

### Google Cloud Console (one-time)

1. [Create a new project](https://console.cloud.google.com/projectcreate) (or select existing)
2. [Enable the Gmail API](https://console.cloud.google.com/apis/api/gmail.googleapis.com)
3. [Set app name](https://console.cloud.google.com/auth/branding) in OAuth branding
4. [Add test users](https://console.cloud.google.com/auth/audience) (all Gmail addresses you want to use)
5. [Create OAuth client](https://console.cloud.google.com/auth/clients):
   - Click "Create Client"
   - Application type: "Desktop app"
   - Download the JSON file

### Configure jgmcli

First check if already configured:
```bash
jgmcli accounts list
```

If no accounts, guide the user through setup:
1. Ask if they have a Google Cloud project with Gmail API enabled
2. If not, walk them through the Google Cloud Console steps above
3. Have them download the OAuth credentials JSON
4. Run: `jgmcli accounts credentials ~/path/to/credentials.json`
5. Run: `jgmcli accounts add <email>` (use `--manual` for browserless OAuth)

## Usage

Run `jgmcli --help` for full command reference.

Common operations:
- `jgmcli search <email> "<query>"` - Search emails using Gmail query syntax
- `jgmcli thread <email> <threadId>` - Read a thread with all messages
- `jgmcli send <email> --to <emails> --subject <s> --body <b>` - Send email
- `jgmcli labels <email> list` - List all labels
- `jgmcli drafts <email> list` - List drafts

## Gmail Query Syntax

```
in:inbox, in:sent, in:drafts, in:trash
is:unread, is:starred, is:important
from:sender@example.com, to:recipient@example.com
subject:keyword, has:attachment, filename:pdf
after:2024/01/01, before:2024/12/31
label:Work, label:UNREAD
Combine: "in:inbox is:unread from:boss@company.com"
```

## Data Storage

- `~/.jgmcli/credentials.json` - OAuth client credentials
- `~/.jgmcli/accounts.json` - Account tokens
- `~/.jgmcli/attachments/` - Downloaded attachments
