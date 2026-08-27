# Grid Documentation

Responsive grid built from two Figma files, which agree on one rule set.
Source file names as they appear in Figma:

- **Capstone Design System 0.9.1** — [Grid, node 2986:266](https://www.figma.com/design/eovyjwT8tOlkTydJutIwOP/Capstone-Design-System-0.9.1?node-id=2986-266)
- **Responsive layout 2.0** — [Page 1, node 0:1](https://www.figma.com/design/LGrgzsayH1G2mIvNg3WBme/Responsive-layout-2.0?node-id=0-1)

No dependencies: [`grid.css`](grid.css) is the system, [`index.html`](index.html) is a live
spec page, [`cta.jpg`](cta.jpg) is the CTA photograph exported from Figma
(downsized from the 4096px original to 1920px / 445K).

## The rule

| | |
|---|---|
| Single breakpoint | **960px** — mobile ≤ 960, desktop ≥ 961 |
| Mobile padding | **16px** fixed |
| Desktop padding | **32px** fixed |
| Max pattern width | **1440px** (a cap, not a floor) |
| Min pattern width | **1170px** |
| Total default desktop width | **1504px** (1440 + 2 × 32) |
| Max supported width | **1920px** |

Between the breakpoint and the cap nothing steps — the pattern widths scale with
the browser ("Scales w/o breakpoint RELATIVE TO BROWSER WIDTH"). The quoted
numbers are that scale evaluated at 1920px:

```
pattern-max = min(page − 2 × padding, 1440)
pattern-min = pattern-max × 0.8125          # 1170 / 1440 = 0.8125
```

That single pair reproduces **every** measured rectangle in both files:

| page | padding | container | pattern-max | pattern-min | drawn in Figma as |
|---|---|---|---|---|---|
| 1920 | 32 | 1504 | 1440 | 1170 | max supported width |
| 1504 | 32 | 1504 | 1440 | 1170 | total default desktop width |
| 1024 | 32 | 1024 | 960 | 780 | `padding DT 1027` |
| 961 | 32 | 961 | 897 | 729 | ">961 – just before small breakpoint" |
| 960 | 16 | 960 | 928 | 754 | `padding M 960`, `usage M` |
| 753 | 16 | 753 | 721 | 585 | "<961 – after mobile breakpoint" |
| 375 | 16 | 375 | 343 | 279 | `padding M 375` |

Verified in-browser against every row.

### On the 32px decision

Padding is 32 on desktop, 16 on mobile. Two consequences worth knowing:

- It makes the **`padding DT 1027`** frame in the Grid page the authoritative
  1024px study — it draws 1024 → 960 / 780, exactly what the formula produces.
  The `padding DT 1024` frame beside it (80px → 864 / 702) is superseded.
- **Total default desktop width is 1504, not the 1600 written on the Grid page.**
  1600 was 1440 + 2×80. `.l-container` derives it from `--grid-pad-desktop`
  rather than hardcoding, so it tracks the token. If any spec text or redline
  still says 1600, that number is stale.

## Usage

```html
<link rel="stylesheet" href="grid.css">

<div class="grid-page">
  <section class="grid-band">          <!-- full-bleed band, background lives here -->
    <div class="l-max">…</div>          <!-- max pattern width -->
  </section>
  <section class="grid-band">
    <div class="l-min">…</div>          <!-- min pattern width -->
  </section>
  <section class="grid-band">
    <div class="l-full">…</div>         <!-- full width content, edge to edge -->
  </section>
</div>
```

Widths are percentages of the containing block, so a width class must sit
directly inside a full-width band.

| Class | Width at ≥1504px |
|---|---|
| `.l-full` | 1920 — full width content |
| `.l-container` | 1504 — padded shell, for page chrome (header, footer, nav) |
| `.l-max` | 1440 — default content width |
| `.l-min` | 1170 — narrow content width |

### Flex grid

```html
<div class="flex-grid cols-4">
  <div class="span-2">…</div>
  <div>…</div>
  <div>…</div>
</div>
```

Columns are nominal, not structural — there is no fixed track, so items keep
filling the row after a wrap.

- `cols-1 … cols-12`, or set `--cols` directly.
- `span-1 … span-6` on a child, or set `--span`.
- `--item-min` (default `0`) turns on content-driven wrapping. At the default,
  `--cols` is honoured exactly and the 960px breakpoint is the only column
  change in the system — matching the design files, which show no intermediate
  column steps.
- `.flex-grid--even` stops items growing, so a wrapped last row sits
  left-aligned instead of stretching to fill.
- `.flex-grid--keep` opts out of the single-column collapse below 960px.

### Components respond to their slot, not the viewport

The grid decides how much room a component gets; the component decides what to
do with it. So components size off **their own width** with container queries,
not off viewport media queries — that way one block of markup is correct at
`.l-max`, at `.l-min`, and inside a three-column grid cell, with no variant
classes and no knowledge of where it was placed.

The demo component is the CTA from Responsive layout 2.0 ([node 1:180](https://www.figma.com/design/LGrgzsayH1G2mIvNg3WBme/Responsive-layout-2.0?node-id=1-180)),
built to the real spec: full-bleed photograph, centred white copy over a radial
scrim, and two 60px pill buttons — `Apply now` filled white on `#212322`,
`Get details` outlined 2px white — with a 24px gap.

It has two arrangements, both container-driven:

| Container | Arrangement |
|---|---|
| < 480px | **stacked** — portrait crop above a light panel, ink text, buttons stacked |
| ≥ 480px | **overlay** — text on the image, inline buttons, 380px tall |
| ≥ 860px | overlay, headline at the full 48px, 440px tall |
| ≥ 1200px | overlay, 500px tall — the height every desktop artboard uses |

The spec page demonstrates this two ways:

1. **Viewport scopes** — the same CTA rendered in real 1920 / 1024 / 375 pages,
   each scaled down to fit so all three are legible at once. Verified: the 1920
   frame produces a 1440 × 500 CTA, matching the artboard exactly.
2. **Three slots, one viewport** — the same CTA dropped into `.l-max`, `.l-min`,
   and a three-column grid without changing viewport at all. At a 1280px window
   that is 1216px → overlay at 500px, 988px → overlay at 440px, and a ~389px
   grid cell → stacked.

The 960px breakpoint stays a viewport concern (it switches page padding). Every
component-level change is container-driven.

Two deliberate departures from the Figma file, both because its values are tied
to one specific 1920 × 500 frame:

- **The crop.** Figma places the subject left of the copy with an explicit
  `-38.07% / -71.54%` offset at `138.87% / 298.54%`. Those percentages break at
  any other aspect ratio, so the overlay tier uses `scale(1.45) translateX(-15%)`
  — the same move, expressed proportionally.
- **The scrim.** See below; the design's radial alone fails contrast at other
  widths.

The headline is set in Acumin VF Extra Condensed in Figma. That face isn't
loaded here, so the stack falls back to `Archivo Narrow` / `Arial Narrow`, which
are wider — the text column is 468px rather than Figma's 346px to keep the
headline on one line. Point `--font-display` at the real Typekit face and the
column can come back down.

### Grid overlay

Add `.grid-overlay` to `.grid-page` to draw the rails from the Figma diagram over
the live page, in the Figma fills. The spec page has a toggle for it.

## Colour

Rail and annotation colours are taken from the Figma "Grid" page, not invented:

| Token | Value | Role |
|---|---|---|
| `--grid-rail-outside` | `#000000` | outside the container |
| `--grid-rail-padding` | `#ffe0ae` | page padding |
| `--grid-rail-max` | `#aef3ff` | max pattern width |
| `--grid-rail-min` | `#edaeff` | min pattern width |

The spec page chrome uses the source tokens: `careerblue-500 #253746`,
`careerblue-200 #a8afb5`, `gl-neutral-600 #383838` (the annotation grey),
`Primary/University Grey #d9d9d6`, and the `image_component` placeholder
(`#e0e0e0` on `#c2c2c2`, 20px radius).

Headlines are set in Inter. The brand-campaign headings in Figma use Acumin VF
Extra Condensed, which isn't loaded here — swap `--font-display` in whichever
project this lands in.

## Accessibility

Contrast ratios below were measured in the browser, not estimated — the CTA
figures by compositing the scrim over the actual image pixels behind each text
box and taking the worst sample of 225.

**The scrim was failing.** Figma's CTA scrim is a radial gradient from
`rgba(0,0,0,0.6)` to transparent. It was authored against one 1920 × 500 crop;
at other widths the text block drifts over lighter parts of the photograph where
the gradient has already fallen off:

| Text | Threshold | Figma scrim only | With 0.18 floor |
|---|---|---|---|
| Headline, 48px 600 (large) | 3:1 | 4.31:1 pass | 5.94:1 |
| Copy, 16px 400 (normal) | 4.5:1 | **3.96:1 fail** | 5.51:1 |

So `.cta__scrim` layers a flat `rgba(0,0,0,0.18)` under the radial. The radial
still does the visible work; the floor guarantees the worst case. This is a
knowing deviation from the Figma value — if the photograph is ever swapped,
re-measure rather than assuming 0.18 is still enough.

**Other fixes:**

- **Diagram labels.** They were bare text on the rails — white on the black
  band, which is fine until the label outgrows its own rail and crosses onto
  peach (`#ffe0ae`) and cyan (`#aef3ff`) at ~1.3:1. Labels are now solid chips
  at 12.3:1, with a colour dot carrying the rail identity instead of the text
  colour. Colour is no longer the only thing distinguishing a rail — each is
  named.
- **Focus.** A two-tone ring (white outline, ink outer shadow) so it stays
  visible on white, on the dark HUD, and on the photograph. No single colour
  clears 3:1 against all three.
- **Duplicate interactive content.** The three scaled preview frames are
  pictures of the component, rendered at up to half size, whose links duplicate
  the full-size copies below. They're `inert`, so keyboard and screen reader
  users don't tab through six unreachable-looking buttons first.
- **Heading order.** The kicker above the `<h1>` was an `<h3>`. Now a paragraph.
- **Borders.** The outlined button on the light panel takes an ink border (the
  design's white would be invisible); the HUD button border went `#4a5a68` →
  `#93a2ae` to clear 3:1.
- **Reflow.** The reference table scrolls inside its own wrapper so the page
  doesn't scroll sideways at 320px.
- **The photograph is decorative** — all the meaning is in the adjacent text —
  so it carries `alt=""` and stays out of the accessibility tree, matching how
  Figma marks it `aria-hidden`.

One thing I did **not** change: `.cta__title` is a `<p>`, not an `<h2>`. In a
real page it should be a heading; here the demo renders the CTA eight times and
eight identical `<h2>`s would be worse than none. Swap it when you lift the
component out.

## Remaining open questions

- **The spec table says "Desktop 960–1920"**, but the 960px artboards in both
  files are drawn with the 16px mobile padding. The artboards win here: desktop
  starts at 961.
- **Column count and gutter are not specified anywhere** in either file — the
  Grid page defines widths and padding only. `--grid-gutter` defaults to
  **24px** (3 × the system's 8px base unit). This is an assumption, not a spec.
- **Container-query tiers (560 / 860 / 1200) are mine**, chosen to match where
  the CTA in Responsive layout 2.0 visibly changes arrangement. They are
  per-component, not system tokens.

## Previewing

Static files with no build step. Serve the directory and open it:

```bash
python3 -m http.server 4321
```

Then visit <http://localhost:4321>. Opening `index.html` from the filesystem
works too, apart from the canvas contrast measurement, which needs an origin.
