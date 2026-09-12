# Report Template

Two outputs, two different readers -- so they can't be the same content in two formats.

- **Repository report** `docs/audit-YYYY-MM-DD.md` -- for the **next person who audits**
  (probably a future you). Must be: reproducible, evidence-backed, clear about what wasn't checked.
- **Summary** -- for the **user/stakeholder**. Must be: current status, what needs their
  decision, what's next.

## 1. Repository report

```markdown
# Audit YYYY-MM-DD (light / full)

> Commit: `<short hash>` · Previous audit: `<previous hash>` (next audit's --since baseline)
> This tier skipped: <list what light tier didn't run>

## 0. One sentence

<The single most important finding. If everything is green, say so --
"no issues found" is a valuable conclusion. Don't invent problems to look busy.>

## 1. Current numbers

| | This time | Last time | Change |
| --- | --- | --- | --- |
| Source lines (main packages) | | | |
| Largest single file | | | |
| Tests: on disk / registered / exceptions | | | |
| Test assertion count | | | |
| Coverage | | | |
| Circular dependencies | | | |

## 2. Findings

**Confirmed** (reproduction steps work)

| ID | Summary | Severity | Evidence | Action |
| --- | --- | --- | --- | --- |
| BUG-N | | High/Med/Low | `file:line` + repro steps | Fix now / Defer / Track |

**Suspicious** (looks wrong but NOT reproduced -- don't upgrade to confirmed)

| Summary | Why suspicious | How to confirm |
| --- | --- | --- |

**Dead code** (no callers)

| Name | Location | Why still present (pinned by a guard?) |
| --- | --- | --- |

## 3. Test credibility

- Registration: <aligned / which ones were missing, what happened when registered>
- Default command: <pass / fail / crashed suites>
- Integration/E2E: <results; assertion count matches source?>
- Isolated vs suite: <which files checked, consistent?>
- Coverage: <lowest modules; low because dead code or missing tests?>
- Guard health: <structural guards, suppression lists still valid?>

## 4. Documentation vs reality

| File:line | Claims | Actual | Action |
| --- | --- | --- | --- |
| `README.md:60` | | | Fixed / Needs user decision |

## 5. Not checked

<List explicitly. Half the value of an audit is knowing what was NOT examined.>
```

Three disciplines when writing the report:

1. **Every finding has runnable evidence.** "Feels wrong" goes to Suspicious, not Confirmed.
2. **Re-check your own previous conclusions.** If you were wrong before, correct in the
   new report -- don't edit the old one. Old reports are records; editing them hides how
   your judgment drifted.
3. **All numbers measured fresh.** The "last time" column can reference old reports;
   the "this time" column cannot.

## 2. User summary

A concise page/message the user can forward or revisit. Structure:

1. **One-sentence status** -- can the project ship? anything bleeding?
2. **Needs your decision** (most prominent -- the only part requiring user action).
   Each item: what it is, 2-3 options, cost of each, your recommendation.
3. **What was checked and results** -- one line per pass, green = just say green.
4. **Findings** -- confirmed / suspicious / dead code, shorter than the report.
5. **Key numbers** -- code size, tests, coverage, compared to last audit.
6. **Next steps** -- ordered by priority, mark which ones don't need user approval.
