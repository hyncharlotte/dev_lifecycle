# CLAUDE.md

Project-level instructions for AI coding assistants working on this repository.

## What this project is

`dev_lifecycle` is a set of reusable AI coding skills (audit / dev / push).
The skills live in `coding-skills/` and are meant to be copied into other projects.

## Working on this repo

### Structure

- `coding-skills/` — the deliverable. Each subdirectory is one skill with a `SKILL.md`
  and optional `references/` folder.
- Everything else (README, CONTRIBUTING, this file) is project infrastructure.

### Editing skills

When editing a skill (`SKILL.md` or references), follow these rules:

1. **Every principle must have a real-world origin.** Don't add advice "just in case."
   If you can't point to a concrete bug, miscommunication, or waste of time that
   the rule prevents, it doesn't belong.

2. **Keep `[ADAPT]` markers for project-specific values.** Skills must stay tool-agnostic
   and project-agnostic. Anything that depends on a specific test runner, file layout,
   or branch convention goes behind an `[ADAPT]` marker.

3. **Gate conditions are the valuable part.** Every step in a workflow must have a
   concrete, verifiable gate condition. "Review the code" is not a gate condition.
   "Lint passes and only spec'd files were touched" is.

4. **Don't duplicate between skills.** If `push` references a checklist from `dev`,
   it says "follow the /dev Step 6 checklist" — it doesn't copy it.

### Language convention

- File names and directory structure: English
- Section headings: English
- Body text: English (bilingual README at project root is the exception)
- Examples and templates: English with universal examples

### Commit messages

```
type: one sentence (feat / fix / docs / refactor)

Body: why, not what. The diff shows what.

Co-Authored-By: ...
```

### Branch

Develop on the designated branch, push with `git push -u origin <branch>`.

### No scripts

This is a documentation-only project. The skills package intentionally does not
include scripts — users write their own tooling for their own stack.
If a skill needs to describe a measurement technique, describe the approach
in prose and let the user implement it.

### Quality bar

Before shipping changes to skills:
- Read the changed skill end-to-end. Does every sentence earn its place?
- Check cross-references between skills (audit ↔ dev ↔ push). Still accurate?
- Verify `[ADAPT]` markers are present wherever a value is project-specific.
- Ensure the README (project-level and coding-skills/) is still accurate.
