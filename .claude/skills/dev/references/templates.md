# Templates: Status Line / Design Doc / Spec

Use the format, not the content. These formats come from real projects that
do documentation well -- they're extracted patterns, not invented ones.

## Table of contents

- [Status line](#status-line) (most important, and most often missing)
- [Design doc template](#design-doc-template)
- [Spec template](#spec-template)
- [How to write acceptance criteria](#how-to-write-acceptance-criteria)
- [Close-out checklist](#close-out-checklist)

---

## Status line

**Every design doc must have a status line before the first `---`.** It's the only
way to tell at a glance "where is this at." In a real audit, half of all "TODO" items
were actually done -- six had no status line at all.

**Format**:

```markdown
**Status: <status word>** (<date>) · <one-line context>
<which acceptance items are satisfied by what `file:line`>
```

**Status vocabulary** (use only these):

| Word | When |
| --- | --- |
| `Done` | All acceptance items verified |
| `Core done` | Main work complete, some items deferred -- **must list which** |
| `Partial` | Some items done, some superseded/paused -- itemize each |
| `In progress` | Actively working -- note which step |
| `Not started` | Nothing built -- **still write this**, with evidence |
| `Needs decision` | Blocked on user/team -- say what specifically |
| `Cancelled` | Cancelled but doc kept for historical value |

**When done, include an evidence table** so the next auditor doesn't have to re-verify:

```markdown
**Status: Done** (2026-09-10) · All 7 acceptance items verified, test `tests/feature_test.py` (19 assertions)

| Acceptance | Satisfied by |
| --- | --- |
| 1. Table page shows toggle button | `templates/table.html` removed conditional guard |
| 7. Detail page also shows it | No code added -- `detail.html:22` already reads from same source |
```

**When superseded by another change, mark per-item, don't delete**:

```markdown
| Acceptance | Current status |
| --- | --- |
| 1. Table hides column X | Superseded (by Feature Y, 2026-09-10) |
| 6. Other tables unaffected | Still valid |
```

> **Why not delete**: the original text records "why we thought this at the time."
> The rejected reasoning is worth keeping -- next time someone proposes the same
> thing, they read here first.

---

## Design doc template

```markdown
# PRD: <one sentence describing the end state>

> User requested (<YYYY-MM-DD>): <user's original words, don't paraphrase>

**Status: <see above>**

---

## 1. Background

<What exists now, who runs into this and when. Include numbers if available.>

## 2. What to build

### 2.1 <One thing>

**Change `<file>`**: <what changes>

Effect:
- <What the user will see>

## 3. What NOT to build

- **Not changing <X>** -- <why>

(This section matters more than it looks: boundaries not written down
get expanded by the next person.)

## 4. Impact scope

| File | What changes |
| --- | --- |

**Migration needed?** -- state explicitly yes or no.

## 5. Acceptance

1. <see "How to write acceptance criteria" below>
```

**Optional sections to add after implementation** (the best design docs have these):

| Section | Content |
| --- | --- |
| Deviations from plan | Numbered list of what differed and why |
| Rejected ideas | Discussed but not built -- prevents re-discussion |
| Overturned assumptions | What we thought was true but wasn't |

---

## Spec template

**Only write a separate spec when the change is large** (multiple files / needs migration /
parallel work). Small changes: add a "How" section to the design doc.

```markdown
# Design: <name> Implementation Spec

**Rules are numbered S-1..., decisions D-1...**
**When something goes wrong, trace back by number. When changing a rule, change here first, then code.**

## S-1 <Rule name>
S-1.1 <Specific requirement>

## D-1 <Decision name>
**Chosen**: <approach>
**Alternative**: <approach B> -- <why not chosen>
```

**The spec must answer three questions** (these are Step 3's gate conditions):

1. Each acceptance item maps to which `file:function`?
2. **Where is the single implementation point?** Same rule in two places = guaranteed future bug.
3. **What does this change break?** Existing test assertions, other docs' acceptance, comments.

---

## How to write acceptance criteria

**One criterion = one runnable test or one clickable verification.** Can't map it? Too vague.

**Two kinds -- know which you're writing**:

| Kind | When it's green | Example |
| --- | --- | --- |
| **Must build** | After implementation | "Settings page shows a toggle" |
| **Must not break** | BEFORE any code change | "Other pages' toggles still work" |

"Must not break" items red before code changes = your test is wrong, not the product.

**When writing acceptance, think "how will users misuse this"**, not just "does it work."
A real bug: PRD said "typing an unknown name can create a new record." Implementation:
"typing half a name auto-creates a record and links it." The acceptance criterion should
have been "half-typed name without selection must NOT auto-create."

---

## Close-out checklist

Step 6 -- verify each item. A simple 2-file change touched 8 places in practice:

- [ ] Design doc status line → Done + evidence table
- [ ] Design doc moved to completed folder
- [ ] **Other design docs affected** -- per-item annotations, don't delete originals
- [ ] Roadmap / priority table updated (done items marked, **new items added same day**)
- [ ] Test registered with test runner
- [ ] README/doc numbers updated if changed

**Gate condition**: run the audit's docs-vs-reality pass. Anything stale = not closed out.
