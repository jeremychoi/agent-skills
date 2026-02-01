# Jeremy Choi's Agent Skills

Each skill teaches an agent how to perform a specific task.

## How to Use

### For Claude Desktop or Claude Code

1. Clone this repository:
   ```bash
   git clone https://github.com/jeremychoi/agent-skills.git
   ```

2. Copy the skill folder to your Claude skills directory:
   ```bash
   cp -r agent-skills/skills/openclaw-hardening ~/.claude/skills/
   ```

3. Claude will detect the skill automatically. Ask it to use the skill:
   ```
   "Audit my OpenClaw configuration for security issues"
   ```

### For Other AI Agents

Point your agent to the `SKILL.md` file. The file contains:
- **Frontmatter**: Name and description (when to use)
- **Instructions**: Step-by-step workflow
- **Examples**: Sample interactions

Your agent reads the skill file and follows the instructions.

## Available Skills

| Skill | Description |
|-------|-------------|
| [openclaw-hardening](skills/openclaw-hardening/SKILL.md) | Security auditor for OpenClaw/Moltbot/Clawdbot. Checks network bindings, auth settings, and installed skills for vulnerabilities. |
| [superset-api](skills/superset-api/SKILL.md) | Work with Apache Superset REST API. |

## Skill Structure

```
skills/
  skill-name/
    SKILL.md          # Required: instructions and examples
    helper-file.md    # Optional: additional context
```

Each `SKILL.md` follows this format:

```markdown
---
name: skill-name
description: When to use this skill.
---

# Skill Title

## Instructions
Step-by-step workflow...

## Examples
Sample user/agent interactions...
```

## License

MIT
