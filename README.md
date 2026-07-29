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
can't move laterally between them.

**The home page is pinned to one screen.** `body[data-view="contents"]` sets `overflow: hidden` and
sizes the sheet to `100dvh`. Type and spacing scale on `min(vw, vh)` so a short window shrinks the
type instead of overflowing, and two height breakpoints drop the least load-bearing lines (the margin
annotation under 720px tall, the entry glosses under 580px). Inner pages scroll normally.

**The scrubber.** Dragging across the episode waveform reveals the transcript line at that timestamp.
There is no audio file and no fake progress bar — the interaction surfaces real text. The waveform
itself is drawn on a canvas from a seeded PRNG, so it looks like speech and is identical on every
visit. It works with a pointer, and with the keyboard (arrows nudge 15s, PageUp/PageDown 2 minutes,
Home/End jump to the edges) via `role="slider"` with a live `aria-valuetext`.

## Editing the content

| What | Where |
| --- | --- |
| Transcript lines behind the scrubber | `markers` array in the `<script>` |
| Episode length | `DURATION` (seconds) — keep the `48:12` display strings in sync |
| Show notes | `.notes` list in the episode section |
| Hosts and producer | `.people` block in the hosts section |
| Sections on the home page | `.entry` links in `<nav class="contents">` |

To add a second episode, add an `.entry` row to the contents page and a matching
`<section class="view" id="view-…">`, then add its name to the `NAMES` array in the script.

## Note

This is a demonstration site. The show, the hosts, the guest, and the episode are invented — no real
person is depicted. Replace the copy before using it for anything real.
