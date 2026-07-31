# Within Earshot

A website for a podcast about artificial intelligence where it actually lands — the triage desk,
the classroom, the kitchen table. The show talks to the people holding the technology, not the
people selling it.

Single self-contained page. No build step, no dependencies, no network requests.

## Running it

Open `index.html` in a browser. That's the whole thing.

If you want a local server (useful for testing on your phone over the same network):

```bash
python -m http.server 8000
# then visit http://localhost:8000
```

## How it's put together

Everything lives in `index.html` — markup, styles, and script. It's one file on purpose: the site
is small enough that splitting it would cost more than it saves.

**Design.** A ruled notebook on a desk. Punched holes down the gutter, a pink margin rule, ballpoint
blue-black ink on off-white paper, red pen for the guest's voice. Deliberately light-only — the CSS
custom properties are declared once on `:root` and never redefined, so there is no dark variant to
maintain and no half-styled state.

Two typefaces, each doing a job:

- a serif (Iowan Old Style / Palatino / Georgia) for anything a person says
- a monospace (Cascadia / Consolas) for anything a machine measures — timecodes, page numbers, labels

Both are system font stacks, so there are no webfont requests and nothing to fall back from.

**Navigation.** The contents page is the only door. There is no persistent nav bar; every section is
reached from the home page, and the only way out of a section is `← Contents`. Routing is hash-based
(`#episode`, `#hosts`, `#listen`, `#about`), so browser back works and sections are linkable, but you
can't move laterally between them. Each route sets its own `document.title`, so history entries and
open tabs are distinguishable.

The section ids *are* those fragments — `id="episode"`, not `id="view-episode"`. That is what makes
the fallbacks work: with script disabled, the `<noscript>` block simply stops hiding the sections and
the same links become ordinary in-page anchors down one continuous read. Printing does the same and
puts each section on its own sheet.

**The home page is pinned to one screen.** `body[data-view="contents"]` sets `overflow: hidden` and
sizes the sheet to `100dvh`; the `.book` and `.sheet` clip rather than scroll, so no inner scrollbar
can appear either. Type and spacing scale on `min(vw, vh)` so a short window shrinks the type instead
of overflowing, and the home page runs a wider measure than the prose views on purpose — an extra
wrapped line costs the same height whether or not it earns it, so the headline, the standfirst and
the entry glosses are each given room to sit on as few lines as they need.

The contents group is centred on the sheet rather than sprung apart. An earlier version anchored the
index to the foot of the page with `margin-top: auto`, which meant all the leftover height collected
into a single hole above it — and the shorter the copy, the bigger the hole. Centring spends it as
even margin above and below, so the page reads as one compact block.

Four breakpoints handle the sizes where scaling alone isn't enough. Above 940px tall the title and
the entry padding grow, so the block scales with the sheet instead of sitting on it as a small island.
Under 720px tall the margin annotation goes; under 580px tall the entry glosses and the footer line
go; under 560px *wide* the glosses go too, since a narrow column doubles each one. Under 460px tall
the page gives up the pin and scrolls, rather than clip the contents out of reach. Inner pages scroll
normally at every size.

The fit is verified, not assumed — it holds with no scroll and no clipping from 2560×1440 down to
360×640, including 1366×768, 1280×720 and 390×844.

**The scrubber.** Dragging across the episode waveform reveals the transcript line at that timestamp.
There is no audio file and no fake progress bar — the interaction surfaces real text. The waveform
itself is drawn on a canvas from a seeded PRNG, so it looks like speech and is identical on every
visit. It works with a pointer, and with the keyboard (arrows nudge 15s, PageUp/PageDown 2 minutes,
Home/End jump to the edges) via `role="slider"`.

Ticks along the foot of the waveform mark where each transcript line starts — without them a drag is
a blind hunt — and the tick for the current line is drawn in red. The show-note timecodes are buttons
wired to the same scrubber, because the notes and the tape are the same recording; a chapter mark
that can't move the tape is just a number printed next to a sentence.

Three things the scrubber is careful about:

- `.readout` is an `aria-live` region. Rewriting it on every pointer sample announced the same line
  sixty times a second, so it only touches the DOM when the tape reaches a *different* line.
- `aria-valuetext` carries position and speaker only. It used to carry the whole transcript line too,
  which meant a 15-second nudge read the paragraph out twice — once from the slider, once from the
  live region.
- Painting is coalesced to one frame with `requestAnimationFrame`, and the CSS custom properties are
  read once into a cache rather than three `getComputedStyle` calls per pointer sample.

`DURATION` is the single source of truth: the `48:12` display strings and `aria-valuemax` are derived
from it at boot, so they cannot drift out of sync.

## Editing the content

| What | Where |
| --- | --- |
| Transcript lines behind the scrubber | `markers` array in the `<script>` |
| Episode length | `DURATION` (seconds) — display strings derive from it |
| Show notes | `.notes` list — keep each `data-t` equal to its marker's `t` |
| Hosts and producer | `.people` block in the hosts section |
| Sections on the home page | `.entry` links in `<nav class="contents">` |
| Per-view page titles | `TITLES` map in the script |

To add a second episode, add an `.entry` row to the contents page and a matching
`<section class="view" id="…">` whose id is the fragment the entry links to, then add that name to
both the `NAMES` array and the `TITLES` map in the script.

## Deploying

The repo carries a [`render.yaml`](render.yaml) blueprint. It declares one static site, no build
command (there is nothing to build), publishing the repo root. Pushing to `main` redeploys.

First time only, on Render: **New → Blueprint**, pick this repo, apply. After that it is just
`git push`.

## Note

This is a demonstration site. The show, the hosts, the guest, and the episode are invented — no real
person is depicted. Replace the copy before using it for anything real.
