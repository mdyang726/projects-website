# Web Design Guidelines

General best practices for building and reviewing pages on this site. These are
craft standards, not project facts — the site's own tokens, fonts, and block
patterns live in `CLAUDE.md` under "Design system". When the two overlap, the
`CLAUDE.md` values win; this file explains the *why* and fills the gaps.

Use it as a checklist when adding or editing a page, and as a review rubric when
inspecting a page in the browser (Playwright MCP / Claude-in-Chrome).

---

## 1. Layout & spacing

- **One measure for reading.** Body text lines should run ~60–75 characters.
  The `#wrapper` max-width of 860px with 80px side padding already lands here;
  don't widen prose containers past that.
- **Space on a scale.** Pick spacing from a consistent set (e.g. 4 / 8 / 12 / 16
  / 24 / 32 / 48 / 80). Avoid one-off values like `padding: 17px`.
- **Vertical rhythm.** Space between blocks should be larger than space within a
  block, so structure reads at a glance. Section gaps > paragraph gaps > line
  height.
- **Align to a grid.** Left edges of headings, body copy, labels, and figures
  should line up. Indentation should be deliberate, not incidental.
- **Group by proximity.** A label belongs closer to the thing it labels than to
  the previous block. Whitespace is the primary grouping tool — use it before
  borders or boxes.
- **Don't center long text.** Center only short headings or single lines; left-
  align anything longer than ~2 lines.

## 2. Typography

- **Three roles, no more.** Headings (DM Serif Display), labels/tags (DM Mono),
  body (DM Sans). Don't introduce a fourth family.
- **Limited size steps.** Use a small type scale (e.g. 0.8 / 1 / 1.25 / 1.6 /
  2 / 2.5 rem). Every distinct size should mean something.
- **Line height by size.** Body ~1.5–1.65; headings tighter, ~1.1–1.25.
- **Line length over font size** for readability — see the measure rule above.
- **Weight and case for emphasis**, not italics-on-italics or ALL CAPS runs
  longer than a few words. Mono labels may be uppercase with slight letter-
  spacing (~0.04em).
- **Hyphenation / wrapping.** Prevent orphaned single words on headings where it
  looks awkward; never `hyphens: auto` on display headings.
- **Real punctuation.** Curly quotes, en/em dashes, `×` for dimensions, non-
  breaking spaces between a number and its unit (`8.23&nbsp;s`).

## 3. Color & contrast

- **Stay in the palette.** bg `#fafaf9` · ink `#1a1a18` · body `#5f5e5a` ·
  muted `#b4b2a9` · rule `#e0dfd8` · hover `#f1efea` · badge `#FBEFD8` /
  `#92600B`. New colors need a real reason.
- **Contrast minimums (WCAG AA):** body text ≥ 4.5:1 against its background;
  large text (≥ 24px, or ≥ 19px bold) and UI borders ≥ 3:1. `muted #b4b2a9` on
  `bg #fafaf9` is ~2:1 — fine for hairlines and decorative marks, **not** for
  text you expect people to read.
- **Never rely on color alone** to carry meaning (links, status, chart series):
  back it with underline, icon, label, or shape.
- **Links** should be visually distinct from body text in a resting state, not
  only on hover.

## 4. Responsive

- **Content-driven breakpoints.** Add a breakpoint where the layout breaks, not
  at fixed device widths. This site's known one is 640px (side padding drops
  80px → 24px).
- **Test the full range:** 320, 375, 414, 640, 768, 1024, 1440. Nothing should
  overflow horizontally; no horizontal scrollbar on the page body.
- **Fluid images:** `max-width: 100%; height: auto`. Galleries reflow to fewer
  columns, they don't shrink each cell to unreadable.
- **Tap targets ≥ 44×44px** on touch; spacing between adjacent targets.
- **No fixed pixel heights** on text containers — let content set the height.
- **Respect `prefers-reduced-motion`**: gate transitions/animations behind it.

## 5. Accessibility

- **Semantic HTML first.** One `<h1>` per page; headings nest without skipping
  levels. Use `<nav>`, `<main>`, `<section>`, `<figure>/<figcaption>`, `<ul>`,
  `<button>` vs `<a>` correctly.
- **Every image has `alt`.** Decorative → `alt=""`. Informative → describe the
  point. Data figures (charts, plots) → describe what the data shows, not just
  "chart".
- **Keyboard.** All interactive elements reachable and operable by Tab/Enter;
  visible focus outline (never `outline: none` without a replacement).
- **Link text is meaningful out of context** — not "click here" / "read more".
- **Color-independent** (see §3). Don't conflate placeholder text with labels.
- **External links:** `target="_blank" rel="noopener"`; consider noting "(opens
  in new tab)" for screen-reader users when it matters.
- **Language:** `<html lang="en">`.

## 6. Performance

- **Images are the budget.** Serve appropriately sized files; prefer modern
  formats (WebP/AVIF) with a fallback where practical. Compress — a portfolio
  photo rarely needs to be > 250 KB.
- **`loading="lazy"`** on below-the-fold images; **eager** on the hero/LCP image.
- **Set `width`/`height`** (or `aspect-ratio`) on images to reserve space and
  avoid layout shift (CLS).
- **Fonts:** subset to the weights/styles actually used; `font-display: swap`;
  preconnect to the font host. Don't ship six weights to use two.
- **No blocking JS** — this is a static Astro site with no client framework;
  keep it that way unless a task explicitly calls for interactivity.
- **Target:** Lighthouse ≥ 95 across Performance / Accessibility / Best
  Practices / SEO on a mid-tier mobile profile.

## 7. Images & figures

- Put files in `public/images/<project-slug>/`, reference absolute (`/images/...`).
- Use `<figure>` + `<figcaption>` for captioned images; caption is body-color,
  smaller, sits directly under the image.
- Consistent aspect ratios within a gallery row; consistent gutters.
- Don't upscale — a 600px-wide source shouldn't be displayed at 900px.

## 8. Motion & interaction

- Motion should clarify, not decorate. Durations 120–250ms for UI feedback;
  ease-out for entrances.
- Hover states use the `hover #f1efea` token; provide an equivalent focus state.
- Avoid parallax, autoplay video with sound, and scroll-jacking.
- Honor `prefers-reduced-motion: reduce` — drop non-essential animation.

## 9. Content & voice

- Copy comes from `context/*.md` and from the user. Never invent project facts,
  dates, or numbers.
- Front-load the point of each section; short paragraphs; one idea per block.
- Sentence case for headings unless a proper noun says otherwise.
- Numbers: consistent units and precision within a page; thin/non-breaking space
  before units.

## 10. Pre-ship checklist

- [ ] `npm run build` clean; `npm run preview` looks right
- [ ] Checked at 320 / 640 / 768 / 1024 / 1440 — no horizontal overflow
- [ ] One `<h1>`, headings nest, landmarks present
- [ ] Every image has meaningful `alt`; hero eager, rest `loading="lazy"`
- [ ] Images have dimensions/aspect-ratio; no visible layout shift on load
- [ ] Body text contrast ≥ 4.5:1; muted used only for hairlines/decoration
- [ ] Keyboard tab order sane; focus visible
- [ ] External links `target="_blank" rel="noopener"`
- [ ] Colors, fonts, spacing all from the documented system
- [ ] LF line endings; indentation/quote style matches the file
