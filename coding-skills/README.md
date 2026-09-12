# Coding Skills

A lightweight, tool-agnostic skill set for AI-assisted software development.
Three skills, each one a proven workflow distilled from real-world project experience.

| Skill | What it does | When to use |
| --- | --- | --- |
| **audit** | Project health check | "What shape is this codebase in?" |
| **dev** | Development workflow | "Build this feature / fix this bug" |
| **push** | Ship workflow | "Push it / open a PR / ship it" |

## How to use with different tools

### Claude Code

Copy `coding-skills/` into `.claude/skills/`:

```bash
cp -r coding-skills/* .claude/skills/
```

Each `SKILL.md` becomes a slash command (`/audit`, `/dev`, `/push`).

### Cursor

Merge the content of each `SKILL.md` into `.cursor/rules/` or `.cursorrules`:

```bash
# Option A: one rules file per skill
cp coding-skills/audit/SKILL.md .cursor/rules/audit.md
cp coding-skills/dev/SKILL.md .cursor/rules/dev.md
cp coding-skills/push/SKILL.md .cursor/rules/push.md

# Option B: append to a single .cursorrules file
cat coding-skills/*/SKILL.md >> .cursorrules
```

### Codex / Other tools

Most AI coding tools accept a system prompt or project instructions file.
Append the content of the relevant `SKILL.md` to that file, or reference it
as context when starting a task.

Common patterns:
- **Codex**: include in `AGENTS.md` or pass as system instructions
- **Windsurf**: add to `.windsurfrules`
- **Aider**: add to `.aider.conf.yml` conventions section
- **Cline**: add to custom instructions in settings

## Customization

Each skill has `[ADAPT]` markers where you should fill in project-specific details:
- Test commands
- Lint/format commands
- Branch naming conventions
- CI pipeline specifics
- Documentation file locations

Search for `[ADAPT]` and replace with your project's values before first use.

## Design principles

These skills are not checklists. Each step has a **gate condition** -- a concrete,
verifiable criterion that must be met before moving on. The gate conditions are the
valuable part; the steps are just scaffolding.

Every principle in these skills was learned the hard way -- from a real bug, a real
miscommunication, or a real waste of time. Nothing is here "just in case."
