# reveal.js Reference

Facts for building the deck. The template in `assets/deck-template.html`
already applies the setup below.

## Setup

- The template loads reveal.js 5.2.1 from the jsDelivr CDN with a pinned
  version. Keep the version pinned, so the deck renders the same way next
  year.
- For a venue without reliable internet, vendor reveal.js next to the deck:
  download `reveal.js-5.2.1` from the GitHub release and copy only the files
  the template loads into `presentation/slides/`, keeping their paths:
  - `dist/reset.css`, `dist/reveal.css`, `dist/reveal.js`
  - `dist/theme/white.css` and `dist/theme/fonts/source-sans-pro/`, because
    `white.css` imports that font
  - `plugin/notes/notes.js`, `plugin/highlight/highlight.js`,
    `plugin/highlight/monokai.css`

  Then change the `https://cdn.jsdelivr.net/npm/reveal.js@5.2.1/` prefixes in
  `index.html` to relative paths.
- The canvas is 1280×720 (`width`, `height` in `Reveal.initialize`). reveal.js
  scales it to the screen. Design every slide for that size.

## Slides

- Each `<section>` is one slide. A `<section>` that contains `<section>`
  children forms a vertical stack: use it for the appendix or for optional
  deep dives the speaker can skip with the right arrow.
- Speaker notes: `<aside class="notes">…</aside>` inside the slide. The notes
  plugin shows them in the speaker view (`S` key), together with the current
  and next slide and a timer.
- Pacing: `data-timing="90"` on a `<section>` sets that slide's planned
  seconds. The speaker view then shows whether the speaker is ahead or
  behind. `defaultTiming` in the config applies to slides without it.
- Fragments: `class="fragment"` reveals an element on the next key press.
  Variants: `fade-up`, `highlight-current-blue`, `semi-fade-out`. Use them to
  pace a reveal the speaker talks through, such as the steps of a diagram.
- Auto-animate: `data-auto-animate` on two consecutive slides animates
  elements that share the same `data-id`. It suits a diagram that grows step
  by step.
- Layout helpers: `r-stack` (stack elements on top of each other, combined
  with fragments), `r-fit-text` (scale one line to full width), `r-stretch`
  (an image fills the remaining height), `r-hstack` and `r-vstack`.
- Backgrounds: `data-background-color`, `data-background-image`. Use them for
  section title slides, so the audience sees the structure.
- Code: `<pre><code class="language-python" data-trim data-line-numbers="2|4-5">`
  highlights the given lines step by step (highlight plugin).

## Styling

- Use one built-in theme (the template uses `white`) and override it with CSS
  custom properties in the `<style>` block: `--r-main-font`,
  `--r-heading-font`, `--r-main-color`, `--r-heading-color`, `--r-link-color`,
  `--r-main-font-size`, `--r-heading1-size` and so on.
- Keep body text at 28 px or larger on the 1280×720 canvas. Smaller text is
  unreadable in the back row.
- Ensure a contrast ratio of at least 4.5:1 between text and background.
  Projectors wash out light colours.
- Headings in reveal.js themes are uppercase by default in some themes. The
  template sets `--r-heading-text-transform: none`, because assertion titles
  are sentences.

## Export and presenting

- PDF: open `presentation/slides/index.html?print-pdf` in Chrome or Chromium and print to PDF
  (margins none, background graphics on). `showNotes: "separate-page"` in the
  config prints the speaker notes on their own pages.
- Keys: `S` speaker view, `F` fullscreen, `O` or `Esc` overview, `B` black
  screen, `?` help.
- The speaker view opens a popup window. When the deck is opened as a local
  file and the popup stays empty, serve the folder instead, for example with
  `python3 -m http.server`.

## Pitfalls

- Text that does not fit is clipped silently, because reveal.js scales the
  canvas but never the content inside it. Check every dense slide.
- The markdown plugin (`data-markdown`) is convenient, but speaker notes and
  attributes need special syntax there. The template uses plain HTML.
- Do not nest vertical stacks deeper than one level; reveal.js supports only
  two dimensions.
