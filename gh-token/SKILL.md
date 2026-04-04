---
name: gh-token
description: Use when the user wants to create a GitHub fine-grained personal access token with minimal permissions for a specific task — determines minimal permissions and generates a prefill URL for the GitHub web UI
argument-hint: <task> [--org <org>] [--repo <owner/repo>]
allowed-tools:
  - Bash
  - Read
  - AskUserQuestion
---

Create a fine-grained GitHub personal access token scoped to exactly what the given task needs — no more.

**Note:** GitHub does not expose an API for creating fine-grained PATs. Token creation requires the web UI. Your job is to determine the minimal permissions, provide URL to go, and give the user exact settings to copy.

## How to run

The user invokes this as `/gh-token <task>` where `<task>` describes what the token will be used for. Examples:
- `/gh-token ecosystem-ci for quarkiverse/quarkus-solr`
- `/gh-token read-only clone of my org repos`
- `/gh-token publish maven packages to quarkiverse`
- `/gh-token trigger workflows in quarkiverse/quarkus-solr`

If no task is given, ask the user what the token is for before proceeding.

## Your process

### 1. Identify the task and map it to minimal permissions

Match the described task to one of the known patterns below, or reason from first principles if it's novel. Always start from zero permissions and add only what's required.

**Known task patterns and their minimal permission sets:**

| Task | Resource | Permission |
|------|----------|------------|
| Read-only checkout / clone | Contents | Read |
| Push commits / create releases | Contents | Write |
| Trigger `workflow_dispatch` | Actions | Write |
| Cancel / re-run workflows | Actions | Write |
| Read workflow run logs | Actions | Read |
| Publish to GitHub Packages / GHCR | Packages | Write |
| Read packages | Packages | Read |
| Dependabot alerts | Vulnerability alerts | Read |
| Create/close issues | Issues | Write |
| Comment on PRs | Pull requests | Write |
| Merge PRs | Pull requests | Write |
| Read org members | Members | Read |
| Manage webhooks | Webhooks | Write |
| Read org audit log | Administration | Read (org-level) |
| Read GitHub Projects v2 | Projects | Read (org-level) |
| Update GitHub Projects v2 field values | Projects | Read and write (org-level) |
| Read issues / issue labels | Issues | Read |
| Add/change issue labels | Issues | Read and write |

> **Projects v2 note:** The Projects permission is an **Organization permission**, not a repository permission. It must be set under "Organization permissions → Projects", not under repository permissions. Repository access still needs to be set (to the relevant repos) for reading issue data alongside project data.

**Default: if unsure, use Read. Never use Write unless the task requires mutating state.**

### 2. Determine scope (repo vs org)

- Prefer **single-repo** scope whenever the task is repo-specific.
- Use **org-wide** only when the task genuinely spans multiple repos (e.g. a bot that comments across all org repos).
- Ask the user if scope is ambiguous.

### 3. Determine expiration

Pick the shortest sensible window for the task:

| Use case | Default expiry | Rationale |
|----------|---------------|-----------|
| One-off / local dev task | **7 days** | Throw away after use |
| Short-lived CI experiment | **30 days** | Covers a decent dev cycle |
| Stable CI secret (e.g. ECOSYSTEM_CI_TOKEN) | **90 days** | Forces regular rotation |

Never suggest or accept more than **90 days**. If the user asks for longer, push back: token compromise windows scale directly with expiry. Shorter is always safer — `gh secret set` makes rotation a one-liner so there's no practical reason to go long.

### 4. Generate a prefill URL and show exact settings

Construct a GitHub URL with the token name pre-filled:

```
https://github.com/settings/personal-access-tokens/new?name=<token-name>
```

Then display a clear settings block the user can copy from — GitHub's form does not support prefilling permissions or repo scope via URL, so those must be set manually:

---
**Click to open:** `https://github.com/settings/personal-access-tokens/new?name=<token-name>`

**Fill in these settings:**

| Field | Value |
|-------|-------|
| Token name | `<token-name>` (pre-filled) |
| Expiration | `<N> days` |
| Resource owner | `<owner>` |
| Repository access | Only selected → `<owner/repo>` |
| **Permissions** | |
|   <resource> | Read / Read and write |

*(All other permissions stay "No access")*

---

The token name should be descriptive and kebab-case, e.g. `ecosystem-ci-quarkiverse-quarkus-solr` or `issues-comment-maxandersen-skills`.

### 5. After the user has the token — show how to verify and store it

To verify what permissions were actually granted, direct the user to:

```
https://github.com/settings/personal-access-tokens
```

Click the token name to see its exact permission set and expiry. There is no reliable CLI command to inspect fine-grained PAT permissions — the introspection endpoints require a fine-grained token themselves, and `X-Accepted-Github-Permissions` response headers only report the minimum required for the specific endpoint called, not the full token permission set.

Then show how to store it:

```bash
# Set as a repo secret
gh secret set <SECRET_NAME> --repo <owner/repo> --body "<token>"

# Set as an org secret (available to specific repos)
gh secret set <SECRET_NAME> --org <org> --repos "<repo1>,<repo2>" --body "<token>"
```

Remind the user to copy the token immediately — GitHub only shows it once. Warn them **never to paste the token value into a conversation or chat** — treat any exposed token as compromised and revoke it immediately via:

```bash
# Find the token ID
gh api /user/personal-access-tokens | jq '.[] | select(.name=="<token-name>") | .id'
# Revoke it
gh api --method DELETE /user/personal-access-tokens/<id>
```

### 6. Warn on anti-patterns

Flag these if the user's request would trigger them:
- **Classic PAT instead of fine-grained**: If the user mentions creating a classic PAT (`github.com/settings/tokens/new` without `/fine-grained`), warn them that classic PATs cannot be scoped to a single repo and push them to use fine-grained instead. Fine-grained tokens are strictly safer.
- Requesting `write` on `contents` + `actions` + `secrets` together (near-full repo access — suggest splitting into separate tokens)
- Token expiry > 90 days
- Org-wide scope when a single repo would suffice
- Storing the token in a file or env var in plaintext (suggest `gh secret set` or a secrets manager)

### 7. Suggest the matching `permissions:` block for the workflow

If the task is GitHub Actions CI-related, also output the minimal `permissions:` block to add to the workflow YAML so the auto-issued `GITHUB_TOKEN` is also locked down:

```yaml
permissions:
  contents: read        # example — adjust to task
```

Remind the user that `permissions:` in the workflow overrides the org default, so this works regardless of org settings.

