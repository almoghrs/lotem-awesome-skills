---
name: create-skill
description: >
  Scaffolds a new AgentSkills-compatible skill for this registry, following
  the agentskills.io specification and lotem-awesome-skills conventions.
  Use when creating or updating a skill: guides directory naming, SKILL.md
  frontmatter, progressive-disclosure structure, and the quality checklist.
  Triggers on phrases like "create a skill", "new skill", "add a skill",
  "write a SKILL.md", or "scaffold a skill".
license: MIT
compatibility: Designed for any AgentSkills-compatible agent (Claude Code, OpenCode, Cursor, Codex, Gemini CLI, etc.)
---

# Create Skill

This skill guides an agent through creating a well-formed, registry-ready
AgentSkills skill from scratch, following the
[agentskills.io specification](https://agentskills.io/specification) and the
conventions of this registry.

## When to Use

- A user asks to create, add, or scaffold a new skill
- You need to write or update a `SKILL.md` file
- You want to ensure a skill meets the registry quality checklist before committing

## Specification Quick Reference

```
skill-name/           # dir name = name field (lowercase, hyphens, max 64 chars)
├── SKILL.md          # Required: frontmatter + markdown body (<500 lines)
├── scripts/          # Optional: executable code (Python, Bash, JS)
├── references/       # Optional: extra docs loaded on demand (<200 lines each)
└── assets/           # Optional: templates, images, static files
```

**Frontmatter fields:**

| Field           | Required | Constraints                                                   |
|-----------------|----------|---------------------------------------------------------------|
| `name`          | Yes      | Lowercase `a-z`, hyphens, max 64 chars. Must match dir name.  |
| `description`   | Yes      | Max 1024 chars. State *what* it does **and** *when* to use it.|
| `license`       | No       | e.g. `MIT`, `Apache-2.0`                                      |
| `compatibility` | No       | Only if skill needs specific tools / environments             |
| `metadata`      | No       | Arbitrary key-value map (`author`, `version`, etc.)           |
| `allowed-tools` | No       | Space-separated pre-approved tools (experimental)             |

## Steps

<!-- TODO: expand each step with concrete agent instructions and examples -->

1. **Name the skill** — Choose a lowercase, hyphen-separated name that matches
   its primary action (e.g. `pdf-extraction`, `git-branch-cleanup`). Verify:
   - Only `a-z` and `-`
   - Does not start or end with `-`
   - No consecutive `--`
   - Max 64 characters

2. **Create the directory** — `skills/<skill-name>/` inside this registry.

3. **Write frontmatter** — Fill `name` and `description` at minimum. The
   description must answer: *what does it do?* and *when should an agent use it?*
   Include specific trigger keywords.

4. **Write the body** — Structure the Markdown with:
   - A 1–3 sentence overview
   - **When to Use** — bullet list of triggers / scenarios
   - **Steps** — numbered, imperative instructions for the agent
   - **References** (optional) — links to `references/` files for detail

5. **Apply progressive disclosure** — If any section exceeds ~50 lines, extract
   it to `references/<topic>.md` and link from the body.

6. **Run the quality checklist** (see below) before committing.

## Quality Checklist

- [ ] `name` in frontmatter matches the directory name exactly
- [ ] `description` states what and when (includes trigger keywords)
- [ ] `SKILL.md` body is under 500 lines
- [ ] No credentials, secrets, or personal data
- [ ] Skill is tested with at least one agent

## Registry Conventions

- Use `SKILL_TEMPLATE.md` at the repo root as a blank starting point
- Update the **Skills Index** table in `README.md` after adding a new skill
- Follow the contribution process in `CONTRIBUTING.md`

## References

<!-- TODO: add references/frontmatter-fields.md with extended field docs -->
<!-- TODO: add references/naming-guide.md with naming decision tree -->
