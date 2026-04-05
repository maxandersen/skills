---
name: jgccli
description: Google Calendar CLI (Java) for listing calendars, viewing/creating/updating events, and checking availability.
---

# Google Calendar CLI (Java)

Command-line interface for Google Calendar operations. Java port of [gccli](https://github.com/badlogic/pi-skills/tree/main/gccli).

## Installation

**Option 1: Run directly with JBang (no install)**
```bash
jbang jgccli@maxandersen/skills --help
```

**Option 2: Install as command**
```bash
jbang app install jgccli@maxandersen/skills
jgccli --help
```

## Setup

### Google Cloud Console (one-time)

1. [Create a new project](https://console.cloud.google.com/projectcreate) (or select existing)
2. [Enable the Google Calendar API](https://console.cloud.google.com/apis/api/calendar-json.googleapis.com)
3. [Set app name](https://console.cloud.google.com/auth/branding) in OAuth branding
4. [Add test users](https://console.cloud.google.com/auth/audience) (all Gmail addresses you want to use)
5. [Create OAuth client](https://console.cloud.google.com/auth/clients):
   - Click "Create Client"
   - Application type: "Desktop app"
   - Download the JSON file

### Configure jgccli

First check if already configured:
```bash
jgccli accounts list
```

If no accounts, guide the user through setup:
1. Ask if they have a Google Cloud project with Calendar API enabled
2. If not, walk them through the Google Cloud Console steps above
3. Have them download the OAuth credentials JSON
4. Run: `jgccli accounts credentials ~/path/to/credentials.json`
5. Run: `jgccli accounts add <email>` (use `--manual` for browserless OAuth)

## Usage

Run `jgccli --help` for full command reference.

Common operations:
- `jgccli calendars <email>` - List all calendars
- `jgccli events <email> <calendarId> [--from <dt>] [--to <dt>]` - List events
- `jgccli event <email> <calendarId> <eventId>` - Get event details
- `jgccli create <email> <calendarId> --summary <s> --start <dt> --end <dt>` - Create event
- `jgccli freebusy <email> <calendarIds> --from <dt> --to <dt>` - Check availability

Use `primary` as calendarId for the main calendar.

## Date/Time Format

- Timed events: `YYYY-MM-DDTHH:MM:SSZ` (UTC) or `YYYY-MM-DDTHH:MM:SS` (local)
- All-day events: `YYYY-MM-DD` with `--all-day` flag

## Data Storage

- `~/.jgccli/credentials.json` - OAuth client credentials
- `~/.jgccli/accounts.json` - Account tokens
