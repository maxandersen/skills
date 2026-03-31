---
name: using-maven-worktrees
description: Use when creating git worktrees for Maven projects - prevents SNAPSHOT artifact clashes and corporate plugin resolution failures through isolated local repositories
---

# Using Maven Worktrees

## Overview

Git worktrees for Maven projects require isolated local repositories to prevent concurrent SNAPSHOT builds from clashing.

**Core principle:** maven.repo.local (isolation) + maven.repo.local.tail (shared cache) = safe parallel builds.

**Announce at start:** "I'm using the using-maven-worktrees skill to set up an isolated Maven workspace."

## The Problem

**Without isolation:** Multiple worktrees building the same SNAPSHOT version write to the same `~/.m2/repository/` path, causing:
- Race conditions during concurrent builds
- Artifacts from one feature contaminating another
- Build failures from incomplete artifact writes

**Solution:** Each worktree gets its own maven.repo.local, with maven.repo.local.tail pointing to shared repository for dependencies.

## Directory Selection Process

Follow the same priority as git worktrees:

### 1. Check Existing Directories

```bash
# Check in priority order
ls -d .worktrees 2>/dev/null     # Preferred (hidden)
ls -d worktrees 2>/dev/null      # Alternative
```

**If found:** Use that directory. If both exist, `.worktrees` wins.

### 2. Check CLAUDE.md

```bash
grep -i "worktree.*director" CLAUDE.md 2>/dev/null
```

**If preference specified:** Use it without asking.

### 3. Ask User

If no directory exists and no CLAUDE.md preference:

```
No worktree directory found. Where should I create worktrees?

1. .worktrees/ (project-local, hidden)
2. ~/.config/superpowers/worktrees/<project-name>/ (global location)

Which would you prefer?
```

## Safety Verification

### For Project-Local Directories (.worktrees or worktrees)

**MUST verify directory is ignored before creating worktree:**

```bash
# Check if directory is ignored
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

**If NOT ignored:**

1. Add appropriate line to .gitignore
2. Commit the change
3. Proceed with worktree creation

**Why critical:** Prevents accidentally committing worktree contents to repository.

### For Global Directory (~/.config/superpowers/worktrees)

No .gitignore verification needed - outside project entirely.

## Maven Isolation Setup

**CRITICAL: Both properties required, not just maven.repo.local.**

### 1. Create Worktree

```bash
# Determine full path (same as git worktrees)
project=$(basename "$(git rev-parse --show-toplevel)")

case $LOCATION in
  .worktrees|worktrees)
    path="$LOCATION/$BRANCH_NAME"
    ;;
  ~/.config/superpowers/worktrees/*)
    path="~/.config/superpowers/worktrees/$project/$BRANCH_NAME"
    ;;
esac

# Create worktree
git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```

### 2. Configure Maven Isolation

**Create `.mvn/maven.config` with BOTH properties:**

```bash
mkdir -p .mvn

cat > .mvn/maven.config << 'EOF'
-Dmaven.repo.local=${session.executionRootDirectory}/.m2/repository
-Dmaven.repo.local.tail=${user.home}/.m2/repository
EOF
```

**Why both are required:**

- `maven.repo.local`: Isolates SNAPSHOT installs (prevents clash)
- `maven.repo.local.tail`: Points to shared repository for dependencies (prevents re-downloads)

**Without tail:** Maven downloads entire repository fresh, fails to resolve corporate plugins.

### 3. Verify Configuration

```bash
# Check maven.config exists and has both properties
cat .mvn/maven.config

# Expected output:
# -Dmaven.repo.local=${session.executionRootDirectory}/.m2/repository
# -Dmaven.repo.local.tail=${user.home}/.m2/repository
```

## Maven Command Selection

**Use Maven wrapper if available, otherwise maven:**

```bash
# Check for wrapper
if [ -f ./mvnw ]; then
  MVN_CMD="./mvnw"
elif [ -f ../mvnw ]; then
  MVN_CMD="../mvnw"
else
  MVN_CMD="mvn"
fi
```

## Baseline Verification

**Use `mvn verify` NOT `mvn install`:**

```bash
$MVN_CMD verify
```

**Why verify not install:**
- `mvn install` pollutes local repository (defeats isolation purpose)
- `mvn verify` runs all tests including integration tests
- `mvn test` might skip integration tests

**If tests fail:** Report failures, ask whether to proceed or investigate.

**If tests pass:** Report ready.

## IDE Files

**IDE project files should be local to each worktree:**

- IntelliJ: `.idea/`, `*.iml` - generated per worktree
- Eclipse: `.project`, `.classpath`, `.settings/` - generated per worktree
- VS Code: `.vscode/` - can be shared via symlink if settings are branch-agnostic

**Do NOT symlink `.idea/` or Eclipse files** - different branches may need different:
- Compiler settings
- Source roots
- Module structure

## Quick Reference

| Situation | Action |
|-----------|--------|
| `.worktrees/` exists | Use it (verify ignored) |
| `worktrees/` exists | Use it (verify ignored) |
| Both exist | Use `.worktrees/` |
| Neither exists | Check CLAUDE.md → Ask user |
| Directory not ignored | Add to .gitignore + commit |
| Tests fail during baseline | Report failures + ask |
| mvnw exists | Use `./mvnw` over `mvn` |
| Baseline verification | Use `mvn verify` not `mvn install` |
| IDE files | Generate per worktree, don't share |

## Maven Config Verification Checklist

Before reporting "ready", verify:

- [ ] `.mvn/maven.config` exists
- [ ] Contains `-Dmaven.repo.local=${session.executionRootDirectory}/.m2/repository`
- [ ] Contains `-Dmaven.repo.local.tail=${user.home}/.m2/repository`
- [ ] Both properties on separate lines
- [ ] No typos in property names

## Common Mistakes

### Using shared `~/.m2/repository`

**Problem:** Multiple worktrees installing same SNAPSHOT version clash
**Fix:** Always create `.mvn/maven.config` with maven.repo.local

### Setting maven.repo.local without maven.repo.local.tail

**Problem:**
- Maven downloads entire repository fresh (slow)
- Corporate/internal plugins fail to resolve
- Defeats purpose of local repository cache

**Fix:** Always set BOTH properties

### Using `mvn install` for baseline verification

**Problem:** Pollutes local repository, defeats isolation
**Fix:** Use `mvn verify` - runs all tests without installing

### Sharing IDE project files between worktrees

**Problem:**
- Different branches may have different source structure
- IDE confusion when switching between worktrees
- Conflicting compiler/module settings

**Fix:** Generate IDE files per worktree, add to .gitignore

### Assuming default Maven config works

**Problem:** No isolation, SNAPSHOT artifacts clash
**Fix:** Always explicitly configure maven.repo.local + tail

## Rationalization Table

| Excuse | Reality |
|--------|---------|
| "Default Maven config is fine" | Concurrent SNAPSHOT installs will clash without isolation |
| "maven.repo.local alone is enough" | Without tail, downloads everything fresh and plugins fail |
| "Shared repository is normal" | Normal for single workspace, broken for parallel worktrees |
| "No time for advanced setup" | Setup takes 30 seconds, debugging clashes takes hours |
| "This is complete isolation" | Complete = maven.repo.local + maven.repo.local.tail |
| "mvn install verifies properly" | mvn install pollutes repo, use mvn verify |
| "IDE files should be consistent" | IDE files reflect code structure, which varies by branch |

## Red Flags - STOP and Fix

**Never:**
- Create Maven worktree without `.mvn/maven.config`
- Set maven.repo.local without maven.repo.local.tail
- Use `mvn install` for baseline verification
- Symlink `.idea/` or Eclipse project files
- Skip maven.config verification before reporting ready
- Assume default Maven behavior works for worktrees

**Always:**
- Configure BOTH maven.repo.local AND maven.repo.local.tail
- Use `mvn verify` for baseline
- Generate IDE files per worktree
- Verify maven.config exists and is correct

## Example Workflow

```
You: I'm using the using-maven-worktrees skill to set up an isolated Maven workspace.

[Check .worktrees/ - exists]
[Verify ignored - git check-ignore confirms]
[Create worktree: git worktree add .worktrees/feature-auth -b feature/auth]
[cd .worktrees/feature-auth]
[Create .mvn/maven.config with maven.repo.local + maven.repo.local.tail]
[Verify config exists and has both properties]
[Run ./mvnw verify - 127 tests passing]

Worktree ready at /Users/max/myproject/.worktrees/feature-auth
Maven isolation configured:
  - maven.repo.local: .m2/repository (worktree-local)
  - maven.repo.local.tail: ~/.m2/repository (shared cache)
Tests passing (127 tests, 0 failures)
Ready to implement authentication feature
```

## Real-World Impact

**Problem solved:** Prevents hours of debugging why "it works in my worktree but fails in yours" when both are building same SNAPSHOT version.

**Performance:** maven.repo.local.tail prevents re-downloading 500MB+ of dependencies per worktree.

**Corporate environments:** Ensures internal/corporate Maven plugins resolve correctly via tail repository.
