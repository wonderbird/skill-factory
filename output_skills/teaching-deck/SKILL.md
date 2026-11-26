---
name: teaching-deck
description: >-
  Teaching presentations as reveal.js decks built from the user's own notes:
  learning goals, a timed storyline, slides with speaker notes, an optional
  workshop part. Use when asked to create, revise or brief on slides for a
  talk, lecture, training or workshop.
argument-hint: "[topic] [notes entry point] [talk minutes] [deck language]"
---

# Teaching Deck

STARTER_CHARACTER = 🎓

Act as a technical writer who teaches. The deck exists to change what the
audience can do afterwards, and its content comes from the user's notes.

Arguments: $ARGUMENTS

## Ground rules

- **Sources first.** Every factual claim on a slide traces to a note the user
  provided or to a source the user approved. Where the notes leave a gap, list
  the gap and ask. Material from your own knowledge enters the deck only after
  the user agrees, and the reading log marks it as such.
- **The folder is the boundary.** Read notes and media only inside
  `presentation/zettelkasten/`. A linked file missing from it is either absent
  on purpose or not exported yet; step 2 asks the user which. Ask before a
  personal detail about a private person goes onto a slide.
- **Deck language.** Slides and speaker notes use the deck language, with its
  typography (for German: „…" quotes, real umlauts, `–` as dash). Chat with
  the user stays in the chat language. Technical terms the audience uses in
  English stay in English.
- **One question at a time**, your recommendation first with its reason.
- **Two gates.** Stop for the user after the understanding summary (step 3)
  and after the outline (step 4). Both gates apply whenever a run passes
  step 3 or step 4, whatever the user asks to do next. Slides written before
  the outline is approved waste the user's review time.

## Folder layout

One presentation per repository. Everything for it lives in
`presentation/` in the current working directory:

- `presentation/zettelkasten/` holds the excerpt of the user's Zettelkasten
  for this talk. It is read-only input.
- `presentation/prep/` holds the working files `outline.md` and `sources.md`.
- `presentation/slides/` holds the deliverable: `index.html`, images, and
  reveal.js when vendored. It holds nothing else, so the user can share or
  host it as it is.

## Existing deck

When `presentation/prep/` or `presentation/slides/` already holds files, the
skill runs in a later session on the same deck. Read `outline.md`,
`sources.md` and `index.html` first; they carry everything earlier sessions
decided. Then serve one of two requests:

- **Changes or questions.** Answer the questions. Make each change by
  re-entering at the step it touches: new or changed content at step 2 (new
  notes) or step 4 (new structure), form and wording at step 5. Always finish
  with step 6.
- **Briefing**, for example on the day of the talk. Run the checks of step 6
  without fixing; report failures in the briefing and ask before changing
  anything. Then give a short briefing: the core message, the sections with
  their minutes, the key transitions, the open gaps, and the sections marked
  "cut if late".

A deck in another format, such as a `.pptx` file, is source material: rebuild
it from step 1.

## Steps

### 1. Intake

**Notes first.** Before anything else, find the user's notes:

1. Look in `presentation/zettelkasten/`. When the folder is missing or
   empty, ask whether the user has notes for this talk and where. Suggest
   `presentation/zettelkasten/` as the place to put them.
2. When the arguments and the message give no topic, ask for it. Then pick
   the entry point: the note that matches the topic and mostly links to other
   notes (a map of content). When several fit, list them.
3. Ask the user to confirm the entry point.

Then take what the arguments and the user's message give, and ask for each
missing required item. Audience and duration shape everything else, so ask for
those first.

Required:

- topic or topics, and how they relate
- audience: role, prior knowledge, what they need the talk for
- deck language
- talk duration, without the workshop part
- entry point into the notes, confirmed by the user

Optional, with defaults to announce when the user gives no value:

- format: a reveal.js HTML deck. When the user asks for another format, such
  as PowerPoint, say that this skill builds reveal.js decks, and ask whether
  to continue.
- learning goals: none up front; step 3 proposes them from the notes.
- workshop part: none. When the user mentions a workshop without its duration
  or format, ask for them. The workshop has its own duration, separate from
  the talk duration.
- venue: a 16:9 projector, internet access so reveal.js loads from the CDN,
  no PDF handout.

Done when the user has confirmed the entry point, every required item is
known, and every optional item has a value or an announced default.

### 2. Read the notes

Start at the entry point. Read every note it links to directly. Follow deeper
links where they carry substance for the topic.

Keep a reading log in `presentation/prep/sources.md`. Per note: path, key
claims, examples and stories, quotes, external sources it cites. Collect
everything the notes leave open or contradict under the heading "Gaps". This
log is the trace for every slide later.

**Missing files.** Collect the missing link targets you would follow by the
rule above, plus every media file embedded in a note you read. A link
`[[Name|alias]]` or `[[Name#Heading]]` points to `Name.md`; a target with an
extension other than `.md`, such as `![[chart.png]]` or `[[paper.pdf]]`, is
media. List a missing link you would not follow under "Skipped" as off-topic,
without asking. Leave out the files listed under "Absent on purpose" in
`sources.md`; an earlier session already settled them. Then:

1. Show the user the remaining missing files in one list and ask which are
   absent on purpose. Record those under "Absent on purpose" in `sources.md`.
2. For the others, give copy commands for the user to run. The user's
   Zettelkasten keeps notes in `Unsorted/`, literature notes in
   `Bibliography/` and media in `Assets/Unsorted/`. Ask once for the
   Zettelkasten root folder and record it under "Zettelkasten root" in
   `sources.md` as an absolute path (expand a leading `~` to the home
   folder). The commands copy into the top of the excerpt and never
   overwrite (`--update=none`); when the excerpt uses subfolders, adapt the
   target. A note may sit in either notes folder, so search both. Escape
   `[`, `]`, `*` and `?` in note names with a backslash, because `find -name`
   reads them as patterns. Tell the user: every file found is printed, and a
   name not printed was found in neither folder.

   ```bash
   ZK="<Zettelkasten root>"
   for n in "<note name 1>" "<note name 2>"; do
     find "$ZK/Unsorted" "$ZK/Bibliography" -maxdepth 1 -name "$n.md" \
       -print -exec cp --update=none {} presentation/zettelkasten/ \;
   done
   cp --update=none "$ZK/Assets/Unsorted/<media file>" \
     presentation/zettelkasten/
   ```

3. When the user has run them, read the new notes and repeat this check.

Done when every note linked from the entry point is in the log or listed
under "Skipped" as off-topic, and every missing file you would follow or that
a note you read embeds is in the folder, under "Absent on purpose", or under
"Skipped" as off-topic.

### 3. Summarize understanding and hand back

Present in chat:

- the core message in one sentence
- 3 to 5 learning goals, each phrased as what the audience can do afterwards
- a proposed storyline: sections with rough minutes
- the gaps from the reading log, including claims that need a primary source
- what the workshop part could practise, if there is one

Then ask the user what they want to achieve next in this session (for
example: refine the goals, write the outline and then the deck, design the
workshop). Use the question tool when one is available.

Done when the user has answered.

### 4. Outline with time budget

Read [references/teaching-design.md](references/teaching-design.md) before
you draft. It holds the storyline pattern, the timing rules and the workshop
pattern.

Write `presentation/prep/outline.md`. Per section: its learning goal, its
minutes, and its slides. Per slide: a title, a one-line message, the source
note it draws on, and its planned seconds from the timing rules. List the
workshop slides in their own part, with the exercise timeboxes as their
planned seconds.

Done when the planned seconds of the talk slides add up to at most the talk
duration minus the buffer from the reference, the workshop slides add up to
at most the workshop duration, every learning goal is served by a section,
and every learning goal is checked in the recap or the workshop. Present the
outline and wait for the user's approval.

### 5. Build the deck

Read [references/revealjs.md](references/revealjs.md) before you write HTML.
Start from [assets/deck-template.html](assets/deck-template.html) and save the
deck as `presentation/slides/index.html`. Images and other files the deck
uses go into `presentation/slides/` as well.

- One message per slide. The title states the message as a sentence (an
  assertion title), not a topic label.
- Visible text stays short: about 30 words per slide. Detail goes into the
  speaker notes.
- Every slide has speaker notes: what to say, the transition to the next
  slide, and the source note.
- Every talk and workshop slide carries `data-timing` with its planned seconds
  from the outline, so the speaker view can show the pace.
- Show processes and relations as a diagram (inline SVG or HTML/CSS boxes)
  rather than as a bullet list.
- Backup material goes into an appendix after the closing slide. Appendix
  slides carry no `data-timing`.

Done when every slide in the approved outline exists in the deck, in outline
order.

### 6. Verify

Check each item and fix what fails:

- The deck opens without console errors. Use a headless browser when one is
  available; otherwise tell the user to open it and press `S` for the speaker
  view.
- No slide overflows the 1280×720 canvas. With a headless browser, take one
  screenshot per slide at a window size of 1280×720, opening
  `index.html#/<h>/<v>` for each slide; otherwise list the dense slides for
  the user to check.
- Every slide (every leaf `<section>`, as `Reveal.getSlides()` returns them)
  holds one `<aside class="notes">`.
- The `data-timing` values of the talk slides add up to at most the talk
  duration minus the buffer; those of the workshop slides add up to at most
  the workshop duration.
- Every factual claim on a slide traces to an entry in
  `presentation/prep/sources.md`.
- `presentation/slides/` holds only files the deck uses.
- Slides and notes use the deck language and its typography throughout.

Report to the user: the deck path, how to present it (open
`presentation/slides/index.html`, `S` speaker view, `F` fullscreen), the
checks that passed, and the gaps from `presentation/prep/sources.md`. When
the deck loads reveal.js from the CDN, also name the fallbacks for a venue
without internet: vendoring reveal.js (see
[references/revealjs.md](references/revealjs.md)) or a PDF export via
`?print-pdf`.
