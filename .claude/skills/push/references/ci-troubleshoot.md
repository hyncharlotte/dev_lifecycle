# CI Failure Triage

User expectation: **fix it yourself, report after, don't interrupt mid-fix.**

## Triage order (don't skip steps)

### 1. Is it this PR's fault?

Both must be true to say "not mine":
- The error points to code **this diff did not touch**
- **The base branch has the same failure** (check the most recent base branch CI run)

If not yours:
- A fix PR exists → cherry-pick it into your branch and push (becomes a no-op once base merges it)
- No fix exists → leave one PR comment: which check, why not yours, fix status
- Then re-run once (the one allowed re-run, see below)

If yours: root-cause and fix.

### 2. Root-cause fix (never work around)

**Never do these**:
- Skip / disable / quarantine the failing test
- Push an empty commit or close-reopen to re-trigger CI
- Loosen assertions until they happen to pass
- `--no-verify` or any hook bypass

**Do this**:
- Reproduce the failure locally first
- Fix the root cause
- See the same test go from red to green locally
- Then push

### 3. "Flake" is not a root cause

Only three situations justify a CI re-run, **and only one re-run total**:

1. Already confirmed not this PR's fault (per step 1)
2. The test runner died before any test body executed (checkout failure, dependency
   install failure, runner crash)
3. The exact same commit already passed CI before

Everything else is a real failure. A real project learned this the hard way:
a test was labeled "intermittent" for weeks. When actually investigated, it failed
**at the exact same point every time** -- position was fixed, not random.

A second failure after re-run = definitely real. No more re-runs.

### 4. Push only when verified

One verified push beats three speculative ones.
Fix locally, see green locally, then push.

## Merge conflicts

Merge the base branch into your PR branch. **Don't rebase or force-push.**

```bash
git fetch origin <base-branch> && git merge origin/<base-branch>
```

- Lock files and generated files: regenerate with project tooling, don't hand-edit
- Both sides changed the same logic: ask the user (picking either side loses behavior)
- Both sides added to the same list: keep both additions

## Known flaky tests

If your project has tests with known stability issues, document them:

```markdown
| Test | Symptom | Workaround |
| --- | --- | --- |
| `test_name` | Times out on line N in CI, passes locally | Run individually to confirm it's healthy |
```

These are tech debt, not excuses -- each one should have a plan to fix.
