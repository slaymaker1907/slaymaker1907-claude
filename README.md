# slaymaker1907-claude

A collection of Claude skills for claude.ai and Claude Code.

## Skills

### [`model-picker`](model-picker/) — Claude Code
Recommends the right Claude Code model + thinking level (Haiku 4.5, Sonnet 4.6, Opus 4.7) for coding tasks. Considers cost, context window, thinking level, and whether you're planning vs. executing.

### [`clauude-model-picker`](clauude-model-picker/) — claude.ai (general chat/writing)
Recommends the right model + reasoning mode for general workflows: writing, summarization, analysis, communications. Covers Haiku/Sonnet/Opus with Thinking ON/OFF across six configurations.

## Installation

Copy any skill directory into your Claude skills folder:

```
~/.claude/skills/<skill-name>/SKILL.md
```

For example:

```bash
cp -r model-picker ~/.claude/skills/
cp -r clauude-model-picker ~/.claude/skills/
```

Restart Claude Code (or reload the page on claude.ai) and the skill will appear in your available skills list.
