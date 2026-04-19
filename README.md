# lotem-awesome-skills

A curated registry of [AgentSkills](https://agentskills.io)-compatible skills. Every skill is a complete, ready-to-use `SKILL.md` that works with Claude Code, OpenCode, Cursor, Codex, Gemini CLI, GitHub Copilot, Windsurf, and any other AgentSkills-compatible agent.

## How to Use a Skill

Copy the skill directory into your agent's skills path:

| Agent          | Project path            | Global path                        |
|----------------|-------------------------|------------------------------------|
| Claude Code    | `.claude/skills/`       | `~/.claude/skills/`                |
| OpenCode       | `.opencode/skills/`     | `~/.opencode/skills/`              |
| Cursor         | `.cursor/skills/`       | `~/.cursor/skills/`                |
| Codex          | `.codex/skills/`        | `~/.codex/skills/`                 |
| Gemini CLI     | `.gemini/skills/`       | `~/.gemini/skills/`                |
| GitHub Copilot | `.github/skills/`       | `~/.copilot/skills/`               |
| Windsurf       | `.windsurf/skills/`     | `~/.codeium/windsurf/skills/`      |

Or install via [skillkit](https://github.com/rohitg00/skillkit):

```bash
skillkit install almoghrs/lotem-awesome-skills
```

## Skills Index

### Developer Tools

| Skill | Description |
|-------|-------------|
| [cli-copilot-review-loop](skills/cli-copilot-review-loop) | Iterative CLI-based code review loop with an AI copilot agent |
| [create-skill](skills/create-skill) | Scaffold a new AgentSkills-compatible skill following registry conventions |
| [skill-creator](skills/skill-creator) | Create, evaluate, and iteratively improve skills with benchmarking and description optimization — sourced from [anthropics/skills](https://github.com/anthropics/skills) |

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) to add or improve skills.

## Specification

All skills follow the open [AgentSkills specification](https://agentskills.io/specification).

## License

MIT
