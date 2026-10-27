# Release Page

One page per build. The first page you make sets the look; every later page copies
it and changes only the content, so a stack of pages reads as one series.

## Structure (top to bottom)

| Block | Content |
| --- | --- |
| Top line | `<date in user's time zone> · PR #… · build #N` |
| Headline | The one or two biggest changes, one sentence |
| Lede | One sentence: "Changes between the last build and this one. Screenshots use test data." |
| Legend | The tags used on this page (New / Changed / Removed) |
| Download box | Source commit + CI link (or checksum); migration sentence if any |
| Table of contents | One chip per group (hidden in the PDF) |
| Groups | Area heading + count, then 1–3 entries |
| Footer | "Screenshots use generated test data, not real customer data." -- never removed |

Each entry:

```
[tag] Title: where + what, one sentence
Body: how to use it, what's different from before (a short list is fine)
Screenshot (+ optional one-line caption saying what to look at)
Boundary line: what did NOT change
```

Two related screenshots (e.g. "first click" / "second click") sit side by side and
stack on narrow screens.

## Style rules

- Single column, max ~880px wide, generous spacing; readable at phone width.
- Colours as tokens on `:root`, with a dark-mode set. The PDF forces light.
- Three tag colours, one per tag type; keep them the same on every page.
- Images: `width`/`height` attributes from the screenshot step, `height: auto`,
  thin border, rounded corners, white background (so a dark page doesn't bleed into
  transparent edges).
- Every `<img>` has `alt` text that describes what the screenshot shows.
- Fonts: one display face for headings, one body face, one monospace for the top line
  and commit hashes. Pick faces that cover the page's language. `[ADAPT: fonts]`

## HTML skeleton

```html
<title>[Product] [theme of this build]</title>
<style>
:root {
  --bg: #f6f7f8; --paper: #fff; --ink: #1d2328; --muted: #5f6b74; --line: #dde2e6;
  --accent: #2f6f62; --accent-soft: #e3efeb;
  --new: #2f6f62; --new-bg: #e3efeb;
  --chg: #8a5a12; --chg-bg: #f6ecd9;
  --del: #9b3b34; --del-bg: #f6e3e1;
}
@media (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]) { /* dark values for the same tokens */ color-scheme: dark; }
}
body { background: var(--bg); color: var(--ink); line-height: 1.75; padding-inline: 16px; }
.wrap { max-width: 880px; margin: 0 auto; padding-block: 48px 72px; display: grid; gap: 48px; }
.tag { font-size: 11.5px; font-weight: 700; padding: 1px 8px; border-radius: 5px; }
.tag.new { color: var(--new); background: var(--new-bg); }
.tag.chg { color: var(--chg); background: var(--chg-bg); }
.tag.del { color: var(--del); background: var(--del-bg); }
.where { font-size: 13px; color: var(--muted); }
figure { margin: 4px 0 0; display: grid; gap: 8px; min-width: 0; }
figure img { display: block; width: 100%; height: auto; border: 1px solid var(--line);
             border-radius: 8px; background: #fff; }
.pair { display: grid; grid-template-columns: repeat(auto-fit, minmax(min(100%, 360px), 1fr)); gap: 20px; }
</style>

<div class="wrap">
  <header>
    <div class="date">2026-10-25 · PR #330 · build #231</div>
    <h1>Payments record on Enter; backups can be linked to orders</h1>
    <p class="lede">Changes between the last build and this one. Screenshots use test data.</p>
  </header>

  <div class="download">
    <span>Built from source d7c9793. The first launch upgrades the database once; existing data is kept.</span>
    <a href="[CI run link]">Download</a>
  </div>
  <nav class="toc"><a href="#payments">Payments</a><a href="#linking">Linking</a></nav>

  <section id="payments">
    <h2>Payments <span class="count">1 item</span></h2>
    <div class="item">
      <span class="tag chg">Changed</span>
      <h3>Edit-order dialog: payments are a table, Enter records one</h3>
      <p>Each payment is a row; the last row stays empty. Type an amount, press Enter, it's recorded.</p>
      <figure>
        <img src="img/pay-table.png" width="828" height="206"
             alt="Payments table in the edit-order dialog: two rows recorded, last row empty">
        <figcaption>The second row was just recorded with Enter.</figcaption>
      </figure>
      <p class="where">The "collect deposit" checkbox on new orders did not change.</p>
    </div>
  </section>

  <footer>Screenshots use generated test data, not real customer data.</footer>
</div>
```

## Print CSS (for the PDF)

```css
@page { size: A4; margin: 14mm 12mm; }
figure, h2, h3 { break-inside: avoid; }
h2, h3 { break-after: avoid; }
.toc { display: none !important; }
```

Not on `.item`: a long entry with several images must be allowed to split across pages.
