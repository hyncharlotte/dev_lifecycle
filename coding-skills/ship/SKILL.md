---
name: ship
description: Release workflow -- build the release artifact, then turn everything that changed between the previous build and this one into a release page (one plain sentence + one real screenshot per change), a PDF of that page, and short release notes committed to the repo. Use when someone says "ship" / "release" / "cut a build" / "build and write release notes". Also use when they only say "build it" (build, then continue with the notes). When they ask "what changed since the last build" / "write release notes", skip the build step and run the rest.
---

# ship: Build + release notes

## What the user wants

Every build comes with a page they can hand to the people who use the product:
**one plain-language sentence per change + one real screenshot**, a download link
at the top, and a PDF of the same page. Plus a short release-notes file in the repo
so the next build knows where this one ended.

This skill exists because the first hand-made release page got three things wrong:

1. **Wrong range.** It listed everything since a calendar date. The user wanted
   "between the last build and this build" -- nothing else.
   The range comes **only from build records**, never from the calendar.
2. **Wrong date.** The assistant's clock was UTC; the user's day was one day
   earlier. **Dates on the page are in the user's time zone.** `[ADAPT: time zone]`
3. **Edited the old page.** New content was added to last build's page, so the link
   people already had changed under them. **One new page per build; old pages are never touched.**

## Six steps

### Step 1 -- Build (skip when only notes were asked for)

Trigger the release build from main.
`[ADAPT: how a release build is triggered -- CI workflow, manual dispatch, local script]`

Wait for it in the background (subscribe to the event or run a background wait);
don't block the conversation with a foreground sleep loop.

- **Red** → stop. Tell the user which step failed. **No release notes** -- there is
  nothing to download.
- **Green** → if the build commits back to the main branch (version bump, artifact
  manifest), re-align the development branch with main before doing anything else.

**CI out of minutes / unavailable → build locally.** Build from the main branch in a
separate worktree so the repo's own build output directory stays untouched, run the
full test suite first (the same two steps CI runs: test, then build), record the
artifact checksum, and send the artifact to the user directly. This build has no CI
record -- so the release page lists the source commit and the checksum instead of a
CI link, and Step 2 takes "previous build" from the latest release-notes file.
If CI is out of minutes, PR checks don't run either: run the full suite locally
before merging and say so in the PR body.

**Gate**: an artifact exists, built from main, with a known source commit
(and a CI link or a checksum).

### Step 2 -- Fix the range: previous build → this build

1. Get the source commit of the last two successful builds from the build records.
   `[ADAPT: where build records live -- CI run history, release tags]`
2. List what was merged in between:
   `git log --first-parent --merges --format='%h %s' <prev>..<this>`
3. See what actually ships:
   `git diff <prev>..<this> --stat -- [ADAPT: paths that go into the artifact]`

Rules:

- Cross-check `<prev>` against the source commit written in the latest release-notes
  file. If they differ, a build went out without notes -- follow the build records
  and say so in one line on the page.
- **Only changes that go into the artifact count.** A PR that only touched docs or
  tests is not a feature; at most one line at the end of the notes ("#123 changed docs only").
- **Nothing shipping changed** → tell the user "this build is identical to the last
  one, no new features" and stop. No page.
- **Data migration in range** → the download box says so, in user terms
  ("the first launch upgrades the database once; existing data is kept").
  `[ADAPT: how to detect a schema/data migration between two commits]`

**Gate**: `<prev>` and `<this>` commits written down, every PR in range classified
as ships / doesn't ship.

### Step 3 -- Write the entries

Read each PR's body and the design doc it implements (the user's own words are
usually in the decisions section). Write **one entry per change the user can see**:

- **Tag**, one of three: *New* (didn't exist) / *Changed* (existed, works differently) /
  *Removed* (no longer shown).
- **Title** says **where and what** in one sentence:
  "Edit-order dialog: payments are a table, Enter records one".
- **Body** says how to use it and what's different from before. No file names,
  function names, migration numbers, or test counts -- those live in the PR.
- **Boundary line last**: what did *not* change ("the schedule column is still locked"),
  so nobody assumes everything moved.
- Group by area of the product, 1–3 entries per group.

**Gate**: every shipping PR maps to at least one entry, and no entry mentions code.

### Step 4 -- Screenshots

Use the project's **baseline test data** -- the same dataset the user downloads to
test with -- never real data. `[ADAPT: how to generate the baseline dataset]`

One screenshot per entry, cropped to **only the part that matters** (the dialog plus
the rows under it, not the whole screen -- full-screen shots are unreadable in a PDF).
The technique, for whatever browser-automation tool you use:

- Start the app on the baseline data, open a fixed viewport at 2× device scale.
- Per shot, describe a small step list: open a page → click / fill / press / run JS /
  wait for a selector or condition → scroll the target into view.
- Compute the crop as the union of the bounding boxes of one or more selectors, plus a
  few pixels of padding, and screenshot just that rectangle.
- Record each image's CSS-pixel width and height -- they go into `<img width height>`
  so the page doesn't jump while loading.
- Collect page JS errors; any error means the shot may not show what you think.

**Look at every image after taking it**: is a dialog cut in half? Grey overlay edge
captured? Does the baseline data actually contain a case that shows the feature?
If not, add that case to the baseline data generator first, then re-shoot.

**Gate**: one image per entry, each one opened and checked by eye.

### Step 5 -- Page + PDF

**Page.** Copy the previous release page and change only the content: colours, fonts
and layout stay identical so a stack of pages reads as one series.
Structure and template: `references/release-page.md`.

- Top line: `<date in user's time zone> · PR #… · build #N`
- Headline: the one or two biggest things in this build, in one sentence.
- Download box: source commit + CI link (or checksum); migration sentence if any.
- Page title: product name + this build's theme, 2–4 words. No filler like "Release notes".
- Publish as a **new** page every time `[ADAPT: where pages are published]`. It is
  private by default -- tell the user how to share it.

**PDF.** Render the same HTML to A4 with a headless browser:

- Force the light colour scheme (it will be printed).
- Wait for web fonts to finish loading before printing; provide a local fallback font
  that covers the page's script, so missing fonts never print as boxes.
- `break-inside: avoid` on **images and headings only**, and `break-after: avoid` on
  headings. Do **not** make whole entries unbreakable: one long entry with three images
  got pushed to the next page, left the previous page mostly blank, and pushed the
  footer on top of an image.
- Hide the in-page table of contents.

Open the PDF and look at every page before sending it.

**Gate**: page published at a new URL; PDF checked page by page.

### Step 6 -- Release notes into the repo + report

Add `[ADAPT: release-notes dir]/<date>_build-<N>.md`: a few lines of metadata
(date, build number, source commit, previous commit, link to the page) and one
sentence per entry. Commit to the development branch and push -- docs only,
no PR of its own; the next PR carries it into main.

Final message to the user has three things only:

```
Build #231 is done (Oct 25). What's new:
- Edit-order dialog: payments are a table; type an amount, press Enter, it's recorded
- Backups table: unlinked rows can be linked to an order
- Income / Chat / Post-production tables: same order picker; Schedule still locked

Page: <link>   PDF: attached   Artifact: <CI link or "sent above, sha256 …">
```

**Gate**: notes file pushed; the message has the entries, the page link, the PDF,
and the artifact.

## Standing rules

- Always build from main.
- Screenshots only from baseline test data, and the page footer says so
  ("screenshots use generated test data, not real customer data") -- never remove it.
- Dates in the user's time zone; range from build records; old pages never edited.
- Never touch the repo's own build output directory.

## `[ADAPT]` section -- fill in for your project

```
Release build trigger:     [e.g., CI workflow_dispatch on main, make release]
Build records:             [e.g., CI run history for the build workflow, git tags]
Local build fallback:      [test command + build command, run in a worktree of main]
Paths that ship:           [directories/files that go into the artifact]
Migration detection:       [e.g., diff of a schema version constant]
Baseline test data:        [command that generates it]
User time zone:            [e.g., America/Los_Angeles]
Page publishing:           [e.g., private artifact, internal docs site]
Release-notes directory:   [e.g., docs/releases/]
```

## Reference files

- `references/release-page.md` -- page structure, style rules, and an HTML skeleton.
