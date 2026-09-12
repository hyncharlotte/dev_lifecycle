# Contributing

Thanks for your interest in improving dev_lifecycle.

## What kind of contributions are welcome

### Highly welcome

- **New skill workflows** that follow the same design principles (gate conditions,
  measure-first, real-world origin for every rule)
- **Improvements to existing skills** — especially when backed by a concrete example
  of a problem the improvement would prevent
- **Integration guides** for more AI coding tools in `coding-skills/README.md`
- **Translations** — the skills are currently English with a bilingual project README

### Please discuss first

- Changing the structure of existing skills (open an issue first)
- Adding scripts or code — this is intentionally a docs-only project
- Adding rules that are "good practice" but don't have a concrete origin story

### Not a fit

- Project-specific configurations (that's what `[ADAPT]` markers are for)
- Tool-specific formats (skills should stay tool-agnostic in the source)

## How to contribute

1. Fork the repository
2. Create a branch (`git checkout -b improve-audit-pass-2`)
3. Make your changes
4. Ensure the quality bar (see below)
5. Open a PR

## Quality bar

Every change to a skill must pass these checks:

- [ ] **Real-world origin**: can you describe the concrete problem this prevents?
- [ ] **Gate condition**: if you added a step, does it have a verifiable gate?
- [ ] **No duplication**: does another skill already cover this?
- [ ] **`[ADAPT]` markers**: project-specific values use markers, not hardcoded examples?
- [ ] **Cross-references**: links between skills (audit ↔ dev ↔ push) still correct?
- [ ] **README**: project-level and coding-skills/ READMEs still accurate?

## PR format

```
## What changed

<Which skill, which section, what's different>

## Why

<The concrete bug / miscommunication / wasted time this prevents.
 "Good practice" without a story is not enough.>

## Verification

<How you checked this doesn't break the skills' coherence>
```

## Style guide

- English for all skill content (file names, headings, body)
- One idea per sentence
- Concrete over abstract: "lint passes and only spec'd files were touched" over "ensure code quality"
- `[ADAPT: description]` for anything project-specific
- No emoji unless the user's project conventions use them

## License

By contributing, you agree that your contributions will be licensed under
the Apache 2.0 license (same as the project).
