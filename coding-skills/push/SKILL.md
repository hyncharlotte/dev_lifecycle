---
name: push
description: Ship workflow -- close out docs, self-review, run full tests, commit, push, open PR, enable auto-merge, watch CI and fix failures yourself. Use when someone says "push" / "commit" / "open a PR" / "ship it" / "done" / "wrap up" / "merge it". Goal is the user only needs to check two things -- tests green and the right things are shipping.
---

# push: Ship the finished work

## What the user wants

The user should only need to look at two things:
1. **Tests**: green or not?
2. **Scope**: does the right set of changes ship?

Everything in between -- committing, pushing, opening the PR, self-review,
CI failures -- should not require their attention.

## Key decisions to configure

| Decision | Options | `[ADAPT]` |
| --- | --- | --- |
| Who merges | Auto-merge on green CI / Manual merge | `[ADAPT]` |
| Review | Self-review before opening / Request external review | `[ADAPT]` |
| PR granularity | One logical change per PR | Default |
| CI failures | Fix yourself, report after | Default |

## Seven steps

### Step 1 -- See what you're shipping

```bash
git status --short && git diff --stat HEAD && git log --oneline -5
```

**One logical change per PR.** Test: can the PR title be one sentence without "and"?

But apply this rule with judgment:

| Situation | Action |
| --- | --- |
| Two unrelated changes | Separate PRs. Worth the wait. |
| Two halves of one goal | **One PR**, title names the shared goal, body has two sections |
| One touches product code, one doesn't | **Separate** -- keeps the "what ships" answer clean |

### Step 2 -- Close out (the non-code bookkeeping)

Follow the `/dev` Step 6 checklist:

- [ ] Known-issues status updated (if a bug was fixed)
- [ ] Design doc status stamped, file moved to completed
- [ ] Roadmap / priority table updated (**new items added same day**)
- [ ] Test registration (unregistered test = invisible test)
- [ ] README numbers updated if changed

### Step 3 -- Self-review

Run your project's review tool or do a manual diff review:
`[ADAPT: your review/lint command, e.g., /code-review]`

**Every finding, reproduce it yourself** before deciding -- this catches:
- False conclusions (grepped similar text, assumed same bug -- actually correct)
- Near-crashes (deleted dead code also deleted a live variable)
- Understated severity (both sides wrong, opposite directions)

For each finding:
- Can fix → **fix now**
- Can't fix or has tradeoffs → write into PR body's "Not done" section
- Can't reproduce → it's not a finding, don't claim it

### Step 4 -- Verify (all must pass)

```bash
[ADAPT: your lint command]
[ADAPT: your test command]
```

Test registration check (ensure no test files are invisible to the runner):
`[ADAPT: your test registration verification]`

If the change touches UI, also run integration/E2E tests.
`[ADAPT: your E2E command, if applicable]`

### Step 5 -- Commit + push + open PR

**Commit message convention**:

- First line: `type: one sentence describing what was done`
  (feat / fix / docs / test / build / refactor)
- Body explains **why** and **how verified**, not which files changed (that's in the diff)
- Include measured numbers: "374ms → 41ms" beats "faster"
- List what's NOT done -- the reader's worst fear is assuming everything is resolved
- Attribution at the end per project convention

Push: `git push -u origin <branch>`. Retry on network errors (up to 4 times, exponential backoff).

**PR body** template: `references/pr-template.md`

**Auto-merge gate** (if auto-merge is configured):

> Changed product code but no test covers this change → **don't enable auto-merge, ask the user.**

The criterion: does the diff touch product source files? If yes, the same diff
must include a corresponding test change (new or modified), or you can point to
an existing test that would catch a regression. If you can't point to one,
"CI green" guarantees nothing about this change -- so it shouldn't auto-merge.

Doc-only / test-only / config-only PRs are exempt.

### Step 6 -- Watch CI, fix red yourself

Subscribe to PR activity and wait for events. **Don't poll with sleep loops.**

When CI fails, follow this triage order:

1. **Is it this PR's fault?**
   Both must be true to say "not mine": the error points to code this diff didn't touch,
   AND the same check is red on the base branch.
   - Not yours: if a fix PR exists, cherry-pick it into your branch and push.
     Either way, leave one comment on the PR explaining which check, why not yours,
     and what fix exists (or doesn't).
   - Yours: root-cause and fix (see below).

2. **Root-cause fix only. Never**:
   - Skip / disable / quarantine a test
   - Empty commit or close-reopen to kick CI
   - Loosen assertions to "just barely pass"

3. **"Flake" is not a root cause.** Only re-run CI when:
   - Confirmed not this PR's fault (per above)
   - Runner crashed before any test body ran (checkout/install/infra failure)
   - Same commit passed before
   Maximum one re-run total. Second failure = real.

4. **Validate locally before pushing.** One verified push beats three speculative ones.

**Don't interrupt the user mid-fix.** Fix, push, and report together:
"CI failed once, cause was X, fixed in commit Y."

Details: `references/ci-troubleshoot.md`

### Step 7 -- Report two things only

Final message to the user covers **only what they need to see**:

```
Tests: 2439 passed / 0 failed (49 suites)
Scope: [product code changed -- build will trigger]
   or: [only tests/docs changed -- no build, and that's correct]
PR #170 opened, auto-merge enabled, will merge on green CI
```

**Always state whether production artifacts (builds, deploys) will trigger and why.**
The user will ask "why didn't it build?" when the answer is "because only tests
changed, which is correct" -- say it preemptively.

## When to stop and ask

Only three situations, everything else runs to completion:

1. **Changed product code but no test covers it** → don't auto-merge
2. **Tradeoff to decide** (two valid approaches, two docs conflict)
3. **Irreversible action** (delete branch, force-push, change repo settings)

## `[ADAPT]` section -- fill in for your project

```
Test command:              [e.g., pytest, npm test]
Lint command:              [e.g., ruff check ., eslint .]
E2E command:               [e.g., playwright test, cypress run]
Test registration check:   [how to verify all test files are registered]
Review command:            [e.g., /code-review, or manual diff review]
Merge strategy:            [auto-merge on green / manual merge]
Branch name:               [your development branch]
Build trigger paths:       [which files trigger a production build]
PR template location:      [e.g., .github/pull_request_template.md]
```

## Reference files

- `references/pr-template.md` -- PR title and body template, "Not done" section guidance.
- `references/ci-troubleshoot.md` -- CI failure triage, known-flake handling, merge conflicts.
