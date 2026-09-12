# dev_lifecycle

A lightweight, tool-agnostic skill set for AI-assisted software development.

Three battle-tested workflows — audit, develop, ship — distilled from real project
experience into reusable instructions for any AI coding assistant.

一套轻量、不绑定特定工具的 AI 辅助开发技能包。
三个经过实战验证的工作流 — 体检、开发、交付 — 从真实项目经验中提炼，适用于任何 AI 编码助手。

## What's inside / 包含什么

```
coding-skills/
├── audit/          project health check / 项目体检
├── dev/            development workflow / 开发流程
└── push/           ship workflow / 交付流程
```

| Skill | What it does | When to use |
| --- | --- | --- |
| **audit** | Measure-first code review: test credibility, code organization, docs vs reality | "What shape is this codebase in?" |
| **dev** | Six-step workflow from requirement to done, each with a gate condition | "Build this feature" / "Fix this bug" |
| **push** | Self-review, verify, commit, PR, watch CI, report | "Ship it" / "Open a PR" |

## Quick start / 快速开始

### Claude Code

```bash
cp -r coding-skills/* .claude/skills/
```

Skills become available as `/audit`, `/dev`, `/push`.

### Cursor

```bash
cp coding-skills/audit/SKILL.md .cursor/rules/audit.md
cp coding-skills/dev/SKILL.md .cursor/rules/dev.md
cp coding-skills/push/SKILL.md .cursor/rules/push.md
```

### Codex / Windsurf / Aider / Other

See [`coding-skills/README.md`](coding-skills/README.md) for detailed
integration instructions for each tool.

### Customize for your project / 适配你的项目

Each skill has `[ADAPT]` markers. Search and replace with your project's values:

```bash
grep -rn '\[ADAPT\]' coding-skills/
```

Common things to fill in:
- Test command (`pytest`, `npm test`, etc.)
- Lint command (`ruff check .`, `eslint .`, etc.)
- Branch naming convention
- Design doc / known-issues file locations

## Design philosophy / 设计理念

These skills are not checklists. Three principles set them apart:

**Gate conditions over steps.**
每道有关门条件，不是"差不多了"就过。
Every step has a concrete, verifiable criterion. Without it, "close enough" becomes the standard.

**Measure before you look.**
先量后查，数字决定看哪儿。
Don't read code top to bottom. Measure coverage, change frequency, and dependency graphs first — then look where the numbers say to look.

**Record decisions, not just conclusions.**
记完整的判断链，不只记结论。
Keep rejected reasoning alongside accepted decisions. When someone proposes the same thing next year, they read the history instead of re-arguing.

## What this is NOT / 这不是什么

- Not a linter or static analysis tool — it's instructions for AI assistants
- Not a CI pipeline — it tells the AI how to use your existing CI
- Not a project template — it adds workflow discipline to existing projects
- Not prescriptive about tech stack — works with any language, framework, or toolchain

## Project structure / 项目结构

```
dev_lifecycle/
├── .claude/skills/          installed skills (configured for this repo)
├── .github/                 PR template
├── coding-skills/           distributable skill set (with [ADAPT] markers)
│   ├── README.md            tool integration guide
│   ├── audit/               project health check
│   ├── dev/                 development workflow
│   └── push/                ship workflow
├── CLAUDE.md                AI tool instructions for this repo
├── CONTRIBUTING.md          how to contribute
├── LICENSE                  Apache 2.0
└── README.md                this file
```

## Origin story / 来历

These workflows were extracted from a production project where:

- An audit caught 7 reproducible bugs, 4 unregistered tests (one red), 268 untested
  UI assertions, and multiple stale documentation numbers — all by measuring first
- A "split into 6 modules" plan was proven impossible by dependency analysis (7 cycles);
  the real answer was 9 modules
- Half of all "TODO" items were already done — 6 had no status line, so nobody knew
- A 3-line code change required updating 8 places in documentation

Every rule in these skills exists because skipping it caused a real problem.
Nothing is here "just in case."

## Contributing / 参与贡献

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

Apache 2.0 — see [LICENSE](LICENSE).
