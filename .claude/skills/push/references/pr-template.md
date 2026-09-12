# PR Title and Body

## Title

`type: one sentence describing what was done`

**Test: fits in one sentence, no "and".** Needing "and" means it's two things → two PRs.

```
fix: BUG-25 -- empty customer no longer breaks the follow-up page
feat: test data split into clean/dirty tiers with scale toggle
docs: acceptance audit -- 132 items verified across all design docs
```

## Body

Written for **the person reviewing the PR**, not for git. They need to decide
"can this merge?" -- so the body answers three things:
**what changed, how it was verified, what's NOT done.**

```markdown
<One paragraph: what problem this solves, with user context or bug reference>

## Changes

<Best as a table or bullet points. **Don't list filenames** -- that's in the diff.
 Say "which code change solves what.">

## Verification

<Measured numbers. "374ms → 41ms" is a hundred times more useful than "faster."
 Include reproduction steps when applicable.>

- Full test suite: N passed / 0 failed (M suites registered)
- Lint: clean
- New/changed tests: `tests/xxx_test.py` (N assertions)

## Not done

<**This section must not be skipped.** It's the most valuable section in the PR --
 the reader's worst fear is assuming everything is resolved.

 Three categories:
 · Deliberately deferred -- and why
 · Found during review but has tradeoffs -- needs user decision
 · Out-of-scope items encountered along the way>

## Build / Deploy impact

<Will this trigger a production build? Which paths changed that are in the build manifest?
 Or: "Only tests/docs changed -- no build triggered, and that's correct.">
```

## How to write "Not done"

**Don't write it as a disclaimer.** It's not "these aren't my fault" --
it's "the next person needs to know these are still open."

Bad:

> Frontend not optimized, could be improved later.

Good:

> **Frontend shows blank page when any API fails instead of graceful degradation.**
> The two panels are independent; when status API fails, the name list should still render
> with status badges hidden. Fix is in `static/app.js` where `Promise.all` should become
> individual catches. One task, separate PR.

The difference: says **what, where, why not now, where to start next time**.

## Footer

End the PR body with your tool's attribution, if applicable.
