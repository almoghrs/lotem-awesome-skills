# Contributing to lotem-awesome-skills

Thanks for contributing! This registry follows the open [AgentSkills specification](https://agentskills.io/specification).

## Adding a Skill

1. **Fork** this repository and create a new branch.
2. **Create** a directory under `skills/` named exactly after your skill (lowercase, hyphens only):
   ```
   skills/your-skill-name/
   └── SKILL.md
   ```
3. **Use** `SKILL_TEMPLATE.md` as your starting point.
4. **Follow** the rules below, then open a Pull Request.

## SKILL.md Requirements

| Field         | Required | Constraints                                               |
|---------------|----------|-----------------------------------------------------------|
| `name`        | Yes      | Lowercase, hyphens only, max 64 chars. Must match dir name. |
| `description` | Yes      | Max 1024 chars. State what it does **and** when to use it. |
| `license`     | No       | License name (e.g. `MIT`, `Apache-2.0`)                   |

**Body:** Markdown instructions under the frontmatter. Keep `SKILL.md` under 500 lines. Move detailed reference material to `references/` files.

## Directory Structure

```
skills/your-skill-name/
├── SKILL.md          # Required
├── scripts/          # Optional — executable scripts
├── references/       # Optional — extra docs loaded on demand
└── assets/           # Optional — templates, images, static files
```

## Quality Checklist

- [ ] `name` in frontmatter matches the directory name exactly
- [ ] `description` clearly states what the skill does and when to use it
- [ ] `SKILL.md` is under 500 lines
- [ ] No credentials, secrets, or personal data included
- [ ] Skill is tested with at least one agent before submitting

## Improving an Existing Skill

Open a PR with a clear explanation of what changed and why. Bug fixes and clarity improvements are always welcome.
