# Design System — Filter Coffee Way / System Design

A spec for the warm-paper editorial redesign. Use this to roll the new look across `designs/*.html`, `patterns/*.html`, `_topic-roadmap.html`, and `templates/design-template.html`.

The reference implementation lives in `index-new.html` (will become `index.html` once approved). Read that file alongside this spec — it's the canonical example of every token, type ramp, and component below.

> **For Sonnet** — when redesigning a page, do not improvise tokens. Pull them from this doc. The whole point is consistency.

---

## 1. Tone

Warm-paper editorial. Think small-press journal — Stripe Press, Frank Chimero, a Filter Coffee Way book. Cream paper, espresso ink, terracotta accents. Serif display + clean sans body. Calm, confident, scannable.

Three rules:

1. **Long-form first.** Every design page is an article. Generous margins, narrow measure (~62ch), comfortable line-height (1.7 for body, 1.6 for UI). Never let a paragraph stretch wider than ~62 characters.
2. **Ornament is a seasoning.** A drop cap, a fleuron divider, a kicker rule — used sparingly. If a page has more than two ornaments, remove one.
3. **Color carries meaning.** Terracotta = primary accent / link / "this is the action". Sage = published / live / success. Gold = highlight. Everything else is paper or ink.

---

## 2. Design tokens

Drop these CSS variables into `:root` at the top of every page (or — preferred — extract into `assets/styles.css` and `<link>` it from every page; see section 9).

```css
:root {
  /* ---- Paper & ink ---- */
  --paper:       #f6efe1;   /* warm cream — base */
  --paper-2:     #fbf6ec;   /* lighter, for cards */
  --paper-3:     #efe5d0;   /* depressed surface, code blocks */
  --ink:         #2b1d12;   /* deep espresso — headings */
  --ink-2:       #6b5947;   /* muted body */
  --ink-3:       #a89580;   /* faint metadata */
  --rule:        #e3d6bf;   /* warm divider */
  --rule-2:      #d6c5a4;   /* stronger divider */

  /* ---- Accents (use sparingly) ---- */
  --accent:      #a85a2a;   /* terracotta — links, primary */
  --accent-soft: #d99c5e;   /* warm gold — highlights */
  --ok:          #6a7d5b;   /* sage — published / success */

  /* ---- Shadow ---- */
  --shadow: 0 1px 0 rgba(43,29,18,0.04), 0 12px 32px -18px rgba(43,29,18,0.25);

  /* ---- Type stack ---- */
  --serif: "Fraunces", "Iowan Old Style", "Apple Garamond", Georgia, "Times New Roman", serif;
  --sans:  "Inter", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
  --mono:  "JetBrains Mono", "SF Mono", "Fira Code", Menlo, monospace;
}
```

**Body background** has a faint two-tone grain — keep it:

```css
body {
  background: var(--paper);
  background-image:
    radial-gradient(rgba(168, 90, 42, 0.035) 1px, transparent 1px),
    radial-gradient(rgba(43, 29, 18, 0.025) 1px, transparent 1px);
  background-size: 24px 24px, 37px 37px;
  background-position: 0 0, 11px 13px;
}
```

---

## 3. Typography

Load these fonts in `<head>`:

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Fraunces:opsz,wght@9..144,300;9..144,400;9..144,500;9..144,600;9..144,700;9..144,800&family=Inter:wght@400;500;600&family=JetBrains+Mono:wght@400;500&display=swap">
```

### Type ramp

| Use | Family | Weight | Size | Line-height | Letter-spacing |
|---|---|---|---|---|---|
| Hero / cover H1 | Fraunces | 700 | clamp(48px, 7vw, 84px) | 0.98 | −0.025em |
| Article H1 (design page title) | Fraunces | 700 | clamp(36px, 4.5vw, 56px) | 1.05 | −0.02em |
| Section title (H2) | Fraunces | 600 | 32px | 1.2 | −0.015em |
| Subhead (H3) | Fraunces | 600 | 22px | 1.3 | −0.01em |
| Mini-head (H4) | Inter | 600 | 11px UPPERCASE | 1.2 | 0.18em |
| Dek / lede | Fraunces 300 italic | 300 | 20–22px | 1.4 | — |
| Body prose | Inter | 400 | 16px | 1.7 | — |
| Caption / meta | Inter | 500 | 12px | 1.5 | 0.04em |
| Kicker | Inter | 600 | 11px UPPERCASE | — | 0.22em |
| Numbers / code | JetBrains Mono | 500 | 13px | — | — |

**Rule of thumb:** if it's prose, sans. If it's a heading or a quote, serif. If it's a number or code, mono.

### Italic mark in headlines

Use italic Fraunces in the accent colour to lift one word in a headline — e.g. `URL <em>Shortener</em>`. Style:

```css
h1 em { font-style: italic; font-weight: 400; color: var(--accent); }
```

Use at most once per headline, never in subheads.

---

## 4. Layout

```css
.container { max-width: 1180px; margin: 0 auto; padding: 0 40px; }
```

- Homepage uses the full 1180px container.
- **Design pages and pattern pages** narrow to **840px** for the article body (or 720px for a tighter measure). Use a wider container only when the page has a right-rail TOC.
- Mobile (`@media (max-width: 900px)`): drop container padding to `0 24px`, collapse multi-column grids to single column.

### Spacing rhythm

| Token | Value | Use |
|---|---|---|
| xs | 8px | inline gaps, pill padding |
| sm | 16px | within-card padding |
| md | 28px | between cards in a grid |
| lg | 48px | between section header and content |
| xl | 72px | between sections (vertical padding inside `.section`) |

Sections separate with a 1px `var(--rule)` border, never a darker line.

---

## 5. Components

Each component lists: the HTML, the CSS (if not already in the homepage), and when to use it.

### 5.1 Masthead

Every page has the same masthead at the top, inside `.container`. Nav links change to be relative to the current page depth.

```html
<header class="masthead">
  <div class="masthead-row">
    <div>
      <a href="../index.html" style="text-decoration:none;">
        <div class="wordmark">System Design <span class="amp">&amp;</span> Trade-offs</div>
        <span class="wordmark-sub">a Filter Coffee Way notebook</span>
      </a>
    </div>
    <nav>
      <a href="../index.html#designs">Designs</a>
      <a href="../index.html#patterns">Patterns</a>
      <a href="../index.html#numbers">Numbers</a>
      <a href="../_topic-roadmap.html">Roadmap</a>
      <a href="https://github.com/filtercoffeeway/system-design" target="_blank" rel="noopener">GitHub</a>
    </nav>
  </div>
  <div class="issue-strip">
    <span>Vol. I &nbsp;·&nbsp; Issue 01</span>
    <span><a href="../index.html" style="color:var(--ink-3);">← back to cover</a></span>
    <span>MMXXVI</span>
  </div>
</header>
```

On a design page, the middle slot of `issue-strip` becomes "← back to cover" or a breadcrumb. On the homepage it shows "● Updated weekly".

### 5.2 Article header (for design + pattern pages)

Replaces the dark-mode page header. Use on every `designs/*.html` and `patterns/*.html` page.

```html
<header class="article-header">
  <div class="kicker">Issue 01 · Design</div>
  <h1 class="article-title">URL <em>Shortener</em></h1>
  <p class="article-dek">A read-heavy redirect at planet scale, designed around a single elegant constraint: never make the user wait.</p>
  <div class="byline">
    <span>Published <strong>25 May 2026</strong></span>
    <span class="sep">·</span>
    <span class="tag">Medium</span>
    <span class="tag">read-heavy</span>
    <span class="tag">caching</span>
    <span class="tag">base62</span>
  </div>
</header>
```

```css
.article-header { padding: 56px 0 40px; border-bottom: 1px solid var(--rule); margin-bottom: 48px; }
.article-title { font-family: var(--serif); font-weight: 700; font-size: clamp(36px, 4.5vw, 56px); line-height: 1.05; letter-spacing: -0.02em; color: var(--ink); margin: 8px 0 16px; }
.article-title em { font-style: italic; font-weight: 400; color: var(--accent); }
.article-dek { font-family: var(--serif); font-weight: 300; font-size: 21px; line-height: 1.4; color: var(--ink-2); max-width: 56ch; margin-bottom: 28px; }
```

The `.kicker`, `.byline`, and `.tag` rules already live in `index-new.html` — reuse verbatim.

### 5.3 Section heading

Use for every H2 inside an article. Pull the small italic Roman numeral from the homepage section heads.

```html
<h2 class="section-h2">
  <span class="section-num">i.</span> Requirements
</h2>
```

```css
.section-h2 {
  font-family: var(--serif);
  font-weight: 600;
  font-size: 28px;
  letter-spacing: -0.015em;
  color: var(--ink);
  margin: 56px 0 20px;
  padding-bottom: 12px;
  border-bottom: 1px solid var(--rule);
}
.section-h2 .section-num {
  font-family: var(--serif);
  font-style: italic;
  font-weight: 300;
  font-size: 18px;
  color: var(--accent);
  margin-right: 10px;
}
```

Numbering convention: lowercase Roman (`i.`, `ii.`, `iii.`, `iv.`…). Reset per page.

### 5.4 Body prose, lists, inline code

```css
article p { font-size: 16px; line-height: 1.75; color: var(--ink-2); margin: 0 0 18px; max-width: 62ch; }
article h3 { font-family: var(--serif); font-weight: 600; font-size: 20px; color: var(--ink); margin: 32px 0 12px; letter-spacing: -0.01em; }
article h4 { font-family: var(--sans); font-size: 11px; font-weight: 600; letter-spacing: 0.18em; text-transform: uppercase; color: var(--accent); margin: 24px 0 10px; }

article ul, article ol { margin: 0 0 18px 0; padding-left: 0; list-style: none; max-width: 62ch; }
article ul li, article ol li { position: relative; padding: 4px 0 4px 22px; font-size: 16px; line-height: 1.7; color: var(--ink-2); }
article ul li::before { content: "—"; position: absolute; left: 0; color: var(--accent-soft); font-weight: 500; }
article ol { counter-reset: olist; }
article ol li { counter-increment: olist; }
article ol li::before { content: counter(olist) "."; position: absolute; left: 0; color: var(--accent); font-family: var(--serif); font-weight: 500; font-size: 14px; }

article strong { color: var(--ink); font-weight: 600; }
article em { font-style: italic; color: var(--ink); }

code { font-family: var(--mono); font-size: 13.5px; color: var(--accent); background: var(--paper-3); padding: 1px 6px; border-radius: 3px; }
```

### 5.5 Code blocks

Warm paper, not dark. Soft border, monospace, slightly smaller than body.

```html
<pre><code>function shorten(longUrl) {
  const id = counter.next();
  return base62.encode(id);
}</code></pre>
```

```css
pre {
  background: var(--paper-3);
  border: 1px solid var(--rule);
  border-radius: 6px;
  padding: 18px 22px;
  overflow-x: auto;
  margin: 16px 0 24px;
  line-height: 1.55;
}
pre code { color: var(--ink); background: transparent; padding: 0; font-size: 13.5px; }
```

### 5.6 Tables

```css
article table { width: 100%; border-collapse: collapse; margin: 16px 0 24px; font-size: 14.5px; }
article th {
  text-align: left;
  font-family: var(--sans);
  font-size: 11px; font-weight: 600;
  letter-spacing: 0.18em; text-transform: uppercase;
  color: var(--accent);
  padding: 10px 14px;
  background: var(--paper-2);
  border-bottom: 1px solid var(--rule-2);
}
article td { padding: 12px 14px; border-bottom: 1px solid var(--rule); color: var(--ink-2); vertical-align: top; }
article tr:last-child td { border-bottom: none; }
article td:first-child { color: var(--ink); font-weight: 500; }
```

### 5.7 Callout

Replaces the old blue dark-mode callout. Three variants — pick by intent.

```html
<aside class="callout callout-note">
  <strong>Note —</strong> a single 64-bit counter on Zookeeper is enough for ~10²⁰ short URLs. Allocate ranges to app servers in chunks of 1M to avoid coordination on every write.
</aside>
```

```css
.callout {
  border-left: 3px solid var(--accent);
  background: var(--paper-2);
  border-radius: 0 6px 6px 0;
  padding: 14px 20px;
  margin: 20px 0 24px;
  font-size: 14.5px;
  line-height: 1.65;
  color: var(--ink-2);
  max-width: 62ch;
}
.callout strong { color: var(--accent); font-weight: 600; }
.callout-note   { border-left-color: var(--accent); }
.callout-tip    { border-left-color: var(--ok); }
.callout-tip strong { color: var(--ok); }
.callout-warn   { border-left-color: var(--accent-soft); }
.callout-warn strong { color: var(--accent-soft); }
```

Use `callout-note` for "remember this", `callout-tip` for "the elegant move", `callout-warn` for "this breaks at scale".

### 5.8 Pull quote

For a quotable line worth lifting out — use **at most once per article**.

```html
<blockquote class="pull-quote">
  Cache is not a database with worse durability. It is a different shape of correctness.
</blockquote>
```

```css
.pull-quote {
  font-family: var(--serif);
  font-style: italic;
  font-weight: 400;
  font-size: 26px;
  line-height: 1.35;
  color: var(--ink);
  margin: 36px 0;
  padding: 12px 0 12px 28px;
  border-left: 2px solid var(--accent);
  max-width: 56ch;
}
```

### 5.9 At-a-glance stat block

Already defined in `index-new.html` as `.stat-block`. Use on the design page above the article body when there are numbers that anchor the whole design (peak QPS, storage, latency target, read/write ratio).

### 5.10 Ornament divider (fleuron)

Use between major sections only when a hard visual pause is wanted — usually once before the footer.

```html
<div class="ornament-divider">
  <span class="line"></span>
  <span class="dot">❦</span>
  <span class="line"></span>
</div>
```

Already styled in `index-new.html`. Don't use more than once per page.

### 5.11 Prev / next footer (article pages)

At the bottom of every design / pattern page, before the site footer:

```html
<nav class="prev-next">
  <a class="pn-prev" href="../index.html">
    <span class="pn-label">Back to</span>
    <span class="pn-title">Cover</span>
  </a>
  <a class="pn-next" href="pastebin.html">
    <span class="pn-label">Next issue →</span>
    <span class="pn-title">Paste Bin</span>
  </a>
</nav>
```

```css
.prev-next {
  display: flex; justify-content: space-between; gap: 24px;
  padding: 40px 0;
  border-top: 1px solid var(--rule);
  margin-top: 64px;
}
.prev-next a { display: flex; flex-direction: column; gap: 4px; color: var(--ink); }
.pn-label { font-size: 11px; letter-spacing: 0.2em; text-transform: uppercase; color: var(--ink-3); font-weight: 600; }
.pn-title { font-family: var(--serif); font-size: 20px; color: var(--ink); }
.pn-next { text-align: right; }
.prev-next a:hover .pn-title { color: var(--accent); }
```

### 5.12 Footer

Same on every page. Copy the `footer` block from `index-new.html` verbatim. Adjust the relative link prefixes (`../` when one level deep).

---

## 6. Page scaffolds

### 6.1 Design page (`designs/<name>.html`)

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>URL Shortener — System Design</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Fraunces:opsz,wght@9..144,300;9..144,400;9..144,500;9..144,600;9..144,700&family=Inter:wght@400;500;600&family=JetBrains+Mono:wght@400;500&display=swap">
  <link rel="stylesheet" href="../assets/styles.css">  <!-- or inline if not using shared CSS -->
</head>
<body>
  <div class="container">

    <!-- masthead (5.1) — note relative paths use ../ -->

    <article>
      <!-- article header (5.2) -->

      <!-- optional at-a-glance stat block (5.9) — only if numbers anchor the design -->

      <h2 class="section-h2"><span class="section-num">i.</span> Requirements</h2>
      <h3>Functional</h3>
      <ul>...</ul>
      <h3>Non-functional</h3>
      <ul>...</ul>

      <h2 class="section-h2"><span class="section-num">ii.</span> Back-of-envelope</h2>
      <p>...</p>
      <pre><code>...</code></pre>

      <h2 class="section-h2"><span class="section-num">iii.</span> High-level design</h2>
      <p>...</p>
      <aside class="callout callout-note">...</aside>

      <h2 class="section-h2"><span class="section-num">iv.</span> Deep dives</h2>
      <h3>ID generation</h3>
      <p>...</p>
      <table>...</table>

      <h2 class="section-h2"><span class="section-num">v.</span> Trade-offs &amp; takeaways</h2>
      <p>...</p>

      <!-- ornament divider (5.10) — optional -->
      <!-- prev / next (5.11) -->
    </article>

    <!-- footer -->
  </div>
</body>
</html>
```

Article body wraps in `<article>` with `max-width: 840px; margin: 0 auto;` (set on the article element directly or via a wrapper inside `.container`).

### 6.2 Pattern page (`patterns/<name>.html`)

Same scaffold as a design page but with a different kicker — `<div class="kicker">Pattern · Reference</div>` — and no stat block. Patterns are shorter, more reference-shaped. Lead with a one-sentence definition in the dek, then sections for **When to use**, **Variants**, **Trade-offs**, **Where it shows up**.

### 6.3 Roadmap page (`_topic-roadmap.html`)

Single-column list, not a grid. Sections: **Published**, **In progress**, **Planned**, **Backlog**. Each entry is a row with title (serif, 20px), kicker tag (Issue NN), one-line description, and a status pill (sage for Published, terracotta for Drafting, ink-3 for Planned).

```html
<div class="roadmap-row">
  <div>
    <div class="kicker">Issue 01</div>
    <a href="designs/url-shortener.html"><h3>URL Shortener</h3></a>
    <p>Read-heavy redirect at planet scale. Counter ranges, base62, caching.</p>
  </div>
  <span class="pill badge-published">✓ Published</span>
</div>
```

```css
.roadmap-row { display: grid; grid-template-columns: 1fr auto; gap: 32px; align-items: center; padding: 24px 0; border-bottom: 1px solid var(--rule); }
.roadmap-row h3 { font-family: var(--serif); font-weight: 600; font-size: 22px; color: var(--ink); margin: 4px 0 6px; }
.roadmap-row p { font-size: 14.5px; color: var(--ink-2); max-width: 60ch; }
```

### 6.4 Design template (`templates/design-template.html`)

A skeleton design page with placeholders. Same as 6.1 but every section body says `<!-- TODO -->` or `Lorem section content.` so future-me can fill it in. Keep the kicker as `Issue NN · Design`.

---

## 7. Old → new mapping

When migrating an existing dark-mode page, find-and-replace by intent:

| Old (dark mode) | New (warm paper) |
|---|---|
| `background: #0f1117` (body) | `background: var(--paper)` + grain |
| `background: #161b27` (sidebar / cards) | `background: var(--paper-2)` |
| `background: #0d1117` (code) | `background: var(--paper-3)` |
| `color: #e2e8f0` (body text) | `color: var(--ink-2)` |
| `color: #f1f5f9` (headings) | `color: var(--ink)` |
| `color: #94a3b8` (muted) | `color: var(--ink-3)` |
| `color: #7dd3fc` (links / accent) | `color: var(--accent)` |
| `color: #4ade80` (success) | `color: var(--ok)` |
| `border: #1e293b` | `border: 1px solid var(--rule)` |
| Sidebar nav (240px left rail) | **Remove.** Replace with masthead nav (5.1). The article gets its full width back. |
| Old `.callout` (blue, `#0c1a2e`) | `.callout.callout-note` (5.7) |
| Old `<h2>` with bottom border | `.section-h2` with Roman numeral (5.3) |
| Old `<table>` (dark th `#1e293b`) | Warm-paper table (5.6) |
| Tags / pills (`background: #1e293b`) | `.tag` (terracotta-outlined pill — see homepage byline) |

**Drop the left sidebar entirely.** It was useful when there was nothing else, but it pinches the article body and adds noise. Top masthead + bottom prev/next is enough navigation.

---

## 8. Implementation checklist (per page)

When redesigning one existing page:

1. Read the current HTML to extract the **content only** (h1, dek/intro, sections, prose, lists, code, tables, callouts). Ignore old CSS — it's all being replaced.
2. Start from the scaffold in §6.1 (designs) or §6.2 (patterns).
3. Drop the content into the new scaffold. Convert:
   - Old `<h2>X</h2>` → `<h2 class="section-h2"><span class="section-num">i.</span> X</h2>` (number them).
   - Old callouts → §5.7.
   - Old tables → §5.6 styling applies automatically once tokens are in.
4. Pick one word in the H1 to italicise in terracotta (`<em>`) — usually the noun. If nothing reads naturally, skip it.
5. Add the kicker (`Issue NN · Design` or `Pattern · Reference`).
6. Fill in the byline tags with the topic's actual tags.
7. If the design has anchor numbers (QPS, latency, storage), add an at-a-glance stat block (§5.9) right after the article header.
8. Add prev/next at the bottom pointing to the previous and next issues (or back to cover if no neighbour exists yet).
9. Update the masthead nav so anchor links resolve to `../index.html#designs` etc. — they're not in the current file anymore.
10. Open the file in a browser at desktop and mobile widths. Check: paragraphs ≤ 62ch, headings aren't crammed, no horizontal scroll, code blocks don't bleed past the article width.
11. Update `index.html` if a "coming soon" card just turned into a "published" card. Update `_topic-roadmap.html` to match.

---

## 9. Shared stylesheet (recommended)

The existing pages each inline their CSS. That worked at 1 design but won't scale. Extract the full stylesheet to `assets/styles.css` and link it from every page:

```html
<link rel="stylesheet" href="../assets/styles.css">  <!-- adjust ../ depth -->
```

For the homepage it's just `assets/styles.css`. For a `designs/` or `patterns/` page it's `../assets/styles.css`.

Once extracted, future design tweaks happen in one file. If you'd rather keep CSS inline per page for portability, that's fine — just keep the tokens block at the top of every file identical.

`index-new.html` currently has everything inlined; on first migration sweep, lift its `<style>` block into `assets/styles.css` and reference it from every page.

---

## 10. Don'ts

- **No dark mode** anywhere — the paper aesthetic is the brand.
- **No emoji** in titles or nav. They worked in the old design (🏠 🗺) but clash with the editorial typography. The fleuron `❦` and the small `●` "live" dot are the only allowed decorative glyphs.
- **No left sidebar** on article pages. The masthead and prev/next replace it.
- **No more than 62ch** for any paragraph or list. Even on a wide screen.
- **No more than one italicised accent word per heading**, and never in subheads.
- **No multi-colour links.** Links are always terracotta. Don't introduce blue or any other hue.
- **No drop shadow on cards by default** — only on hover. The paper aesthetic dislikes elevation noise.
- **Don't reintroduce `.md` design or pattern files.** Per the project rules, all designs and patterns live as `.html` only. This `_design-system.md` is an exception — it's internal docs, not a published page.

---

## 11. File-by-file rollout plan

Order of operations once the new index is approved:

1. **Lift `index-new.html` styles to `assets/styles.css`.** Replace `index.html` with the linked version. Verify locally.
2. **`designs/url-shortener.html`** — first migration. This is the canonical example for future designs, so get it right. Refactor sections to Roman numerals, drop sidebar, add at-a-glance stat block, prev/next pointing to cover.
3. **`patterns/caching.html`**, then `token-generation.html`, then `database-selection.html`. Patterns are shorter; expect ~30 min each once URL Shortener sets the pattern.
4. **`_topic-roadmap.html`** — convert to the single-column row layout (§6.3).
5. **`templates/design-template.html`** — final, so it benefits from any tweaks made during the rollout.
6. Commit per file (`git commit -m "redesign: url-shortener"`) for a clean revert path if any single page goes sideways.

---

## 12. Quick reference — what to read first

If you're Sonnet picking this up cold:

1. Open `index-new.html` — read the whole file. That's the reference.
2. Read §2 (tokens) and §3 (typography) here.
3. Pick a page to redesign. Find its scaffold in §6.
4. Use §7 as the find-and-replace cheatsheet for old → new colours and structures.
5. Run through §8 before declaring the page done.

Everything else in this doc is reference; you don't need to read it linearly.
