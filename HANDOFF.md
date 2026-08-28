# Handoff

Picking this up cold? Read in this order:

1. **This file** — where things stand, what's verified, what isn't, and what
   will bite you.
2. **[README.md](README.md)** — the rule, the API, the colour and accessibility
   notes.
3. **The two Figma files** — linked in the README. Everything here was read out
   of them; nothing was guessed.

Repo: <https://github.com/camila-go/responsiveGridDocumentation> (`origin`, SSH).

---

## ⚠️ Before this repo goes anywhere public

`cta.jpg` is a licensed brand photograph of an identifiable person, exported
from Responsive layout 2.0. It is committed to this repo.

- If the repo is **public**, that image is public. I could not check visibility
  from this machine (no `gh` CLI), so **verify it.**
- If that isn't intended, swap it for a placeholder and rewrite history — it
  stays recoverable from the commit otherwise. Doing that now is much easier
  than after the repo has forks or clones.
- The photo is decorative in the markup (`alt=""`) — no copy depends on it — so
  substituting a placeholder costs nothing but the demo's realism.

---

## Status

| Area | State |
| --- | --- |
| Width system (`.l-full` / `.l-container` / `.l-max` / `.l-min`) | Built; measured in-browser at 7 widths against the Figma rectangles |
| Flex grid (`--cols`, `--span`, wrap behaviour) | Built; column maths verified at 5 configurations |
| Grid overlay toggle | Built; label, `aria-pressed` and class flip all verified |
| CTA component | Rebuilt from node 1:180; 1920 frame produces a 1440 × 500 CTA, matching the artboard |
| Container-query tiers | Built; **tier breakpoints are mine, not spec'd** — see Assumptions |
| Contrast | Measured by compositing the scrim over real image pixels; one genuine failure found and fixed |
| Mobile / reflow | Verified at 320 and 375: no page-level horizontal scroll, grids collapse to one column, CTA stacks. Wide content (the reference table) scrolls inside its own wrapper |
| Screen reader testing | **Not done.** Reasoned about the a11y tree; never ran VoiceOver or NVDA |
| Cross-browser | **Not done.** Only exercised in the preview pane's Chromium |
| Real device testing | **Not done.** All responsive checks were emulated viewports |

**Four sections of the page are currently hidden**, not deleted: Content
widths, Flex grid, How a component responds, and Reference. They carry a
`hidden` attribute on their `<section>` — remove it to bring one back. The page
is presenting as a prototype of the grid itself, so it shows only the hero and
the rail diagram. Everything documented below still exists in the file and
still works; you just won't see it on screen.

Nothing is uncommitted. Everything described here is on `origin/main`.

---

## Running it

Static files, no build step, no Node required.

```bash
python3 -m http.server 4321
```

Then <http://localhost:4321>.

Opening `index.html` straight off the filesystem works. The only thing that
needs a real origin is the contrast-measuring console snippet under
Accessibility, which reads image pixels back off a canvas.

> On the machine this was built on, the macOS sandbox blocks serving from
> `~/Documents`, so the preview ran from a `/tmp` mirror registered as
> `grid-documentation` on port 4321 in the parent workspace's
> `.claude/launch.json`. That file is **outside this repo** and deliberately not
> committed — it's local tooling, not part of the deliverable.

---

## The one rule

Everything in the width system falls out of two lines:

```
pattern-max = min(page − 2 × padding, 1440)
pattern-min = pattern-max × 0.8125        # 1170 / 1440
```

**1440 is the default content width**, and the padding sits outside that rail
rather than inside it — so content reaches 1440 once the viewport is 1504
(1440 + 2×32) or wider. 1504 is a consequence of the padding token, not a
design number: never hardcode it, and never quote it as "the default width".

The numbers written on the Figma Grid page (1440, 1170, 1600) are **that
formula evaluated at 1920px**, not independent constants. Treating them as
fixed values is the single most likely way to break this.

Verified output — every row matches a measured rectangle in the source files:

| page | padding | content (`.l-max`) | narrow (`.l-min`) |
| --- | --- | --- | --- |
| 1920 | 32 | 1440 | 1170 |
| 1504 | 32 | 1440 | 1170 |
| 1440 | 32 | 1376 | 1118 |
| 1024 | 32 | 960 | 780 |
| 961 | 32 | 897 | 729 |
| 960 | 16 | 928 | — |
| 753 | 16 | 721 | — |
| 375 | 16 | 343 | — |

Below the breakpoint `.l-min` resolves to `.l-max`, so there is no separate
narrow width on mobile.

---

## Decisions that departed from what's written in Figma

Each of these is a case where the file's literal text is stale or frame-specific.
They are the things most likely to cause an argument in review, so they're
listed with the reasoning rather than buried.

| Decision | Why |
| --- | --- |
| **1440 is the default; the 1600 on the Grid page is stale** | 1600 was 1440 + 2×80 — the old padding. With 32px the equivalent total is 1504, but that number is derived and deliberately **not** headlined: 1440 is the default content width and padding sits outside it. **Any redline still saying 1600 is stale.** |
| **Desktop starts at 961, not 960** | The spec table says "Desktop 960–1920", but the 960px artboards in *both* files are drawn with the 16px mobile padding. The artboards win. |
| **`padding DT 1027` is the authoritative 1024 study** | It draws 1024 → 960 / 780, exactly what the formula produces at 32px. The `padding DT 1024` frame beside it (80px → 864 / 702) is the superseded option. |
| **CTA crop uses `scale(1.45) translateX(-15%)`** | Figma positions the subject with `-38.07% / -71.54%` at `138.87% / 298.54%`. Those are pinned to the 1920 × 500 frame and break at every other aspect ratio. |
| **`.l-min` does not apply below 960** | At 375 the narrow width would be a 279px column inside 343px of space — 32px of dead gutter each side on top of the page padding. Below the breakpoint `.l-min` resolves to `.l-max`, and the overlay stops drawing a separate min band. The `usage M` frame in the Grid page *does* draw a 754px narrow band at 960, so this is a product decision, not a reading of the file. |
| **CTA scrim has a flat 0.18 floor added** | The design's radial alone measured 3.96:1 behind the 16px copy. See Accessibility. |
| **Text column is 468px, not Figma's 346px** | Acumin Extra Condensed isn't loaded; the fallback condensed faces are wider and the headline wrapped. Point `--font-display` at the real Typekit face and this can come back down. |

---

## Assumptions that are not spec

Do not treat these as decided. Nothing in either Figma file specifies them.

| Thing | Current value | Notes |
| --- | --- | --- |
| Gutter | `--grid-gutter: 24px` | 3 × the system's 8px base unit. Pure inference. |
| Column count | none — `--cols` per grid | The Grid page defines widths and padding only; there is no 12-column system in the file. |
| Container-query tiers | 480 / 860 / 1200 | Chosen to match where the CTA visibly rearranges across the artboards. Per-component, **not** system tokens. |
| Stacked CTA layout | portrait crop over a light panel | Inferred from the narrow column in Responsive layout 2.0. The text nodes there sit outside the CTA frame, so the exact composition is not pinned down. |

---

## Accessibility

Ratios below were **measured**, not estimated — the CTA figures by compositing
the scrim over the actual image pixels behind each text box and taking the
worst of 225 samples.

The real finding: **the Figma CTA scrim was failing.** Its radial gradient was
authored against one 1920 × 500 crop; at other widths the text drifts over
lighter parts of the photograph.

| Text | Threshold | Figma scrim only | With 0.18 floor |
| --- | --- | --- | --- |
| Headline, 48px 600 (large) | 3:1 | 4.31:1 pass | 5.94:1 |
| Copy, 16px 400 (normal) | 4.5:1 | **3.96:1 fail** | 5.51:1 |

> **If the photograph is ever swapped, re-measure.** 0.18 is tuned to this
> image, not derived from a principle.

To re-measure, serve the page and paste this into the console. It composites
the scrim over the real image pixels behind a text box and reports the worst
sample, which is the number that has to clear the threshold:

```js
const srgb = c => (c /= 255) <= 0.03928 ? c/12.92 : ((c+0.055)/1.055)**2.4;
const lum  = (r,g,b) => 0.2126*srgb(r) + 0.7152*srgb(g) + 0.0722*srgb(b);
const FLOOR = 0.18;                              // .cta__scrim's flat layer

const cta    = document.querySelector('#slot-max .cta');   // must be overlay tier
const img    = cta.querySelector('.cta__img');
const mediaR = cta.querySelector('.cta__media').getBoundingClientRect();
const cv = Object.assign(document.createElement('canvas'),
                         {width: img.naturalWidth, height: img.naturalHeight});
cv.getContext('2d').drawImage(img, 0, 0);
const ctx = cv.getContext('2d');

const worst = el => {
  const t = el.getBoundingClientRect();
  let out = Infinity;
  for (let i = 0; i <= 24; i++) for (let j = 0; j <= 8; j++) {
    const nx = (t.left - mediaR.left + t.width  * i/24) / mediaR.width;
    const ny = (t.top  - mediaR.top  + t.height * j/8 ) / mediaR.height;
    const ix = Math.round(nx * cv.width);
    const iy = Math.round(ny * cv.height * 0.55 + cv.height * 0.18);
    if (ix < 0 || iy < 0 || ix >= cv.width || iy >= cv.height) continue;
    const d = ctx.getImageData(ix, iy, 1, 1).data;
    // scrim: ellipse 36.7% x 145% at 50% 65.6%, 0.6 -> 0, then the flat floor
    const ex = (nx - 0.5) / 0.367, ey = (ny - 0.656) / 1.45;
    const a  = 1 - (1 - 0.6 * (1 - Math.min(1, Math.hypot(ex, ey)))) * (1 - FLOOR);
    const L  = lum(...[d[0], d[1], d[2]].map(c => c * (1 - a)));
    out = Math.min(out, 1.05 / (L + 0.05));      // vs white text
  }
  return +out.toFixed(2);
};

console.table({
  headline: {ratio: worst(cta.querySelector('.cta__title')), needs: 3.0},
  copy:     {ratio: worst(cta.querySelector('.cta__copy')),  needs: 4.5},
});
```

Needs a real origin — `file://` blocks the canvas readback. Set `FLOOR = 0`
to see what the Figma scrim does on its own.

**Expect different numbers at different window sizes**, and don't read that as
a discrepancy. The crop and the text position both move with width, so the
worst-case pixel changes. The table above is measured at a 1216px CTA; the same
snippet at a 606px CTA returns 4.59 and 4.80 — still passing, but with much
less headroom. Check at a few widths, and judge by the lowest.

Also fixed, and worth not regressing:

- **Diagram labels are chips**, not bare text on the rails. A label is wider
  than its own rail at most viewports, so it always crosses onto peach or cyan
  — bare text was ~1.3:1 there. Chips are 12.3:1, with a colour dot carrying the
  rail identity so colour is no longer the only differentiator.
- **Scaled preview frames are `inert`.** They're pictures of the component,
  rendered at up to half size, whose links duplicate the full-size copies below.
  Without `inert` you tab through six unreachable-looking buttons first.
- **Focus rings are two-tone** (white outline, ink outer shadow). No single
  colour clears 3:1 against white, the dark HUD *and* the photograph.
- **`.cta__title` is a `<p>`, deliberately.** It should be an `<h2>` in a real
  page; the demo renders the CTA eight times and eight identical headings would
  be worse than none. **Change it when you lift the component out.**

---

## Gotchas

Things that already caused a bug once, or will.

1. **Width classes must be a direct child of a full-width band.** They're
   percentages of the containing block. Nest `.l-max` inside another `.l-max`
   and you get 1440 of 1440, not 1440 of the page.

2. **`--item-min` defaults to `0` on purpose.** It started at `280px`, which
   silently overrode the column count — a `cols-4` grid became 3 columns plus a
   full-width orphan at any width under ~1400. At `0`, `--cols` is honoured
   exactly and the 960px breakpoint is the only column change in the system,
   which is what both files depict. Raise it per-grid if you want
   content-driven wrapping, and pair it with `.flex-grid--even` so the last row
   doesn't stretch.

3. **`.flex-grid--keep` makes a grid non-responsive. That is its whole job.**
   It opts out of the single-column collapse below 960px, so the columns stay
   put no matter how narrow the page gets. On the demo page's three-slot grid
   that produced **98px cells at 375px** — three CTAs crushed side by side.
   Only use `--keep` on grids of genuinely small items (chips, thumbnails,
   stat tiles), and pair it with `--item-min` so they still wrap. Never on
   cards or anything with a button in it.

4. **`.cta__img` must stay `position: absolute` in the overlay tier.** In flow,
   the photograph's intrinsic aspect drives the grid row height and a 1440-wide
   CTA renders ~806px tall instead of 500.

5. **`object-position` X does nothing when the box is wider than the image's
   aspect ratio.** `cover` crops only on the constrained axis. That's why the
   subject is placed with a transform, not `object-position`.

6. **Percentage widths can go negative** below ~64px page width. Guarded with
   `max(0px, …)` in CSS. Not reachable in a real browser, but it surfaced when
   the preview pane reported a 0px viewport.

---

## Public API

Everything in `grid.css`. No build step, no preprocessor, no dependencies.

**Layout**

| Selector | Purpose |
| --- | --- |
| `.grid-page` | Page shell; caps at max supported width and centres |
| `.grid-band` | Full-bleed horizontal band; backgrounds live here |
| `.l-full` | Full width content, edge to edge |
| `.l-container` | Padded shell — same 1440 rail, padding inside the box so a background spans the band |
| `.l-max` | Max pattern width — the default content width |
| `.l-min` | Narrow content width — desktop only; resolves to `.l-max` below 960 |
| `.grid-overlay` | Debug: draws the rails over the live page |

**Flex grid**

| Selector / property | Purpose |
| --- | --- |
| `.flex-grid` | The grid; set `--cols` and optionally `--item-min` |
| `.cols-1 … .cols-12` | Nominal column count |
| `.span-1 … .span-6` | Child spans N nominal columns plus gutters |
| `.flex-grid--even` | Items hold column width; last row left-aligns |
| `.flex-grid--keep` | Opt out of the single-column collapse below 960px |

**Tokens**

```
--grid-max-supported   1920px
--grid-pattern-max     1440px      cap, not a floor
--grid-pattern-min     1170px
--grid-pattern-ratio   0.8125
--grid-pad-mobile      16px
--grid-pad-desktop     32px
--grid-pad             resolved per breakpoint
--grid-gutter          24px        assumption, see above
--grid-rail-outside / -padding / -max / -min / -alpha        overlay colours
```

---

## Dropping this into an existing project

`CU-Home-Page-Test/css/tokens.css` already carries these two rails as hardcoded
values:

```css
--max-content: 1440px;      /* == --grid-pattern-max */
--tiles-max-width: 1170px;  /* == --grid-pattern-min */
```

They are the same two numbers. Folding that project onto this system is mostly
a matter of pointing those tokens at `--grid-pattern-max` / `--grid-pattern-min`
and replacing bespoke `width: min(...)` rules with `.l-max` / `.l-min`. Watch
for its existing `1280px` / `1024px` / `768px` media queries — they predate the
single-breakpoint model and will fight it.

---

## What I'd do next

In rough priority order:

1. **Settle the `cta.jpg` licensing question** (top of this file).
2. **Confirm the gutter.** 24px is inference. It's the one number in daily use
   that nobody has actually specified.
3. **Run a screen reader pass.** The a11y work here is measured for contrast and
   reasoned for structure, but never driven by an actual AT.
4. **Load Acumin** and pull the CTA text column back toward Figma's 346px.
5. **Update the Figma Grid page**, which still writes 1600 as the total default
   desktop width. That was 1440 + 2×80 and no longer matches the 32px padding.
   The code doesn't depend on it, but the file will keep misleading readers.
