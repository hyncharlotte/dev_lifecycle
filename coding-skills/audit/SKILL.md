---
name: audit
description: Project health check -- measure first, then look. Runs test credibility checks, code organization analysis, targeted code review, and documentation-vs-reality reconciliation. Use when someone says "audit" / "health check" / "what shape is this in" / "anything broken" / "review everything". Also use for handoff or status reports.
---

# audit: Project Health Check

## Why this exists

This workflow is not a generic checklist. Every step is here because it caught
a real problem in a real project -- and the order matters because later steps
use numbers from earlier ones to decide where to look.

The key insight: **the most valuable bugs are found by measuring first, then
looking where the numbers say to look** -- not by reading code top to bottom.

## Two tiers

Default to **light** unless user says otherwise.

| Tier | Trigger | What runs |
| --- | --- | --- |
| **light** | `audit` | Passes 1, 2 (no coverage), 5 + code review only on recently changed files |
| **full** | `audit full` | All five passes: full coverage, full code org analysis, full code review |

Tell the user which tier you're running and roughly how long it'll take.

## Five passes, in dependency order

### Pass 1 -- Measure current state

Collect these numbers fresh -- **never copy from old reports or READMEs**:

- **Code size**: line counts per module/package, sorted descending, with subtotals
- **Test registration check**: files on disk vs files actually referenced by your test runner.
  A test file that exists but isn't registered = a test that never runs.
  `[ADAPT: your test runner command and registration mechanism]`
- **Change frequency**: which files changed most since the last audit (or last N commits).
  This feeds Pass 4 -- code review looks here first.
- **Numbers in docs**: grep your README / docs for hard-coded counts, percentages, timings.
  These feed Pass 5.

### Pass 2 -- Test credibility

The question is not "do tests pass" but "can you trust the test suite."
Check from cheapest/most-likely-broken to most expensive:

1. **Registration check** -- Pass 1 already found unregistered tests. **Register them
   immediately and run them** -- they may have been red for months without anyone knowing.
2. **Run the default test command**: `[ADAPT: your test command]`.
   Watch for suites that crash mid-run -- they report 0 failures but exit non-zero.
   Check that the assertion count matches what the source code contains.
3. **Run integration/browser/E2E tests** if they exist and aren't in the default command.
   These are the most likely to be silently broken because nobody runs them regularly.
4. **Isolated vs suite consistency**: pick a few test files and run them alone, then
   in the full suite. Inconsistency = pollution or timing dependency between suites.
   This manifests as random CI failures and destroys trust.
5. **Dirty-data testing**: if you have a test that runs against realistic/messy data
   (as opposed to clean fixtures), run it. Clean fixtures never exercise the paths
   that break on real data.
6. **Coverage** (full tier only): `[ADAPT: your coverage command]`.
   Low-coverage modules feed Pass 4. Distinguish "low because dead code" from
   "low because missing tests" -- they're different findings.

**Credibility criterion**: a test's value isn't how many assertions it has;
it's whether it actually runs and whether someone notices when it's red.

### Pass 3 -- Code organization (full tier only)

Analyze the dependency graph of your modules. The rule is **one-way**:
lower layers don't import from higher layers.

What to look for:
- **Circular dependencies** between modules -- each one is a defect to record
- **Layering violations** -- a utility module importing from a page/route handler
- **Oversized files** -- before splitting, **measure the block-level dependency
  graph first**, don't guess how many pieces it should be

**Never decide a split by gut feel.** Count the actual dependency edges.
A previous project guessed "split into 6 modules" -- real measurement showed
7 circular dependencies that made that plan impossible; the actual answer was 9.

### Pass 4 -- Code review (guided by Passes 1-2)

**Don't read everything.** Prioritize files by:
lowest coverage → largest size → most frequently changed → newly added since last audit.
Light tier: only files listed in Pass 1's change frequency.

Focus on these proven bug patterns:

- **Partial update becomes full overwrite**: reads merge old+new values, but writes only
  save the new ones → silent data loss
- **Empty collection hits SQL/query builder**: produces malformed queries → crash
- **Write path missing a filter the read path has**: creates orphan references
- **Fake validation**: `1 <= day <= 31` is not date validation
- **Write-before-existence-check**: modifying a record that doesn't exist should 404
- **Insert-only sync**: re-running creates duplicates instead of updating
- **Date update clobbers time**: writing a date-only value over a datetime
- **Guessing user intent then writing**: auto-creating records based on partial input.
  Ask: if the guess is wrong, can the user tell?

**Record, don't fix.** Findings go into a known-issues file. Whether to fix depends
on severity and the user's priorities.

### Pass 5 -- Documentation vs reality

Take the numbers from Pass 1 and check every claim in your docs:

| Check | Against |
| --- | --- |
| README test counts / coverage / timing | Pass 2 actual results |
| README directory structure / line counts | Pass 1 actual numbers |
| Roadmap / TODO status entries | Actual code state |
| Known-issues status fields | **Look at the actual code, don't trust the status field** |
| Exemption lists (lint suppressions, skip lists) | Whether exemptions are still valid |

**Fix stale numbers on the spot.** This is the one category of change that doesn't
need user approval -- you're just aligning documentation with reality.

This pass catches things every single time because these numbers only get updated
when someone remembers to. The mechanism of doc rot is: a number gets copied once,
then nobody knows when it changed.

## Standing rules (more important than the five passes)

**Reproduce every finding yourself.** Someone else's conclusion (including your own
from a previous run) is a hypothesis, not a finding. Run the code. Past audits have
caught: a false conclusion (grepped similar text and assumed "same bug" -- both
were actually correct), a near-crash (deleting dead code also deleted a live variable),
and a finding more severe than reported (both sides wrong, opposite directions).

**Classify findings -- don't mix**:
- **Confirmed**: reproduction steps work
- **Suspicious**: looks wrong but not reproduced
- **Dead code**: no callers

Promoting suspicious to confirmed is the fastest way to lose credibility.

**Check the checkers themselves.** Guard tests and lint suppressions can become stale:
- A structural guard that can't distinguish "name is used" from "name is called"
  pins dead code in place
- A `# noqa` / `// eslint-disable` on one line of a multi-line statement only
  suppresses that line, silently hiding findings on the others

**Every number, measured fresh.** Never copy from old reports. Doc rot starts
when a number gets copied the second time.

## Output (both required)

### 1. Repository report

Create `docs/audit-YYYY-MM-DD.md` with:
- Commit hash at time of audit (next audit's `--since` baseline)
- What this tier skipped
- Current numbers vs last audit
- Findings: confirmed / suspicious / dead code (with evidence)
- Test credibility summary
- Doc discrepancies found and fixed
- What was NOT checked

Template: `references/report-template.md`

### 2. Summary for the user

A concise handoff showing:
- One-sentence status
- Items awaiting their decision (most prominent -- this is the only part requiring action)
- What was checked and results
- Findings summary
- Key numbers
- Suggested next steps

## `[ADAPT]` section -- fill in for your project

```
Test command:          [e.g., pytest, npm test, python tests/run_all.py]
Lint command:          [e.g., ruff check ., eslint ., cargo clippy]
Coverage command:      [e.g., pytest --cov, nyc, coverage run]
Test registration:     [how tests get discovered -- file naming? explicit list?]
Known-issues file:     [e.g., docs/known-issues.md, BUGS.md]
Branch convention:     [e.g., feature/*, fix/*]
Docs with hard numbers: [e.g., README.md, docs/architecture.md]
```
