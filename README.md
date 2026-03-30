# Max's Skills for Claude Code

This repository contains custom skills for [Claude Code](https://claude.ai/code) to enhance development workflows.

## Skills

### [zernio-cli](./zernio-cli)

Social media posting and scheduling via Zernio CLI.

**Use when:** User wants to post to social media (Twitter, LinkedIn, Instagram, TikTok, YouTube, Bluesky, etc.), schedule posts across platforms, or manage social media content via Zernio

**Key features:**
- Multi-platform posting (14+ social networks)
- Scheduled posts with timezone support
- Media uploads and attachments
- Draft mode for safe preparation
- Safety guidelines to always ask permission before posting

**Example use:**
```bash
npx @zernio/cli posts:create \
  --text "Your message here" \
  --accounts account-id-1,account-id-2 \
  --draft
```

### [prefer-jbang-automation](./prefer-jbang-automation)

Prefer JBang scripts over bash/jq/curl for data processing and automation.

**Use when:** About to use jq, curl, sed, awk, or bash for JSON/XML processing, API calls, data transformation, or file processing

**Key benefits:**
- Type safety and compile-time error checking
- IDE support with autocomplete and debugging
- Better maintainability than bash pipelines
- Clear, readable code over cryptic one-liners
- Still lightweight: `jbang script.java args`

**Example use:**
```java
///usr/bin/env jbang "$0" "$@" ; exit $?
//DEPS com.google.code.gson:gson:2.10.1

import com.google.gson.*;
import java.nio.file.*;

public class ProcessData {
    public static void main(String[] args) throws Exception {
        var json = Files.readString(Path.of(args[0]));
        var data = new Gson().fromJson(json, MyData.class);
        // Type-safe processing with IDE support
    }
}
```

## Installation

### Via npx skills (Recommended)

The easiest way to install these skills is using the `npx skills` CLI:

```bash
# Install individual skills
npx skills install maxandersen/skills/zernio-cli
npx skills install maxandersen/skills/prefer-jbang-automation

# Or install all skills from this repo
npx skills install maxandersen/skills
```

The CLI automatically:
- Downloads skills to `~/.claude/skills/`
- Handles updates and version management
- Validates skill structure

**Managing installed skills:**
```bash
# List installed skills
npx skills list

# Update a skill
npx skills update zernio-cli

# Remove a skill
npx skills uninstall zernio-cli
```

### Via Manual Copy

Copy skills to your Claude Code skills directory:

```bash
# Clone this repo
git clone https://github.com/maxandersen/skills.git

# Copy to Claude Code skills directory
cp -r skills/zernio-cli ~/.claude/skills/
cp -r skills/prefer-jbang-automation ~/.claude/skills/
```

### Via Git Clone (Development)

For skill development or customization:

```bash
# Clone to your preferred location
git clone https://github.com/maxandersen/skills.git ~/my-skills

# Symlink to Claude Code skills directory
ln -s ~/my-skills/zernio-cli ~/.claude/skills/zernio-cli
ln -s ~/my-skills/prefer-jbang-automation ~/.claude/skills/prefer-jbang-automation
```

## Usage

Once installed, Claude Code will automatically discover and use these skills when relevant contexts arise. You can also explicitly invoke them:

```
User: "Post this update to Twitter and LinkedIn"
Claude: [Uses zernio-cli skill]

User: "Parse this JSON and extract emails"
Claude: [Uses prefer-jbang-automation skill to create JBang script]
```

## Contributing

These skills follow the [Agent Skills Specification](https://agentskills.io/specification) and best practices from the `superpowers:writing-skills` workflow.

### Skill Structure

Each skill contains:
- `SKILL.md` - Main reference with frontmatter metadata
- Supporting files (if needed) - Scripts, examples, tools

### Testing

Skills should be tested before deployment following TDD principles:
1. **RED**: Run pressure scenarios without the skill
2. **GREEN**: Write skill addressing observed failures
3. **REFACTOR**: Close loopholes and edge cases

See [superpowers:writing-skills](https://github.com/anthropics/claude-code) for complete testing methodology.

## License

MIT - See individual skill files for specific licensing information.

## Links

- [Claude Code](https://claude.ai/code)
- [Agent Skills Specification](https://agentskills.io/specification)
- [Skills CLI](https://www.npmjs.com/package/skills) - Tool for installing agent skills
- [Zernio CLI](https://docs.zernio.com/cli)
- [JBang](https://www.jbang.dev)
