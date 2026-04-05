---
name: jgdcli
description: Google Drive CLI (Java) for listing, searching, uploading, downloading, and sharing files and folders.
---

# Google Drive CLI (Java)

Command-line interface for Google Drive operations. Java port of [gdcli](https://github.com/badlogic/pi-skills/tree/main/gdcli).

## Installation

**Option 1: Run directly with JBang (no install)**
```bash
jbang jgdcli@maxandersen/skills --help
```

**Option 2: Install as command**
```bash
jbang app install jgdcli@maxandersen/skills
jgdcli --help
```

## Setup

### Google Cloud Console (one-time)

1. [Create a new project](https://console.cloud.google.com/projectcreate) (or select existing)
2. [Enable the Google Drive API](https://console.cloud.google.com/apis/api/drive.googleapis.com)
3. [Set app name](https://console.cloud.google.com/auth/branding) in OAuth branding
4. [Add test users](https://console.cloud.google.com/auth/audience) (all Gmail addresses you want to use)
5. [Create OAuth client](https://console.cloud.google.com/auth/clients):
   - Click "Create Client"
   - Application type: "Desktop app"
   - Download the JSON file

### Configure jgdcli

First check if already configured:
```bash
jgdcli accounts list
```

If no accounts, guide the user through setup:
1. Ask if they have a Google Cloud project with Drive API enabled
2. If not, walk them through the Google Cloud Console steps above
3. Have them download the OAuth credentials JSON
4. Run: `jgdcli accounts credentials ~/path/to/credentials.json`
5. Run: `jgdcli accounts add <email>` (use `--manual` for browserless OAuth)

## Usage

Run `jgdcli --help` for full command reference.

Common operations:
- `jgdcli ls <email> [folderId]` - List files/folders
- `jgdcli ls <email> --query "<query>"` - List with Drive query filter
- `jgdcli search <email> "<text>"` - Full-text content search
- `jgdcli download <email> <fileId> [destPath]` - Download a file
- `jgdcli upload <email> <localPath> [--folder <folderId>]` - Upload a file
- `jgdcli mkdir <email> <name>` - Create a folder
- `jgdcli share <email> <fileId> --anyone` - Share publicly

## Search

**Two different commands:**
- `search "<text>"` - Searches inside file contents (fullText)
- `ls --query "<query>"` - Filters by metadata (name, type, date, etc.)

**Use `ls --query` for filename searches!**

## Query Syntax (for ls --query)

Format: `field operator value`. Combine with `and`/`or`, group with `()`.

**Operators:** `=`, `!=`, `contains`, `<`, `>`, `<=`, `>=`

**Examples:**
```bash
# By filename
jgdcli ls <email> --query "name = 'report.pdf'"           # exact match
jgdcli ls <email> --query "name contains 'IMG'"           # prefix match

# By type
jgdcli ls <email> --query "mimeType = 'application/pdf'"
jgdcli ls <email> --query "mimeType contains 'image/'"
jgdcli ls <email> --query "mimeType = 'application/vnd.google-apps.folder'"  # folders

# By date
jgdcli ls <email> --query "modifiedTime > '2024-01-01'"

# By owner/sharing
jgdcli ls <email> --query "'me' in owners"
jgdcli ls <email> --query "sharedWithMe"

# Exclude trash
jgdcli ls <email> --query "trashed = false"

# Combined
jgdcli ls <email> --query "name contains 'report' and mimeType = 'application/pdf'"
```

Ref: https://developers.google.com/drive/api/guides/ref-search-terms

## Data Storage

- `~/.jgdcli/credentials.json` - OAuth client credentials
- `~/.jgdcli/accounts.json` - Account tokens
- `~/.jgdcli/downloads/` - Default download location
