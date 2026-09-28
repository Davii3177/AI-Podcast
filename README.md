# Within Earshot

A website for a podcast about artificial intelligence where it actually lands — the triage desk,
the classroom, the kitchen table. The show talks to the people holding the technology, not the
people selling it.

Single self-contained page. No build step and no dependencies. The only network request is one
Google Fonts stylesheet for the heading typeface.

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

**Design.** Editorial and high-contrast: black ink on warm newsprint, with a single signal red as
the only accent. Corners are square everywhere (`border-radius: 0` is set globally) and there are
no drop shadows. Blocks are separated by 1px black rules. The home page's blocks sit on a 1px-gap
grid over an ink background, so each block is ruled off without any border being drawn twice. The
layouts are deliberately asymmetric: the hero headline runs across ten of twelve columns next to a
narrow "issue" sidebar, the home page mixes block sizes, and the episode page is a split screen
whose left column stays put while the right column scrolls. There are no illustrations; big type
is the focal point, whether that's the headline, a pull quote or the huge "01".

The site uses three typefaces, each doing a job:

- Instrument Serif (Google Fonts) for headlines and anything a person says out loud
- a system text serif (Charter / Iowan / Georgia) for body copy
- a system monospace for labels, timecodes and navigation

If the webfont never loads, the headings fall back to a system serif. The site is deliberately
light-only: the CSS custom properties are declared once on `:root`, and the canvas reads them once
at boot.

**Navigation.** A sticky top bar links to every section, and the current one is marked with
`aria-current`. The home page also leads into each section through its cards. Routing is
hash-based (`#episode`, `#hosts`, `#listen`, `#about`), so browser back works and sections are
linkable. Each route sets its own `document.title`, so history entries and open tabs are
distinguishable.

The section ids *are* those fragments — `id="episode"`, not `id="view-episode"`. That is what makes
the fallbacks work: with script disabled, the `<noscript>` block simply stops hiding the sections and
the same links become ordinary in-page anchors down one continuous read. Printing does the same and
puts each section on its own sheet.

**The home page** is an asymmetric hero (headline, "issue" sidebar, standfirst) over a board of
unequal blocks: the episode with a pull quote, the hosts, an inverted listen block, a big "01"
and the house rule. Everything collapses to a single column under 900px.

**The scrubber.** Dragging across the episode waveform reveals the transcript line at that timestamp.
There is no audio file and no fake progress bar — the interaction surfaces real text. Hovering alone
does nothing, so a jump from the show notes isn't undone when the pointer crosses the waveform. The waveform
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
| Sections on the home page | `.block` links in `<nav class="board">`, plus the masthead `.nav` |
| Per-view page titles | `TITLES` map in the script |

To add a second episode, add a `.block` to the home page board and a matching
`<section class="view" id="…">` whose id is the fragment the block links to, then add that name to
both the `NAMES` array and the `TITLES` map in the script.

## Deploying

The repo carries a [`render.yaml`](render.yaml) blueprint. It declares one static site, no build
command (there is nothing to build), publishing the repo root. Pushing to `main` redeploys.

First time only, on Render: **New → Blueprint**, pick this repo, apply. After that it is just
`git push`.

## Note

This is a demonstration site. The show, the hosts, the guest, and the episode are invented — no real
person is depicted. Replace the copy before using it for anything real.
