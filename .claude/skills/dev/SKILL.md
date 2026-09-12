---
name: dev
description: Development workflow -- from requirement to done, six steps each with a gate condition. Use when someone says "build this" / "implement this" / "fix this bug" / "do this feature" / "work on this PRD". Also for "where is this feature at" / "what's left on this". When in doubt, use it -- the most expensive mistakes come from skipping the process and jumping straight to code.
---

# dev: How to finish a piece of work

## Why this exists

This workflow does two concrete things:

1. **Codifies what good projects already do** -- requirements with acceptance criteria,
   tests before code, specs that map acceptance to code locations.
2. **Adds the step that every project skips** -- going back to stamp the docs when done.

The second point is not cosmetic. In a real project, a status audit found that
half of all "TODO" items were already done -- six of them had no status line at all,
so nobody could tell. Every time the audit pass checks docs-vs-reality, everything
it catches is a step this workflow missed.

## Three tiers

Determine the tier during Step 0 (triage) and **say it out loud**.

| Tier | When | Steps |
| --- | --- | --- |
| **Small fix** | Typo, UI tweak, bug fix within existing rules -- **no new decisions** | 4 → 5 → 6 |
| **One feature** | Default. Fits in one design doc | All six steps |
| **Architecture** | Splitting files, changing schemas, migrations, parallel work | All six + **must measure before spec** |

**The criterion is not line count -- it's "are there new decisions to record?"**
A 3-line change that overturns another feature's acceptance criteria has decisions
worth recording, so it's a full feature run. A toast message change is a small fix.

## Six steps and their gate conditions

**The process itself isn't valuable -- the gate conditions are.** Without them,
every step becomes "close enough, let's move on."

### Step 0 -- Triage

Three questions, answer and move on (no need to ask the user):

1. **Bug or feature?** Bugs go to the known-issues tracker; features go to design docs.
   Keep these two ledgers separate.
2. **In scope or deferred?** Check if any category of work has been explicitly paused
   by the user/team. (No deferred categories for this project.)
3. **Which tier?**

### Step 1 -- Confirm requirements ← **must ask the user**

**Gate**: You can say in one sentence "**who will hit this, and when**", and the user agrees.

Before asking, **investigate everything that doesn't depend on their answer**: what the
current code looks like, which tests pin the current behavior, whether this change
would contradict other design docs.

Come to the user with findings, not a list of questions.

The real output of this step is not "user said OK" -- it's **the gap in the
requirements you found**. Example: a PRD specified behavior for two pages but
forgot a third page that reads the same data source and would silently change too.
One question, one new acceptance criterion, and a surprise was prevented.

### Step 2 -- Design doc (PRD)

**Gate (all three required)**:

1. **Has a status line.** Format and examples in `references/templates.md`.
2. **Has a testable acceptance checklist.** Each item can be verified with code or a click.
   When writing acceptance, think "**how will the user misuse this**", not just "does it work."
3. **Has a "not doing" section.** Boundaries not written down will be expanded by the next person.

**Two kinds of acceptance items -- don't mix them**:
- "Must build" (green only after implementation)
- "Must not break" (should be green BEFORE any code changes)

This distinction is critical for Step 2.5.

### Step 2.5 -- Acceptance becomes tests (red first)

This happens BEFORE writing product code.

Create test file(s), write assertions for each acceptance item, run them, **record
how many are red and which ones**.

**After the first red run, classify each failure**:

| Why it's red | How to tell | What to do |
| --- | --- | --- |
| Product not built yet | It's a "must build" item | Normal -- this is your TODO list |
| Test itself is wrong | Exception instead of assertion failure; or "must not break" item is red | **Fix the test first** |

**"Must not break" items being red before any code change = your test setup is wrong,
not the product.** If you don't classify first, you'll try to fix a problem that doesn't exist.

### Step 3 -- Spec ← **ask user when there are tradeoffs**

**Gate (all three required)**:

1. **Each acceptance item maps to `file:function`.** If you can't map it, the acceptance
   is too vague -- go back to Step 2.
2. **Each rule has exactly one implementation point.** The most expensive recurring bug pattern
   is "same rule implemented in two places, one of them incomplete." Conversely: when one change
   makes three screens update correctly, it's because there's only one source of truth.
   **Write the single implementation point in the spec.**
3. **List what this change will break.** Existing test assertions, other design docs'
   acceptance criteria, code comments. Miss this and the test suite goes red after you
   think you're done.

**For architecture tier: measure before you spec.** Analyze the dependency graph.
Never decide a file split by gut feel.

### Step 4 -- Development

**Gate**: lint passes + **only touched what the spec pointed to**.

One thing to do alongside coding: **update comments that your change makes false.**
"Why we do it this way" comments become lies after the change. Record the full
decision chain, not just the new conclusion:

> Field X: **was hidden, restored on YYYY-MM-DD.** Original reason for hiding was "..."
> -- user rejected that because "...". Keeping the rejected reasoning so the next person
> who proposes the same thing reads here first.

### Step 5 -- Testing

**Gate (all required)**:

1. Step 2.5's test file goes **from red to all green**.
2. **Register the test** with your test runner. (This project has no tests -- skip.)
   An unregistered test = a test that doesn't exist. This has caused real missed bugs.
3. **Isolated green == suite green.** Run the new test alone and inside the full suite.
   Inconsistency = pollution or timing dependency between tests.
4. **If the change touches paths that could receive messy data, add a dirty-data test case.**
   Clean fixtures never exercise the paths that break on real-world data.

### Step 6 -- Close out

**Gate**: run the audit's docs-vs-reality check (Pass 5). If anything is stale, you haven't closed out.

Checklist -- a simple feature touches more places than you expect:

- [ ] Design doc status line updated (with evidence: which `file:line` satisfies each acceptance item)
- [ ] Design doc moved to "completed" if applicable
- [ ] **Other design docs affected by this change** -- mark which items were superseded, don't delete originals
- [ ] Roadmap / priority table updated (completed items marked, **new items added same day**)
- [ ] Test registration
- [ ] README numbers updated if they changed

**"New work items go into the tracking table the same day."** If they don't, they're invisible
until the next audit discovers them.

## When to ask the user

**Only at two points**: Step 1 (requirement confirmation) and Step 3 (spec tradeoffs).
The other four steps are self-closing -- finish them and report together.

Reason: those two are where a wrong direction means wasted work.

**But one situation always requires stopping, regardless of step**: discovering that
**two design docs require opposite things.** Don't pick one yourself -- present the
conflict and let the user decide.

## Project-specific configuration

```
Test command:              N/A (docs-only project)
Lint command:              N/A
Test registration:         N/A
Design doc location:       GitHub Issues / PR descriptions
Known-issues file:         GitHub Issues
Deferred work categories:  None
Branch convention:         claude/* (AI-assisted branches)
```

## Reference files

- `references/templates.md` -- status line format, design doc template, spec template,
  acceptance checklist guidance, close-out checklist.
